# Rate Limiting Strategies

## Token Bucket Algorithm

```typescript
class TokenBucket {
  private tokens: number;
  private lastRefill: number;

  constructor(
    private capacity: number,      // Max tokens
    private refillRate: number,    // Tokens per second
    private refillInterval: number = 1000 // Refill check interval
  ) {
    this.tokens = capacity;
    this.lastRefill = Date.now();
  }

  private refill(): void {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    const tokensToAdd = elapsed * this.refillRate;
    this.tokens = Math.min(this.capacity, this.tokens + tokensToAdd);
    this.lastRefill = now;
  }

  async acquire(tokens = 1): Promise<void> {
    this.refill();

    if (this.tokens >= tokens) {
      this.tokens -= tokens;
      return;
    }

    // Wait for tokens to become available
    const tokensNeeded = tokens - this.tokens;
    const waitTime = (tokensNeeded / this.refillRate) * 1000;
    await new Promise(resolve => setTimeout(resolve, waitTime));

    this.refill();
    this.tokens -= tokens;
  }

  getAvailable(): number {
    this.refill();
    return Math.floor(this.tokens);
  }
}
```

---

## Request Queue

```typescript
interface QueuedRequest<T> {
  fn: () => Promise<T>;
  resolve: (value: T) => void;
  reject: (error: any) => void;
  priority: number;
}

class RequestQueue {
  private queue: QueuedRequest<any>[] = [];
  private processing = false;
  private tokenBucket: TokenBucket;

  constructor(
    requestsPerSecond: number,
    burstCapacity: number = requestsPerSecond
  ) {
    this.tokenBucket = new TokenBucket(burstCapacity, requestsPerSecond);
  }

  async enqueue<T>(fn: () => Promise<T>, priority = 0): Promise<T> {
    return new Promise((resolve, reject) => {
      const request: QueuedRequest<T> = { fn, resolve, reject, priority };

      // Insert by priority (higher first)
      const index = this.queue.findIndex(r => r.priority < priority);
      if (index === -1) {
        this.queue.push(request);
      } else {
        this.queue.splice(index, 0, request);
      }

      this.processQueue();
    });
  }

  private async processQueue(): Promise<void> {
    if (this.processing || this.queue.length === 0) return;

    this.processing = true;

    while (this.queue.length > 0) {
      await this.tokenBucket.acquire();
      const request = this.queue.shift()!;

      try {
        const result = await request.fn();
        request.resolve(result);
      } catch (error) {
        request.reject(error);
      }
    }

    this.processing = false;
  }

  getQueueLength(): number {
    return this.queue.length;
  }
}
```

---

## Rate Limit Header Parsing

```typescript
interface RateLimitInfo {
  limit: number;       // Max requests in window
  remaining: number;   // Requests left
  reset: Date;         // When limit resets
  retryAfter?: number; // Seconds to wait (if 429)
}

function parseRateLimitHeaders(headers: Headers): RateLimitInfo | null {
  // Standard headers
  const limit = headers.get('X-RateLimit-Limit')
    || headers.get('RateLimit-Limit');
  const remaining = headers.get('X-RateLimit-Remaining')
    || headers.get('RateLimit-Remaining');
  const reset = headers.get('X-RateLimit-Reset')
    || headers.get('RateLimit-Reset');
  const retryAfter = headers.get('Retry-After');

  if (!limit || !remaining) return null;

  return {
    limit: parseInt(limit),
    remaining: parseInt(remaining),
    reset: new Date(parseInt(reset || '0') * 1000),
    retryAfter: retryAfter ? parseInt(retryAfter) : undefined
  };
}

// Common header patterns by API
const RATE_LIMIT_HEADERS = {
  github: {
    limit: 'X-RateLimit-Limit',
    remaining: 'X-RateLimit-Remaining',
    reset: 'X-RateLimit-Reset'
  },
  twitter: {
    limit: 'x-rate-limit-limit',
    remaining: 'x-rate-limit-remaining',
    reset: 'x-rate-limit-reset'
  },
  stripe: {
    // Uses Retry-After on 429
    retryAfter: 'Retry-After'
  }
};
```

---

## Adaptive Rate Limiting

```typescript
class AdaptiveRateLimiter {
  private currentRate: number;
  private minRate: number;
  private maxRate: number;
  private tokenBucket: TokenBucket;
  private consecutiveSuccess = 0;
  private consecutiveFailure = 0;

  constructor(
    initialRate: number,
    minRate: number,
    maxRate: number
  ) {
    this.currentRate = initialRate;
    this.minRate = minRate;
    this.maxRate = maxRate;
    this.tokenBucket = new TokenBucket(initialRate, initialRate);
  }

  async acquire(): Promise<void> {
    await this.tokenBucket.acquire();
  }

  onSuccess(): void {
    this.consecutiveSuccess++;
    this.consecutiveFailure = 0;

    // Increase rate after 10 consecutive successes
    if (this.consecutiveSuccess >= 10 && this.currentRate < this.maxRate) {
      this.adjustRate(this.currentRate * 1.1);
      this.consecutiveSuccess = 0;
    }
  }

  onRateLimit(retryAfter?: number): void {
    this.consecutiveFailure++;
    this.consecutiveSuccess = 0;

    // Decrease rate on rate limit
    this.adjustRate(this.currentRate * 0.5);

    // If retry-after provided, use it
    if (retryAfter) {
      console.log(`Rate limited. Waiting ${retryAfter}s`);
    }
  }

  private adjustRate(newRate: number): void {
    this.currentRate = Math.max(this.minRate, Math.min(this.maxRate, newRate));
    this.tokenBucket = new TokenBucket(
      Math.ceil(this.currentRate),
      this.currentRate
    );
    console.log(`Adjusted rate to ${this.currentRate.toFixed(1)} req/s`);
  }

  getCurrentRate(): number {
    return this.currentRate;
  }
}
```

---

## Rate-Limited Client

```typescript
class RateLimitedAPIClient {
  private queue: RequestQueue;
  private rateLimiter: AdaptiveRateLimiter;

  constructor(
    private baseURL: string,
    private apiKey: string,
    requestsPerSecond = 10
  ) {
    this.queue = new RequestQueue(requestsPerSecond);
    this.rateLimiter = new AdaptiveRateLimiter(
      requestsPerSecond,
      1,
      requestsPerSecond * 2
    );
  }

  async request<T>(
    endpoint: string,
    options: RequestInit = {},
    priority = 0
  ): Promise<T> {
    return this.queue.enqueue(async () => {
      await this.rateLimiter.acquire();

      const response = await fetch(`${this.baseURL}${endpoint}`, {
        ...options,
        headers: {
          'Authorization': `Bearer ${this.apiKey}`,
          'Content-Type': 'application/json',
          ...options.headers
        }
      });

      const rateLimitInfo = parseRateLimitHeaders(response.headers);

      if (response.status === 429) {
        this.rateLimiter.onRateLimit(rateLimitInfo?.retryAfter);
        throw new Error('Rate limited');
      }

      this.rateLimiter.onSuccess();

      if (!response.ok) {
        throw new Error(`API error: ${response.status}`);
      }

      return response.json();
    }, priority);
  }
}
```

---

## Batch Request Optimization

```typescript
class BatchProcessor<T, R> {
  private batch: T[] = [];
  private pending: Map<T, { resolve: (r: R) => void; reject: (e: any) => void }[]> = new Map();
  private timer?: NodeJS.Timeout;

  constructor(
    private processBatch: (items: T[]) => Promise<Map<T, R>>,
    private maxBatchSize: number = 100,
    private maxWaitMs: number = 50
  ) {}

  async add(item: T): Promise<R> {
    return new Promise((resolve, reject) => {
      // Add to pending
      if (!this.pending.has(item)) {
        this.pending.set(item, []);
        this.batch.push(item);
      }
      this.pending.get(item)!.push({ resolve, reject });

      // Process if batch is full
      if (this.batch.length >= this.maxBatchSize) {
        this.flush();
      } else if (!this.timer) {
        // Or wait for more items
        this.timer = setTimeout(() => this.flush(), this.maxWaitMs);
      }
    });
  }

  private async flush(): Promise<void> {
    if (this.timer) {
      clearTimeout(this.timer);
      this.timer = undefined;
    }

    if (this.batch.length === 0) return;

    const batchToProcess = [...this.batch];
    const pendingToResolve = new Map(this.pending);

    this.batch = [];
    this.pending.clear();

    try {
      const results = await this.processBatch(batchToProcess);

      for (const [item, callbacks] of pendingToResolve) {
        const result = results.get(item);
        if (result !== undefined) {
          callbacks.forEach(cb => cb.resolve(result));
        } else {
          callbacks.forEach(cb => cb.reject(new Error('No result for item')));
        }
      }
    } catch (error) {
      for (const callbacks of pendingToResolve.values()) {
        callbacks.forEach(cb => cb.reject(error));
      }
    }
  }
}
```

---

## Common Rate Limits by API

| API | Rate Limit | Notes |
|-----|------------|-------|
| GitHub | 5,000/hr (auth) | Per-user, separate search limit |
| Twitter | 300-900/15min | Varies by endpoint |
| Slack | 1+ req/sec | Tier-based, Burst allowed |
| OpenAI | Varies by tier | Token + request limits |
| Stripe | 100/sec (read) | 100/sec write in test |
| Google APIs | Varies | Per-project quotas |

## Best Practices

1. **Always implement rate limiting** - Even if API doesn't enforce it
2. **Parse rate limit headers** - Adapt to actual limits
3. **Use request queuing** - Prevent burst overloads
4. **Implement backoff** - Exponential with jitter
5. **Monitor usage** - Log rate limit info
6. **Batch when possible** - Reduce request count
7. **Cache responses** - Avoid redundant requests
8. **Use priority queues** - Critical requests first
