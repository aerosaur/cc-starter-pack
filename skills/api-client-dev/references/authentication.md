# Authentication Patterns

## OAuth 2.0 Implementation

### Authorization Code Flow (Most Secure)

```typescript
interface OAuthConfig {
  clientId: string;
  clientSecret: string;
  authorizationUrl: string;
  tokenUrl: string;
  redirectUri: string;
  scopes: string[];
}

class OAuthClient {
  constructor(private config: OAuthConfig) {}

  getAuthorizationUrl(state: string): string {
    const params = new URLSearchParams({
      client_id: this.config.clientId,
      redirect_uri: this.config.redirectUri,
      response_type: 'code',
      scope: this.config.scopes.join(' '),
      state
    });
    return `${this.config.authorizationUrl}?${params}`;
  }

  async exchangeCodeForTokens(code: string): Promise<TokenResponse> {
    const response = await fetch(this.config.tokenUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: new URLSearchParams({
        grant_type: 'authorization_code',
        code,
        client_id: this.config.clientId,
        client_secret: this.config.clientSecret,
        redirect_uri: this.config.redirectUri
      })
    });
    if (!response.ok) throw new Error(`Token exchange failed: ${response.status}`);
    return response.json();
  }

  async refreshAccessToken(refreshToken: string): Promise<TokenResponse> {
    const response = await fetch(this.config.tokenUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: new URLSearchParams({
        grant_type: 'refresh_token',
        refresh_token: refreshToken,
        client_id: this.config.clientId,
        client_secret: this.config.clientSecret
      })
    });
    if (!response.ok) throw new Error(`Token refresh failed: ${response.status}`);
    return response.json();
  }
}
```

### Client Credentials Flow (Server-to-Server)

```typescript
async function getClientCredentialsToken(
  tokenUrl: string,
  clientId: string,
  clientSecret: string,
  scopes: string[]
): Promise<string> {
  const response = await fetch(tokenUrl, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
      'Authorization': `Basic ${Buffer.from(`${clientId}:${clientSecret}`).toString('base64')}`
    },
    body: new URLSearchParams({
      grant_type: 'client_credentials',
      scope: scopes.join(' ')
    })
  });
  const data = await response.json();
  return data.access_token;
}
```

---

## Token Management

### Secure Token Storage (macOS Keychain)

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

class KeychainTokenStore {
  constructor(private serviceName: string) {}

  async store(key: string, value: string): Promise<void> {
    const escaped = value.replace(/"/g, '\\"');
    await execAsync(
      `security add-generic-password -s "${this.serviceName}" -a "${key}" -w "${escaped}" -U`
    );
  }

  async retrieve(key: string): Promise<string | null> {
    try {
      const { stdout } = await execAsync(
        `security find-generic-password -s "${this.serviceName}" -a "${key}" -w`
      );
      return stdout.trim();
    } catch {
      return null;
    }
  }

  async delete(key: string): Promise<void> {
    try {
      await execAsync(
        `security delete-generic-password -s "${this.serviceName}" -a "${key}"`
      );
    } catch {
      // Key doesn't exist
    }
  }
}
```

### Auto-Refresh Token Manager

```typescript
interface TokenData {
  accessToken: string;
  refreshToken: string;
  expiresAt: number;
}

class TokenManager {
  private tokenData: TokenData | null = null;
  private refreshPromise: Promise<TokenData> | null = null;

  constructor(
    private store: KeychainTokenStore,
    private refreshFn: (refreshToken: string) => Promise<TokenResponse>
  ) {}

  async getValidToken(): Promise<string> {
    if (!this.tokenData) {
      this.tokenData = await this.loadFromStore();
    }

    // Refresh if expiring within 5 minutes
    if (this.tokenData && this.tokenData.expiresAt < Date.now() + 300000) {
      this.tokenData = await this.refreshToken();
    }

    if (!this.tokenData) throw new Error('No valid token available');
    return this.tokenData.accessToken;
  }

  private async refreshToken(): Promise<TokenData> {
    // Prevent concurrent refresh attempts
    if (this.refreshPromise) return this.refreshPromise;

    this.refreshPromise = (async () => {
      try {
        const response = await this.refreshFn(this.tokenData!.refreshToken);
        const newData: TokenData = {
          accessToken: response.access_token,
          refreshToken: response.refresh_token || this.tokenData!.refreshToken,
          expiresAt: Date.now() + (response.expires_in * 1000)
        };
        await this.saveToStore(newData);
        return newData;
      } finally {
        this.refreshPromise = null;
      }
    })();

    return this.refreshPromise;
  }

  private async loadFromStore(): Promise<TokenData | null> {
    const data = await this.store.retrieve('tokens');
    return data ? JSON.parse(data) : null;
  }

  private async saveToStore(data: TokenData): Promise<void> {
    await this.store.store('tokens', JSON.stringify(data));
  }
}
```

---

## JWT Handling

### JWT Validation

```typescript
import { createVerify } from 'crypto';

interface JWTPayload {
  sub: string;
  exp: number;
  iat: number;
  [key: string]: any;
}

function decodeJWT(token: string): { header: any; payload: JWTPayload; signature: string } {
  const [headerB64, payloadB64, signature] = token.split('.');
  return {
    header: JSON.parse(Buffer.from(headerB64, 'base64url').toString()),
    payload: JSON.parse(Buffer.from(payloadB64, 'base64url').toString()),
    signature
  };
}

function isTokenExpired(token: string, bufferSeconds = 60): boolean {
  const { payload } = decodeJWT(token);
  return payload.exp * 1000 < Date.now() + (bufferSeconds * 1000);
}
```

### JWT Creation (for Service Auth)

```typescript
import { createSign } from 'crypto';

function createServiceJWT(
  privateKey: string,
  clientEmail: string,
  audience: string,
  scopes: string[]
): string {
  const now = Math.floor(Date.now() / 1000);

  const header = { alg: 'RS256', typ: 'JWT' };
  const payload = {
    iss: clientEmail,
    sub: clientEmail,
    aud: audience,
    scope: scopes.join(' '),
    iat: now,
    exp: now + 3600
  };

  const headerB64 = Buffer.from(JSON.stringify(header)).toString('base64url');
  const payloadB64 = Buffer.from(JSON.stringify(payload)).toString('base64url');

  const sign = createSign('RSA-SHA256');
  sign.update(`${headerB64}.${payloadB64}`);
  const signature = sign.sign(privateKey, 'base64url');

  return `${headerB64}.${payloadB64}.${signature}`;
}
```

---

## API Key Patterns

### Header-Based API Keys

```typescript
class APIKeyClient {
  constructor(
    private baseURL: string,
    private apiKey: string,
    private headerName: string = 'X-API-Key'
  ) {}

  private getHeaders(): Record<string, string> {
    return {
      [this.headerName]: this.apiKey,
      'Content-Type': 'application/json'
    };
  }

  async request<T>(endpoint: string, options: RequestInit = {}): Promise<T> {
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      ...options,
      headers: { ...this.getHeaders(), ...options.headers }
    });
    if (!response.ok) throw new Error(`API error: ${response.status}`);
    return response.json();
  }
}
```

### Common API Key Header Names

| Service | Header Name |
|---------|-------------|
| OpenAI | `Authorization: Bearer sk-...` |
| Stripe | `Authorization: Bearer sk_...` |
| SendGrid | `Authorization: Bearer SG...` |
| Ahrefs | `Authorization: Bearer ...` |
| Generic | `X-API-Key: ...` |

---

## Security Best Practices

1. **Never log tokens** - Redact in all logging
2. **Use HTTPS only** - Never send credentials over HTTP
3. **Rotate secrets regularly** - Implement key rotation
4. **Minimum scopes** - Request only needed permissions
5. **Validate tokens server-side** - Don't trust client validation
6. **Secure storage** - Use Keychain/Credential Manager, never plaintext
