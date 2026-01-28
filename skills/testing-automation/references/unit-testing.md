# Unit Testing

## Jest/Vitest Configuration

### Jest Setup
```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/*.test.ts', '**/*.spec.ts'],
  collectCoverageFrom: ['src/**/*.ts', '!src/**/*.d.ts'],
  coverageThreshold: {
    global: { branches: 80, functions: 80, lines: 80 }
  },
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1'
  }
};
```

### Vitest Setup
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    include: ['**/*.{test,spec}.{js,ts}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'dist/']
    }
  }
});
```

---

## Mocking Strategies

### Module Mocking
```typescript
// Mock entire module
jest.mock('./database', () => ({
  query: jest.fn(),
  connect: jest.fn().mockResolvedValue(true)
}));

// Mock with factory
jest.mock('./config', () => {
  return {
    __esModule: true,
    default: { apiUrl: 'http://test.com' },
    getConfig: jest.fn(() => ({ debug: true }))
  };
});

// Partial mock
jest.mock('./utils', () => ({
  ...jest.requireActual('./utils'),
  formatDate: jest.fn(() => '2024-01-01')
}));
```

### Spy and Stub
```typescript
// Spy on method
const spy = jest.spyOn(userService, 'save');
await userService.save(userData);
expect(spy).toHaveBeenCalledWith(userData);

// Stub implementation
jest.spyOn(api, 'fetch').mockImplementation(async () => ({
  data: mockData,
  status: 200
}));

// Mock once vs always
mockFn.mockReturnValueOnce('first').mockReturnValue('default');
```

### Mock Service Worker (MSW)
```typescript
import { setupServer } from 'msw/node';
import { rest } from 'msw';

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json([{ id: 1, name: 'Test' }]));
  }),
  rest.post('/api/users', async (req, res, ctx) => {
    const body = await req.json();
    return res(ctx.status(201), ctx.json({ id: 2, ...body }));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Override for specific test
it('handles error', async () => {
  server.use(
    rest.get('/api/users', (req, res, ctx) => {
      return res(ctx.status(500));
    })
  );
  // test error handling
});
```

---

## Test Fixtures

### Factory Pattern
```typescript
// factories/user.factory.ts
interface UserProps {
  id?: number;
  email?: string;
  role?: 'admin' | 'user';
}

export const createUser = (overrides: UserProps = {}) => ({
  id: 1,
  email: 'test@example.com',
  role: 'user' as const,
  createdAt: new Date('2024-01-01'),
  ...overrides
});

// Usage
const admin = createUser({ role: 'admin' });
const users = [createUser({ id: 1 }), createUser({ id: 2 })];
```

### Builder Pattern
```typescript
class UserBuilder {
  private user: Partial<User> = {};

  withId(id: number) {
    this.user.id = id;
    return this;
  }

  withEmail(email: string) {
    this.user.email = email;
    return this;
  }

  asAdmin() {
    this.user.role = 'admin';
    return this;
  }

  build(): User {
    return {
      id: 1,
      email: 'default@test.com',
      role: 'user',
      ...this.user
    } as User;
  }
}

// Usage
const user = new UserBuilder().withEmail('custom@test.com').asAdmin().build();
```

---

## React Testing Library

### Component Testing
```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('LoginForm', () => {
  it('submits form with credentials', async () => {
    const onSubmit = jest.fn();
    const user = userEvent.setup();

    render(<LoginForm onSubmit={onSubmit} />);

    // Query elements
    const emailInput = screen.getByLabelText(/email/i);
    const passwordInput = screen.getByLabelText(/password/i);
    const submitBtn = screen.getByRole('button', { name: /sign in/i });

    // Interact
    await user.type(emailInput, 'test@example.com');
    await user.type(passwordInput, 'password123');
    await user.click(submitBtn);

    // Assert
    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123'
      });
    });
  });

  it('shows validation error', async () => {
    render(<LoginForm onSubmit={jest.fn()} />);

    await userEvent.click(screen.getByRole('button', { name: /sign in/i }));

    expect(await screen.findByText(/email is required/i)).toBeInTheDocument();
  });
});
```

### Query Priority
```typescript
// ✅ Accessible queries (preferred)
screen.getByRole('button', { name: /submit/i })
screen.getByLabelText(/email address/i)
screen.getByPlaceholderText(/search/i)
screen.getByText(/welcome/i)
screen.getByAltText(/profile/i)

// 🟡 Semantic queries
screen.getByTitle(/close/i)

// 🔴 Test IDs (last resort)
screen.getByTestId('custom-element')
```

### Hook Testing
```typescript
import { renderHook, act, waitFor } from '@testing-library/react';

describe('useCounter', () => {
  it('increments counter', () => {
    const { result } = renderHook(() => useCounter(0));

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });
});

describe('useApi', () => {
  it('fetches data', async () => {
    const { result } = renderHook(() => useApi('/users'));

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.data).toEqual(expectedData);
  });
});
```

### Context Wrapper
```typescript
const AllProviders = ({ children }) => (
  <ThemeProvider theme={testTheme}>
    <AuthProvider value={mockAuth}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </AuthProvider>
  </ThemeProvider>
);

const customRender = (ui, options = {}) =>
  render(ui, { wrapper: AllProviders, ...options });

// Usage
customRender(<UserProfile />);
```

---

## Async Testing

### Waiting for Elements
```typescript
// Wait for element to appear
const element = await screen.findByText(/loaded/i);

// Wait for condition
await waitFor(() => {
  expect(screen.getByText(/success/i)).toBeInTheDocument();
});

// Wait for element to disappear
await waitForElementToBeRemoved(() => screen.queryByText(/loading/i));

// Custom timeout
await waitFor(() => expect(mockFn).toHaveBeenCalled(), { timeout: 3000 });
```

### Testing Async Functions
```typescript
it('handles async operation', async () => {
  const result = await asyncFunction();
  expect(result).toBe(expected);
});

it('rejects with error', async () => {
  await expect(asyncFunction()).rejects.toThrow('Error message');
});

it('resolves eventually', async () => {
  await expect(asyncFunction()).resolves.toEqual({ success: true });
});
```
