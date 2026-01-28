---
name: api-client-dev
description: Expert guidance for building REST API clients, handling authentication (OAuth, JWT, API keys), implementing retry logic, rate limiting, and error handling. Use when integrating with third-party APIs (Slack, Spotify, GitHub, Google Calendar, Ahrefs), creating API wrappers, implementing authentication flows, or building robust API communication layers. Covers TypeScript/JavaScript patterns, HTTP clients, and API design.
allowed-tools: Read, Grep, Glob, Edit, Write
---

# API Client Development

Comprehensive toolkit for building robust API clients with proper authentication, error handling, rate limiting, and retry strategies.

## When to Use This Skill

- Integrating with third-party APIs (Slack, GitHub, Spotify, etc.)
- Building API client libraries or SDKs
- Implementing OAuth flows
- Creating MCP servers with API integrations
- Handling complex authentication requirements
- Implementing retry and rate limiting logic

## Quick Start

### Basic API Client

```typescript
class APIClient {
  constructor(private baseURL: string, private apiKey?: string) {}

  async get<T>(endpoint: string, params?: Record<string, any>): Promise<T> {
    const url = new URL(endpoint, this.baseURL);
    if (params) {
      Object.entries(params).forEach(([k, v]) =>
        v != null && url.searchParams.append(k, String(v))
      );
    }
    const response = await fetch(url, {
      headers: {
        'Content-Type': 'application/json',
        ...(this.apiKey && { 'Authorization': `Bearer ${this.apiKey}` })
      }
    });
    if (!response.ok) throw new Error(`${response.status}: ${response.statusText}`);
    return response.json();
  }

  async post<T>(endpoint: string, data: any): Promise<T> {
    const response = await fetch(new URL(endpoint, this.baseURL), {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        ...(this.apiKey && { 'Authorization': `Bearer ${this.apiKey}` })
      },
      body: JSON.stringify(data)
    });
    if (!response.ok) throw new Error(`${response.status}: ${response.statusText}`);
    return response.json();
  }
}
```

## Core Capabilities

| Category | Features |
|----------|----------|
| **Authentication** | OAuth 2.0, JWT, API keys, Bearer tokens, Token refresh |
| **Error Handling** | HTTP codes, Retry with backoff, Circuit breaker, Timeouts |
| **Rate Limiting** | Token bucket, Request queuing, Header parsing, Adaptive limiting |
| **Resilience** | Graceful degradation, Network recovery, Fallbacks |

## Best Practices

### Authentication
- Store tokens securely (Keychain on macOS)
- Implement automatic token refresh
- Handle token expiration gracefully
- Never log sensitive credentials

### Error Handling
- Categorize errors (network, auth, validation, server)
- Use exponential backoff for retries
- Handle rate limiting (429) specially
- Timeout long-running requests

### Rate Limiting
- Parse X-RateLimit-* headers
- Implement request queuing
- Use adaptive rate limiting
- Monitor API usage

## Detailed Documentation

- [Authentication Patterns](references/authentication.md) - OAuth, JWT, token management
- [Error Handling & Retry](references/error-retry.md) - Backoff, circuit breakers
- [Rate Limiting](references/rate-limiting.md) - Token bucket, queuing
