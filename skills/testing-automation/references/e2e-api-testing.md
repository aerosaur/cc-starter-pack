# E2E & API Testing

## Playwright

### Test Structure
```typescript
import { test, expect } from '@playwright/test';

test.describe('User Authentication', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('successful login redirects to dashboard', async ({ page }) => {
    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('h1')).toHaveText('Welcome');
  });

  test('invalid credentials show error', async ({ page }) => {
    await page.fill('[data-testid="email"]', 'wrong@example.com');
    await page.fill('[data-testid="password"]', 'wrongpass');
    await page.click('button[type="submit"]');

    await expect(page.locator('.error-message')).toBeVisible();
    await expect(page.locator('.error-message')).toContainText('Invalid credentials');
  });
});
```

### Configuration
```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [['html'], ['list']],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    { name: 'mobile', use: { ...devices['iPhone 13'] } }
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI
  }
});
```

### Locator Strategies
```typescript
// Recommended: Role-based
page.getByRole('button', { name: 'Submit' })
page.getByRole('link', { name: /learn more/i })
page.getByRole('textbox', { name: 'Email' })

// Text-based
page.getByText('Welcome back')
page.getByLabel('Email address')
page.getByPlaceholder('Enter your email')

// Test IDs (fallback)
page.getByTestId('submit-button')

// CSS/XPath (avoid if possible)
page.locator('.submit-btn')
page.locator('//button[@type="submit"]')
```

### Assertions
```typescript
// Visibility
await expect(locator).toBeVisible();
await expect(locator).toBeHidden();

// Text
await expect(locator).toHaveText('Expected text');
await expect(locator).toContainText('partial');

// Attributes
await expect(locator).toHaveAttribute('href', '/about');
await expect(locator).toHaveClass(/active/);

// State
await expect(locator).toBeEnabled();
await expect(locator).toBeDisabled();
await expect(locator).toBeChecked();

// Page
await expect(page).toHaveURL('/dashboard');
await expect(page).toHaveTitle(/Dashboard/);

// Count
await expect(locator).toHaveCount(5);
```

### Waiting Strategies
```typescript
// Auto-waiting (built-in)
await page.click('button'); // waits for actionable

// Explicit waits
await page.waitForSelector('.loaded');
await page.waitForURL('/success');
await page.waitForLoadState('networkidle');
await page.waitForResponse('/api/data');

// Custom wait
await page.waitForFunction(() => window.appLoaded === true);
```

---

## Page Object Pattern

### Page Object Class
```typescript
// pages/LoginPage.ts
import { Page, Locator, expect } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Sign In' });
    this.errorMessage = page.locator('[data-testid="error"]');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async expectError(message: string) {
    await expect(this.errorMessage).toContainText(message);
  }
}
```

### Using Page Objects
```typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';
import { DashboardPage } from './pages/DashboardPage';

test.describe('Authentication Flow', () => {
  test('login and view dashboard', async ({ page }) => {
    const loginPage = new LoginPage(page);
    const dashboardPage = new DashboardPage(page);

    await loginPage.goto();
    await loginPage.login('user@example.com', 'password123');

    await expect(page).toHaveURL('/dashboard');
    await dashboardPage.expectWelcomeMessage('Welcome, User');
  });
});
```

### Fixtures
```typescript
// fixtures.ts
import { test as base } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

type MyFixtures = {
  loginPage: LoginPage;
  authenticatedPage: Page;
};

export const test = base.extend<MyFixtures>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await use(loginPage);
  },

  authenticatedPage: async ({ page }, use) => {
    await page.goto('/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'password');
    await page.click('button[type="submit"]');
    await page.waitForURL('/dashboard');
    await use(page);
  }
});

// Usage
test('uses authenticated page', async ({ authenticatedPage }) => {
  await authenticatedPage.goto('/settings');
  // already logged in
});
```

---

## API Testing with Supertest

### Basic Setup
```typescript
import request from 'supertest';
import app from '../src/app';

describe('Users API', () => {
  describe('GET /api/users', () => {
    it('returns list of users', async () => {
      const response = await request(app)
        .get('/api/users')
        .expect('Content-Type', /json/)
        .expect(200);

      expect(response.body).toHaveLength(3);
      expect(response.body[0]).toHaveProperty('id');
      expect(response.body[0]).toHaveProperty('email');
    });
  });

  describe('POST /api/users', () => {
    it('creates new user', async () => {
      const userData = {
        email: 'new@example.com',
        name: 'New User'
      };

      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .set('Authorization', `Bearer ${token}`)
        .expect(201);

      expect(response.body).toMatchObject(userData);
      expect(response.body).toHaveProperty('id');
    });

    it('validates required fields', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({})
        .set('Authorization', `Bearer ${token}`)
        .expect(400);

      expect(response.body.errors).toContainEqual(
        expect.objectContaining({ field: 'email' })
      );
    });
  });
});
```

### Authentication Testing
```typescript
describe('Authentication', () => {
  let authToken: string;

  beforeAll(async () => {
    const response = await request(app)
      .post('/api/auth/login')
      .send({ email: 'test@example.com', password: 'password' });
    authToken = response.body.token;
  });

  it('rejects unauthenticated requests', async () => {
    await request(app)
      .get('/api/protected')
      .expect(401);
  });

  it('accepts valid token', async () => {
    await request(app)
      .get('/api/protected')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);
  });

  it('rejects expired token', async () => {
    const expiredToken = generateExpiredToken();
    await request(app)
      .get('/api/protected')
      .set('Authorization', `Bearer ${expiredToken}`)
      .expect(401);
  });
});
```

### Database Integration
```typescript
import { db } from '../src/database';

describe('Users API with DB', () => {
  beforeAll(async () => {
    await db.migrate.latest();
  });

  beforeEach(async () => {
    await db.seed.run();
  });

  afterAll(async () => {
    await db.destroy();
  });

  it('creates user in database', async () => {
    const userData = { email: 'test@example.com', name: 'Test' };

    const response = await request(app)
      .post('/api/users')
      .send(userData)
      .expect(201);

    const dbUser = await db('users').where({ id: response.body.id }).first();
    expect(dbUser.email).toBe(userData.email);
  });
});
```

---

## Visual Testing

### Playwright Screenshots
```typescript
test('visual regression', async ({ page }) => {
  await page.goto('/dashboard');

  // Full page
  await expect(page).toHaveScreenshot('dashboard.png');

  // Specific element
  const card = page.locator('.user-card');
  await expect(card).toHaveScreenshot('user-card.png');

  // With options
  await expect(page).toHaveScreenshot('dashboard-full.png', {
    fullPage: true,
    maxDiffPixelRatio: 0.1
  });
});
```

### Update Snapshots
```bash
# Update all snapshots
npx playwright test --update-snapshots

# Update specific test
npx playwright test dashboard.spec.ts --update-snapshots
```
