# Best Practices & CI/CD

## Coverage Configuration

### Jest Coverage
```javascript
// jest.config.js
module.exports = {
  collectCoverage: true,
  collectCoverageFrom: [
    'src/**/*.{js,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/index.ts',
    '!src/**/*.stories.tsx',
    '!src/test/**/*'
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html', 'json-summary'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/critical/': {
      branches: 95,
      functions: 95,
      lines: 95
    }
  }
};
```

### Vitest Coverage
```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8', // or 'istanbul'
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'test/',
        '**/*.d.ts',
        '**/*.test.ts'
      ],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 80,
        statements: 80
      }
    }
  }
});
```

### Running Coverage
```bash
# Jest
npm test -- --coverage
npm test -- --coverage --coverageReporters="text-summary"

# Vitest
npx vitest --coverage
npx vitest --coverage --reporter=verbose
```

---

## GitHub Actions

### Unit Tests Workflow
```yaml
# .github/workflows/test.yml
name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true
```

### E2E Tests Workflow
```yaml
# .github/workflows/e2e.yml
name: E2E Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  e2e:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Build application
        run: npm run build

      - name: Run E2E tests
        run: npx playwright test

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### Parallel Test Execution
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npx playwright install --with-deps

      - name: Run tests (shard ${{ matrix.shard }}/4)
        run: npx playwright test --shard=${{ matrix.shard }}/4
```

---

## Debugging Tests

### Jest Debugging
```bash
# Run single test file
npm test -- user.test.ts

# Run tests matching pattern
npm test -- --testNamePattern="should create user"

# Run in watch mode
npm test -- --watch

# Debug with Node inspector
node --inspect-brk node_modules/.bin/jest --runInBand

# Verbose output
npm test -- --verbose

# Show individual test results
npm test -- --reporters=default --reporters=jest-junit
```

### Playwright Debugging
```bash
# Debug mode (opens inspector)
npx playwright test --debug

# UI mode (interactive)
npx playwright test --ui

# Headed mode (see browser)
npx playwright test --headed

# Slow motion
npx playwright test --headed --slowmo=500

# Specific test
npx playwright test tests/login.spec.ts

# Specific test by title
npx playwright test -g "should login"

# Generate trace
npx playwright test --trace on

# View trace
npx playwright show-trace trace.zip
```

### Console Debugging
```typescript
// Jest
it('debug test', () => {
  console.log('Debug:', variable);
  debugger; // works with --inspect-brk
});

// Playwright
test('debug test', async ({ page }) => {
  await page.pause(); // opens inspector
  console.log(await page.content());
});
```

---

## Testing Best Practices

### Test Organization
```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx      # Unit tests
│   │   └── Button.stories.tsx
│   └── Form/
│       ├── Form.tsx
│       └── Form.test.tsx
├── services/
│   ├── api.ts
│   └── api.test.ts
└── utils/
    ├── format.ts
    └── format.test.ts

e2e/
├── auth.spec.ts
├── checkout.spec.ts
└── pages/
    ├── LoginPage.ts
    └── CheckoutPage.ts
```

### Test Isolation
```typescript
// ❌ Bad: Tests depend on each other
let userId: number;
it('creates user', async () => {
  userId = await createUser();
});
it('updates user', async () => {
  await updateUser(userId); // depends on previous test
});

// ✅ Good: Each test is independent
it('creates user', async () => {
  const userId = await createUser();
  expect(userId).toBeDefined();
});
it('updates user', async () => {
  const userId = await createUser(); // creates own data
  await updateUser(userId);
});
```

### Deterministic Tests
```typescript
// ❌ Bad: Non-deterministic
it('sorts by date', () => {
  const items = [{ date: new Date() }];
  expect(sort(items)).toEqual([...]); // fails randomly
});

// ✅ Good: Fixed dates
it('sorts by date', () => {
  const items = [
    { date: new Date('2024-01-15') },
    { date: new Date('2024-01-10') }
  ];
  expect(sort(items)[0].date).toEqual(new Date('2024-01-10'));
});

// Mock Date when needed
beforeEach(() => {
  jest.useFakeTimers();
  jest.setSystemTime(new Date('2024-01-01'));
});
afterEach(() => {
  jest.useRealTimers();
});
```

### Flaky Test Prevention
```typescript
// ❌ Bad: Race condition
test('loads data', async ({ page }) => {
  await page.goto('/dashboard');
  expect(await page.textContent('.data')).toBe('loaded');
});

// ✅ Good: Wait for condition
test('loads data', async ({ page }) => {
  await page.goto('/dashboard');
  await expect(page.locator('.data')).toHaveText('loaded');
});

// ❌ Bad: Fixed timeout
await page.waitForTimeout(2000);

// ✅ Good: Wait for specific condition
await page.waitForSelector('.loaded');
await expect(page.locator('.spinner')).not.toBeVisible();
```

### Test Naming
```typescript
// ✅ Good: Describes behavior
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid email', () => {});
    it('should throw error when email already exists', () => {});
    it('should hash password before storing', () => {});
  });
});

// Pattern: should [action] when [condition]
it('should return empty array when no users exist', () => {});
it('should throw ValidationError when email is invalid', () => {});
```

---

## Test Patterns

### Snapshot Testing
```typescript
// Component snapshot
it('renders correctly', () => {
  const { container } = render(<Button>Click me</Button>);
  expect(container).toMatchSnapshot();
});

// Inline snapshot
it('formats date', () => {
  expect(formatDate(new Date('2024-01-15'))).toMatchInlineSnapshot(
    `"January 15, 2024"`
  );
});

// Update snapshots
// npm test -- -u
```

### Error Testing
```typescript
// Sync errors
it('throws on invalid input', () => {
  expect(() => validate(null)).toThrow('Input required');
  expect(() => validate(null)).toThrow(ValidationError);
});

// Async errors
it('rejects with error', async () => {
  await expect(fetchUser(-1)).rejects.toThrow('User not found');
});

// Error properties
it('includes error details', async () => {
  try {
    await fetchUser(-1);
    fail('Should have thrown');
  } catch (error) {
    expect(error.code).toBe('NOT_FOUND');
    expect(error.statusCode).toBe(404);
  }
});
```

### Parameterized Tests
```typescript
// Jest each
it.each([
  ['hello', 'HELLO'],
  ['world', 'WORLD'],
  ['', '']
])('toUpperCase(%s) returns %s', (input, expected) => {
  expect(toUpperCase(input)).toBe(expected);
});

// With objects
it.each([
  { input: 1, expected: true },
  { input: 0, expected: false },
  { input: -1, expected: false }
])('isPositive($input) returns $expected', ({ input, expected }) => {
  expect(isPositive(input)).toBe(expected);
});

// Table syntax
it.each`
  a    | b    | sum
  ${1} | ${2} | ${3}
  ${2} | ${3} | ${5}
`('$a + $b = $sum', ({ a, b, sum }) => {
  expect(add(a, b)).toBe(sum);
});
```
