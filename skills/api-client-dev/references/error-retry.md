# Error Handling & Retry Strategies

## Error Categorization

```typescript
enum ErrorCategory {
  NETWORK = 'network',      // Connection issues, timeouts
  AUTH = 'auth',            // 401, 403
  VALIDATION = 'validation', // 400, 422
  RATE_LIMIT = 'rate_limit', // 429
  SERVER = 'server',        // 500, 502, 503, 504
  CLIENT = 'client'         // Other 4xx
}

interface APIError extends Error {
  category: ErrorCategory;
  status?: number;
  retryable: boolean;
  retryAfter?: number;
}

function categorizeError(error: any): APIError {
  const apiError = new Error(error.message) as APIError;

  if (!error.status) {
    apiError.category = ErrorCategory.NETWORK;
    apiError.retryable = true;
    return apiError;
  }

  const status = error.status;
  apiError.status = status;

  if (status === 401 || status === 403) {
    apiError.category = ErrorCategory.AUTH;
    apiError.retryable = status === 401; // Can retry after re-auth
  } else if (status === 429) {
    apiError.category = ErrorCategory.RATE_LIMIT;
    apiError.retryable = true;
    apiError.retryAfter = parseInt(error.headers?.['retry-after'] || '60');
  } else if (status >= 500) {
    apiError.category = ErrorCategory.SERVER;
    apiError.retryable = true;
  } else if (status === 400 || status === 422) {
    apiError.category = ErrorCategory.VALIDATION;
    apiError.retryable = false;
  } else {
    apiError.category = ErrorCategory.CLIENT;
    apiError.retryable = false;
  }

  return apiError;
}
```

---

## Exponential Backoff

### Basic Implementation

```typescript
interface RetryOptions {
  maxRetries: number;
  baseDelay: number;
  maxDelay: number;
  jitter: boolean;
}

const DEFAULT_RETRY: RetryOptions = {
  maxRetries: 3,
  baseDelay: 1000,
  maxDelay: 30000,
  jitter: true
};

function calculateDelay(attempt: number, options: RetryOptions): number {
  // Exponential: 1s, 2s, 4s, 8s...
  let delay = Math.min(
    options.baseDelay * Math.pow(2, attempt),
    options.maxDelay
  );

  // Add jitter (0-100% of delay)
  if (options.jitter) {
    delay = delay * (0.5 + Math.random());
  }

  return Math.floor(delay);
}

async function sleep(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

### Retry Wrapper

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  options: Partial<RetryOptions> = {}
): Promise<T> {
  const opts = { ...DEFAULT_RETRY, ...options };
  let lastError: Error | undefined;

  for (let attempt = 0; attempt <= opts.maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error: any) {
      lastError = error;
      const apiError = categorizeError(error);

      if (!apiError.retryable || attempt === opts.maxRetries) {
        throw apiError;
      }

      const delay = apiError.retryAfter
        ? apiError.retryAfter * 1000
        : calculateDelay(attempt, opts);

      console.log(`Retry ${attempt + 1}/${opts.maxRetries} after ${delay}ms`);
      await sleep(delay);
    }
  }

  throw lastError;
}
```

---

## Axios Interceptors

### Request Interceptor (Auth)

```typescript
import axios, { AxiosInstance } from 'axios';

function createAuthenticatedClient(
  baseURL: string,
  tokenManager: TokenManager
): AxiosInstance {
  const client = axios.create({ baseURL });

  client.interceptors.request.use(async (config) => {
    const token = await tokenManager.getValidToken();
    config.headers.Authorization = `Bearer ${token}`;
    return config;
  });

  return client;
}
```

### Response Interceptor (Retry)

```typescript
function addRetryInterceptor(
  client: AxiosInstance,
  options: Partial<RetryOptions> = {}
): void {
  const opts = { ...DEFAULT_RETRY, ...options };

  client.interceptors.response.use(
    response => response,
    async error => {
      const config = error.config;
      config.__retryCount = config.__retryCount || 0;

      const apiError = categorizeError({
        status: error.response?.status,
        message: error.message,
        headers: error.response?.headers
      });

      if (!apiError.retryable || config.__retryCount >= opts.maxRetries) {
        throw apiError;
      }

      config.__retryCount++;
      const delay = apiError.retryAfter
        ? apiError.retryAfter * 1000
        : calculateDelay(config.__retryCount - 1, opts);

      await sleep(delay);
      return client(config);
    }
  );
}
```

---

## Circuit Breaker Pattern

```typescript
enum CircuitState {
  CLOSED = 'closed',   // Normal operation
  OPEN = 'open',       // Failing, reject requests
  HALF_OPEN = 'half_open' // Testing recovery
}

interface CircuitBreakerOptions {
  failureThreshold: number;  // Failures before opening
  resetTimeout: number;      // Time before half-open
  successThreshold: number;  // Successes to close
}

class CircuitBreaker {
  private state: CircuitState = CircuitState.CLOSED;
  private failures = 0;
  private successes = 0;
  private lastFailure?: number;

  constructor(private options: CircuitBreakerOptions) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.OPEN) {
      if (Date.now() - this.lastFailure! > this.options.resetTimeout) {
        this.state = CircuitState.HALF_OPEN;
        this.successes = 0;
      } else {
        throw new Error('Circuit breaker is open');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess(): void {
    this.failures = 0;
    if (this.state === CircuitState.HALF_OPEN) {
      this.successes++;
      if (this.successes >= this.options.successThreshold) {
        this.state = CircuitState.CLOSED;
      }
    }
  }

  private onFailure(): void {
    this.failures++;
    this.lastFailure = Date.now();
    if (this.failures >= this.options.failureThreshold) {
      this.state = CircuitState.OPEN;
    }
  }

  getState(): CircuitState {
    return this.state;
  }
}
```

---

## Timeout Handling

```typescript
async function withTimeout<T>(
  promise: Promise<T>,
  timeoutMs: number,
  message = 'Request timed out'
): Promise<T> {
  let timeoutId: NodeJS.Timeout;

  const timeoutPromise = new Promise<never>((_, reject) => {
    timeoutId = setTimeout(() => reject(new Error(message)), timeoutMs);
  });

  try {
    return await Promise.race([promise, timeoutPromise]);
  } finally {
    clearTimeout(timeoutId!);
  }
}

// Usage with fetch
async function fetchWithTimeout(
  url: string,
  options: RequestInit = {},
  timeoutMs = 30000
): Promise<Response> {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs);

  try {
    return await fetch(url, { ...options, signal: controller.signal });
  } finally {
    clearTimeout(timeoutId);
  }
}
```

---

## Error Response Formatting

```typescript
interface ErrorResponse {
  error: {
    code: string;
    message: string;
    details?: Record<string, any>;
    retryable: boolean;
  };
}

function formatErrorResponse(error: APIError): ErrorResponse {
  return {
    error: {
      code: `${error.category.toUpperCase()}_ERROR`,
      message: error.message,
      details: error.status ? { status: error.status } : undefined,
      retryable: error.retryable
    }
  };
}
```

---

## Best Practices

| Scenario | Strategy |
|----------|----------|
| Network errors | Retry with exponential backoff |
| 429 Rate limit | Honor Retry-After header |
| 401 Unauthorized | Refresh token, retry once |
| 403 Forbidden | Don't retry, check permissions |
| 400/422 Validation | Don't retry, fix request |
| 500 Server error | Retry with backoff |
| 502/503/504 | Retry with backoff |
| Timeout | Retry with increased timeout |
