# README & API Documentation

## README Best Practices

### Full Template
```markdown
# Project Name

Brief one-line description.

## Overview

2-3 paragraph explanation of what the project does and why it exists.

## Features

- Feature 1
- Feature 2
- Feature 3

## Installation

```bash
npm install project-name
```

## Quick Start

```typescript
import { feature } from 'project-name';

// Minimal example
const result = feature();
```

## Documentation

Full documentation: [Link to docs](https://docs.example.com)

- [Getting Started](docs/getting-started.md)
- [API Reference](docs/api.md)
- [Configuration](docs/configuration.md)
- [Examples](docs/examples.md)

## Development

```bash
# Install dependencies
npm install

# Run tests
npm test

# Build
npm run build
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md)

## License

MIT © [Author Name]
```

### Key Sections
1. **Project Title & Badge**: Clear name and status indicators
2. **Overview**: What, why, and for whom
3. **Installation**: Quick setup instructions
4. **Quick Start**: Minimal working example
5. **Documentation**: Links to detailed docs
6. **Development**: How to contribute/develop
7. **License**: Legal information

---

## API Documentation (OpenAPI/Swagger)

### Complete Example
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
  description: User management API

paths:
  /users:
    get:
      summary: List all users
      description: Returns a paginated list of users
      parameters:
        - name: page
          in: query
          description: Page number (1-indexed)
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          description: Items per page
          schema:
            type: integer
            default: 20
            maximum: 100
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  meta:
                    $ref: '#/components/schemas/PaginationMeta'

    post:
      summary: Create a new user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserInput'
      responses:
        '201':
          description: User created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          description: Invalid input
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
          example: 1
        email:
          type: string
          format: email
          example: user@example.com
        name:
          type: string
          example: John Doe
        createdAt:
          type: string
          format: date-time
```

---

## JSDoc/TSDoc

### Function Documentation
```typescript
/**
 * Creates a new user in the system.
 *
 * @param userData - The user data to create
 * @param userData.email - User's email address (must be unique)
 * @param userData.name - User's full name
 * @param userData.password - User's password (will be hashed)
 * @returns Promise resolving to the created user (password excluded)
 * @throws {ValidationError} If email format is invalid
 * @throws {DuplicateError} If email already exists
 *
 * @example
 * ```typescript
 * const user = await createUser({
 *   email: 'john@example.com',
 *   name: 'John Doe',
 *   password: 'secure123'
 * });
 * console.log(user.id); // 1
 * ```
 */
export async function createUser(userData: CreateUserInput): Promise<User> {
  // Implementation
}
```

### Comprehensive Example
```typescript
/**
 * Calculates shipping cost based on weight and destination.
 *
 * Uses USPS Ground rates for domestic shipping and DHL for international.
 * Rates are cached for 24 hours to reduce API calls.
 *
 * @param weight - Package weight in pounds
 * @param destination - Destination country code (ISO 3166-1 alpha-2)
 * @param expedited - Whether to use expedited shipping
 * @returns Shipping cost in USD
 * @throws {InvalidWeightError} If weight is negative or over 150 lbs
 * @throws {UnsupportedCountryError} If destination is not supported
 *
 * @see https://docs.example.com/shipping-calculation
 *
 * @example
 * Basic usage:
 * ```typescript
 * const cost = await calculateShipping(5, 'US');
 * console.log(cost); // 12.50
 * ```
 *
 * @example
 * Expedited international:
 * ```typescript
 * const cost = await calculateShipping(10, 'GB', true);
 * console.log(cost); // 45.00
 * ```
 */
export async function calculateShipping(
  weight: number,
  destination: string,
  expedited = false
): Promise<number> {
  // Implementation
}
```

---

## Inline Code Documentation

### Comments Best Practices
```typescript
// ✅ Good: Explains WHY, not WHAT
// Use exponential backoff to avoid overwhelming the API during high load
const delay = Math.pow(2, retryCount) * 1000;

// ❌ Bad: States the obvious
// Multiply 2 by retry count and 1000
const delay = Math.pow(2, retryCount) * 1000;

// ✅ Good: Explains business logic
// Discount applies only to orders over $100 and for premium users
if (order.total > 100 && user.isPremium) {
  applyDiscount(order);
}

// ✅ Good: Documents workaround
// WORKAROUND: API returns null instead of empty array (Bug #1234)
const items = response.data?.items ?? [];

// ✅ Good: Marks TODO with context
// TODO(username): Optimize this query once we have proper indexes (see DB-123)
const users = await db.users.findMany();
```
