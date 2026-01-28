---
name: backend-nodejs
description: Expert guidance for backend development using Node.js, Express, TypeScript, and REST/GraphQL APIs. Covers API design, authentication, middleware, error handling, database integration, and deployment. Use when building or debugging backend services, creating APIs, implementing authentication, or optimizing server performance.
allowed-tools: Read, Grep, Glob, Edit, Write
---

# Backend Node.js Development

Complete toolkit for building production-grade backend services with Node.js, Express, and TypeScript.

## Core Capabilities

### API Development
- RESTful API design and implementation
- GraphQL schema design and resolvers
- API versioning strategies
- Request/response handling
- Input validation and sanitization
- Error handling and middleware

### Authentication & Security
- JWT token management
- OAuth 2.0 / OpenID Connect
- Session management
- Password hashing (bcrypt, argon2)
- Rate limiting and throttling
- CORS configuration
- Security headers (helmet.js)
- SQL injection prevention
- XSS protection

### Database Integration
- PostgreSQL with Prisma ORM
- Connection pooling
- Transaction management
- Query optimization
- Migration strategies
- Database seeding

### Architecture Patterns
- MVC (Model-View-Controller)
- Clean Architecture / Hexagonal
- Repository pattern
- Dependency injection
- Service layer pattern
- Error boundary handling

### Performance Optimization
- Caching strategies (Redis)
- Query optimization
- Connection pooling
- Load balancing
- Horizontal scaling
- Monitoring and profiling

## Tech Stack

**Runtime:** Node.js (v18+), TypeScript
**Frameworks:** Express.js, Fastify, NestJS
**Databases:** PostgreSQL, MySQL, MongoDB
**ORMs:** Prisma, TypeORM, Sequelize
**Testing:** Jest, Supertest, Mocha
**API Docs:** OpenAPI/Swagger
**DevOps:** Docker, PM2, nginx

## Best Practices

### Code Organization
```
src/
├── controllers/    # Request handlers
├── services/       # Business logic
├── models/         # Data models
├── middleware/     # Express middleware
├── routes/         # API routes
├── utils/          # Helper functions
├── config/         # Configuration
└── types/          # TypeScript types
```

### Error Handling
```typescript
// Use centralized error handler
class AppError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message);
  }
}

// Global error middleware
app.use((err, req, res, next) => {
  const { statusCode = 500, message } = err;
  res.status(statusCode).json({
    status: 'error',
    statusCode,
    message
  });
});
```

### API Response Format
```typescript
// Consistent response structure
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
  };
  meta?: {
    page: number;
    limit: number;
    total: number;
  };
}
```

### Security Checklist
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention
- [ ] XSS protection
- [ ] CSRF tokens for stateful APIs
- [ ] Rate limiting configured
- [ ] Secure headers (helmet.js)
- [ ] Environment variables for secrets
- [ ] Password hashing with salt
- [ ] JWT secret rotation
- [ ] HTTPS enforced in production

### Testing Strategy
```typescript
// Integration tests
describe('POST /api/users', () => {
  it('should create a new user', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'test@example.com' })
      .expect(201);

    expect(response.body.data).toHaveProperty('id');
  });
});

// Unit tests
describe('UserService', () => {
  it('should hash password before saving', async () => {
    const user = await userService.create({ password: 'test123' });
    expect(user.password).not.toBe('test123');
  });
});
```

## Common Patterns

### Middleware Chain
```typescript
router.post('/api/users',
  authenticate,
  authorize(['admin']),
  validateRequest(userSchema),
  userController.create
);
```

### Database Transactions
```typescript
await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({ data: userData });
  await tx.profile.create({ data: { userId: user.id } });
  return user;
});
```

### Async Error Handling
```typescript
// Wrapper to catch async errors
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

router.get('/users', asyncHandler(async (req, res) => {
  const users = await userService.findAll();
  res.json({ success: true, data: users });
}));
```

## Performance Tips

1. **Use connection pooling** for database connections
2. **Implement caching** for frequently accessed data
3. **Use compression** middleware for responses
4. **Index database queries** appropriately
5. **Implement pagination** for large datasets
6. **Use streaming** for large file uploads/downloads
7. **Profile and monitor** with APM tools

## Debugging

```bash
# Development with auto-reload
npm run dev

# Debug mode with breakpoints
node --inspect dist/index.js

# Environment-specific logging
NODE_ENV=development npm start
```

## Deployment Checklist

- [ ] Environment variables configured
- [ ] Database migrations run
- [ ] SSL/TLS certificates installed
- [ ] Logging configured
- [ ] Monitoring/alerting setup
- [ ] Health check endpoint
- [ ] Graceful shutdown handling
- [ ] Process manager (PM2) configured
- [ ] Load balancer configured
- [ ] Backup strategy in place

## Common Commands

```bash
# Development
npm run dev
npm run build
npm run test
npm run lint

# Database
npx prisma migrate dev
npx prisma generate
npx prisma studio

# Production
npm run start:prod
pm2 start ecosystem.config.js
pm2 logs
```

## Resources

- Express.js: https://expressjs.com/
- Prisma: https://www.prisma.io/
- TypeScript: https://www.typescriptlang.org/
- Node.js Best Practices: https://github.com/goldbergyoni/nodebestpractices
