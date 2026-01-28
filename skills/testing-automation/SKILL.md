---
name: testing-automation
description: Expert guidance for automated testing including unit tests, integration tests, E2E tests, and test-driven development. Covers Jest, Vitest, Playwright, Cypress, React Testing Library, and API testing with Supertest. Use when writing tests, setting up test infrastructure, debugging test failures, or implementing TDD/BDD practices.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Testing Automation

Complete toolkit for automated testing across the entire stack: unit tests, integration tests, E2E tests, and performance tests.

## When to Use This Skill

- Writing unit tests for functions/components
- Setting up E2E testing with Playwright
- Testing React components with Testing Library
- API testing with Supertest
- Implementing test coverage requirements
- Debugging flaky or failing tests
- Setting up CI/CD test pipelines

## Tech Stack

| Category | Tools |
|----------|-------|
| **Unit Testing** | Jest, Vitest, Mocha, Chai |
| **React Testing** | React Testing Library, Enzyme |
| **E2E Testing** | Playwright, Cypress, Puppeteer |
| **API Testing** | Supertest, Postman, Newman |
| **Mocking** | jest.mock, MSW (Mock Service Worker) |
| **Coverage** | Istanbul, c8 |
| **CI/CD** | GitHub Actions, GitLab CI, CircleCI |

## Core Principles

### AAA Pattern
```typescript
it('should create user', async () => {
  // Arrange
  const userData = { email: 'test@example.com' };

  // Act
  const user = await userService.create(userData);

  // Assert
  expect(user).toHaveProperty('id');
});
```

### What to Test
- ✅ Public API/interface
- ✅ Edge cases and boundaries
- ✅ Error conditions
- ✅ Business logic
- ❌ Implementation details
- ❌ Third-party libraries
- ❌ Trivial getters/setters

### Naming Convention
```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid data', () => {});
    it('should throw error when email exists', () => {});
  });
});
```

## Quick Reference

### Common Commands
```bash
# Unit tests
npm test
npm run test:watch
npm run test:coverage

# E2E tests
npx playwright test
npx playwright test --ui
npx playwright test --debug

# Run specific test
npm test -- user.test.ts
npm test -- --testNamePattern="should create"
```

### Jest Matchers
```typescript
expect(value).toBe(exact)
expect(value).toEqual(deepEqual)
expect(array).toContain(item)
expect(object).toHaveProperty('key')
expect(fn).toThrow(Error)
expect(fn).toHaveBeenCalledWith(args)
```

## Detailed Documentation

- [Unit Testing](references/unit-testing.md) - Jest/Vitest, mocking, fixtures, React Testing Library
- [E2E & API Testing](references/e2e-api-testing.md) - Playwright, Supertest, Page Objects
- [Best Practices & CI](references/best-practices-ci.md) - Coverage, CI/CD, debugging
