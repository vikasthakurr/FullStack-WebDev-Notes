# E2E Testing with Playwright

## What Is End-to-End (E2E) Testing?

E2E testing verifies your **entire application** works correctly by simulating real user interactions — from the browser through the frontend, API, and database. It tests the full stack together.

### Testing Pyramid

```
         /  E2E Tests  \        ← Few, slow, high confidence
        / Integration    \      ← Moderate amount
       /   Unit Tests     \     ← Many, fast, low confidence per test
      ─────────────────────
```

| Test Type   | What It Tests             | Speed  | Confidence           |
| ----------- | ------------------------- | ------ | -------------------- |
| Unit        | Single function/component | Fast   | Low (isolated)       |
| Integration | Multiple modules together | Medium | Medium               |
| E2E         | Full app as a user        | Slow   | High (real behavior) |

### Analogy

Unit tests check that each ingredient is fresh. Integration tests check that the recipe steps work. E2E tests are eating the final dish and confirming it tastes right.

---

## Why Playwright?

| Feature          | Playwright                   | Cypress                   | Selenium        |
| ---------------- | ---------------------------- | ------------------------- | --------------- |
| Cross-browser    | ✅ Chromium, Firefox, WebKit | ❌ Chromium only (mostly) | ✅ All browsers |
| Speed            | Fast (parallel by default)   | Medium                    | Slow            |
| Auto-wait        | ✅ Built-in                  | ✅ Built-in               | ❌ Manual waits |
| Multi-tab/window | ✅                           | ❌                        | ✅              |
| API testing      | ✅ Built-in                  | ✅ Plugin                 | ❌              |
| Codegen          | ✅ Record and generate       | ✅ Studio                 | ❌              |
| Language support | JS, TS, Python, Java, .NET   | JS/TS only                | Many            |

Playwright is developed by Microsoft and handles modern web patterns (SPAs, web components, iframes) well.

---

## Installation and Setup

```bash
npm init playwright@latest
```

This interactive setup:

- Installs `@playwright/test`
- Creates `playwright.config.ts`
- Creates example test files
- Optionally installs browsers

### Configuration File

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI, // Fail if .only is left in CI
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: "html",

  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry", // Collect trace on failure
    screenshot: "only-on-failure",
  },

  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
    {
      name: "firefox",
      use: { ...devices["Desktop Firefox"] },
    },
    {
      name: "webkit",
      use: { ...devices["Desktop Safari"] },
    },
    {
      name: "Mobile Chrome",
      use: { ...devices["Pixel 5"] },
    },
  ],

  // Start your dev server before running tests
  webServer: {
    command: "npm run dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
  },
});
```

### Package.json Scripts

```json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:headed": "playwright test --headed",
    "test:e2e:codegen": "playwright codegen localhost:3000"
  }
}
```

---

## Writing Your First Test

```typescript
// e2e/home.spec.ts
import { test, expect } from "@playwright/test";

test("homepage loads and shows welcome message", async ({ page }) => {
  // Navigate
  await page.goto("/");

  // Assert title
  await expect(page).toHaveTitle(/My App/);

  // Assert visible text
  await expect(page.getByRole("heading", { name: "Welcome" })).toBeVisible();
});

test("navigation works", async ({ page }) => {
  await page.goto("/");

  // Click a link
  await page.getByRole("link", { name: "About" }).click();

  // Verify URL changed
  await expect(page).toHaveURL(/.*about/);

  // Verify content loaded
  await expect(page.getByText("About Us")).toBeVisible();
});
```

### Test Structure

```typescript
import { test, expect } from "@playwright/test";

test.describe("Feature Name", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/");
  });

  test("does something", async ({ page }) => {
    // Arrange, Act, Assert
  });

  test("does something else", async ({ page }) => {
    // ...
  });
});
```

---

## Locators

Locators are Playwright's way of finding elements on the page. They are **lazy** (do not search until action) and **auto-retry** (wait for the element to appear).

### Recommended (Accessible)

```typescript
// By role (best — mirrors screen readers)
page.getByRole("button", { name: "Submit" });
page.getByRole("heading", { level: 1 });
page.getByRole("link", { name: "Home" });
page.getByRole("textbox", { name: "Email" });
page.getByRole("checkbox", { name: "Remember me" });
page.getByRole("combobox", { name: "Country" });

// By label (for form fields)
page.getByLabel("Email address");

// By text (visible text content)
page.getByText("Welcome back!");
page.getByText(/welcome/i); // regex for flexible matching

// By placeholder
page.getByPlaceholder("Search...");

// By alt text (images)
page.getByAltText("Company logo");

// By title attribute
page.getByTitle("Close modal");
```

### Test ID (Escape Hatch)

```typescript
// When accessible locators don't work
page.getByTestId("custom-calendar");
```

```html
<!-- Component must have data-testid -->
<div data-testid="custom-calendar">...</div>
```

### CSS and XPath (Last Resort)

```typescript
// CSS selector
page.locator(".card:first-child");
page.locator("#submit-btn");

// XPath
page.locator("xpath=//div[@class='container']//button");

// Chaining locators (narrow down)
page.locator(".card").filter({ hasText: "Premium" }).getByRole("button");
```

### Locator Priority

```
Best ──────────────────────────────── Worst
getByRole > getByLabel > getByText > getByTestId > CSS > XPath
```

---

## Actions

All actions auto-wait for the element to be actionable (visible, enabled, stable).

```typescript
// Click
await page.getByRole("button", { name: "Submit" }).click();
await page.getByRole("button").dblclick();
await page.getByRole("button").click({ button: "right" }); // right-click

// Fill input (clears existing value first)
await page.getByLabel("Email").fill("vikas@test.com");

// Type (simulates keystrokes one by one)
await page.getByLabel("Search").pressSequentially("hello", { delay: 100 });

// Clear input
await page.getByLabel("Email").clear();

// Check / Uncheck
await page.getByRole("checkbox", { name: "Agree" }).check();
await page.getByRole("checkbox", { name: "Agree" }).uncheck();

// Select option from dropdown
await page.getByRole("combobox", { name: "Country" }).selectOption("India");
await page.getByRole("combobox").selectOption({ label: "India" });

// Press keyboard keys
await page.getByLabel("Search").press("Enter");
await page.keyboard.press("Escape");

// Hover
await page.getByText("Menu").hover();

// Drag and drop
await page.locator("#source").dragTo(page.locator("#target"));

// Upload file
await page.getByLabel("Upload").setInputFiles("./test-data/photo.png");
```

---

## Assertions

Playwright assertions auto-retry until the condition is met (or timeout).

```typescript
// Element visibility
await expect(page.getByText("Welcome")).toBeVisible();
await expect(page.getByText("Loading")).not.toBeVisible();
await expect(page.getByRole("button")).toBeEnabled();
await expect(page.getByRole("button")).toBeDisabled();

// Text content
await expect(page.getByRole("heading")).toHaveText("Dashboard");
await expect(page.getByRole("heading")).toContainText("Dash");

// URL and title
await expect(page).toHaveURL("http://localhost:3000/dashboard");
await expect(page).toHaveURL(/dashboard/); // regex
await expect(page).toHaveTitle("My App - Dashboard");

// Count elements
await expect(page.getByRole("listitem")).toHaveCount(5);

// Input value
await expect(page.getByLabel("Email")).toHaveValue("vikas@test.com");

// CSS class / attribute
await expect(page.locator(".alert")).toHaveClass(/alert-success/);
await expect(page.getByRole("button")).toHaveAttribute("disabled", "");

// Checked state
await expect(page.getByRole("checkbox")).toBeChecked();
await expect(page.getByRole("checkbox")).not.toBeChecked();
```

### Soft Assertions (Don't Stop on Failure)

```typescript
await expect.soft(page.getByText("Title")).toBeVisible();
await expect.soft(page.getByText("Subtitle")).toBeVisible();
// Test continues even if first assertion fails — reports all failures
```

---

## Page Object Model Pattern

For large test suites, encapsulate page interactions in reusable classes.

### Page Object

```typescript
// e2e/pages/LoginPage.ts
import { Page, Locator, expect } from "@playwright/test";

export class LoginPage {
  private readonly page: Page;
  private readonly emailInput: Locator;
  private readonly passwordInput: Locator;
  private readonly submitButton: Locator;
  private readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel("Email");
    this.passwordInput = page.getByLabel("Password");
    this.submitButton = page.getByRole("button", { name: "Log in" });
    this.errorMessage = page.getByRole("alert");
  }

  async goto() {
    await this.page.goto("/login");
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async expectError(message: string) {
    await expect(this.errorMessage).toContainText(message);
  }

  async expectRedirectToDashboard() {
    await expect(this.page).toHaveURL(/.*dashboard/);
  }
}
```

### Using the Page Object

```typescript
// e2e/login.spec.ts
import { test } from "@playwright/test";
import { LoginPage } from "./pages/LoginPage";

test.describe("Login", () => {
  let loginPage: LoginPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.goto();
  });

  test("successful login redirects to dashboard", async () => {
    await loginPage.login("vikas@test.com", "password123");
    await loginPage.expectRedirectToDashboard();
  });

  test("shows error for invalid credentials", async () => {
    await loginPage.login("vikas@test.com", "wrong-password");
    await loginPage.expectError("Invalid email or password");
  });
});
```

### Benefits

- **Reusability** — same page object used in many tests
- **Maintenance** — if UI changes, update one file
- **Readability** — tests read like user stories

---

## Testing Authentication Flows

### Login Test

```typescript
// e2e/auth.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Authentication", () => {
  test("user can log in and access protected page", async ({ page }) => {
    await page.goto("/login");

    await page.getByLabel("Email").fill("vikas@test.com");
    await page.getByLabel("Password").fill("password123");
    await page.getByRole("button", { name: "Log in" }).click();

    // Verify redirect to dashboard
    await expect(page).toHaveURL(/.*dashboard/);
    await expect(page.getByText("Welcome, Vikas")).toBeVisible();
  });

  test("unauthenticated user is redirected to login", async ({ page }) => {
    await page.goto("/dashboard");

    // Should redirect to login
    await expect(page).toHaveURL(/.*login/);
  });

  test("user can log out", async ({ page }) => {
    // Log in first
    await page.goto("/login");
    await page.getByLabel("Email").fill("vikas@test.com");
    await page.getByLabel("Password").fill("password123");
    await page.getByRole("button", { name: "Log in" }).click();

    // Log out
    await page.getByRole("button", { name: "Logout" }).click();

    // Verify redirect to login page
    await expect(page).toHaveURL(/.*login/);
  });
});
```

### Reusing Auth State (Avoid Logging In Every Test)

```typescript
// e2e/auth.setup.ts
import { test as setup, expect } from "@playwright/test";

const authFile = "e2e/.auth/user.json";

setup("authenticate", async ({ page }) => {
  await page.goto("/login");
  await page.getByLabel("Email").fill("vikas@test.com");
  await page.getByLabel("Password").fill("password123");
  await page.getByRole("button", { name: "Log in" }).click();

  await expect(page).toHaveURL(/.*dashboard/);

  // Save signed-in state
  await page.context().storageState({ path: authFile });
});
```

```typescript
// playwright.config.ts
export default defineConfig({
  projects: [
    { name: "setup", testMatch: /.*\.setup\.ts/ },
    {
      name: "chromium",
      dependencies: ["setup"],
      use: {
        ...devices["Desktop Chrome"],
        storageState: "e2e/.auth/user.json", // Reuse auth
      },
    },
  ],
});
```

Now all tests start logged in — no repeated login flows.

---

## Visual Regression Testing (Screenshots)

### Full Page Screenshot Comparison

```typescript
test("homepage visual regression", async ({ page }) => {
  await page.goto("/");

  await expect(page).toHaveScreenshot("homepage.png");
});
```

First run creates a reference screenshot in `e2e/__screenshots__/`. Subsequent runs compare against it.

### Element Screenshot

```typescript
test("card component visual regression", async ({ page }) => {
  await page.goto("/products");

  const card = page.locator(".product-card").first();
  await expect(card).toHaveScreenshot("product-card.png");
});
```

### Updating Screenshots

```bash
npx playwright test --update-snapshots
```

### Configuration Options

```typescript
await expect(page).toHaveScreenshot("name.png", {
  maxDiffPixels: 100, // Allow small pixel differences
  maxDiffPixelRatio: 0.01, // Allow 1% difference
  threshold: 0.2, // Color comparison sensitivity
  animations: "disabled", // Freeze animations for consistency
  mask: [page.locator(".date")], // Mask dynamic content
});
```

### Tips for Stable Visual Tests

- Mask dynamic content (dates, times, avatars)
- Disable animations (`animations: "disabled"`)
- Use consistent viewport sizes
- Set up fonts in CI (or use `--ignore-snapshots` flag)

---

## Running Tests

### Basic Commands

```bash
# Run all tests
npx playwright test

# Run specific test file
npx playwright test e2e/login.spec.ts

# Run tests with specific title
npx playwright test -g "login"

# Run in headed mode (see the browser)
npx playwright test --headed

# Run specific browser
npx playwright test --project=chromium
npx playwright test --project=firefox

# Run in debug mode (step through)
npx playwright test --debug

# Run in UI mode (interactive)
npx playwright test --ui
```

### Parallel Execution

Playwright runs tests in parallel by default (one worker per CPU core):

```bash
# Control workers
npx playwright test --workers=4
npx playwright test --workers=1  # Sequential (for debugging)
```

### Run Single Test (During Development)

```typescript
// Focus a single test
test.only("my test", async ({ page }) => {
  // Only this test runs
});
```

### Show HTML Report

```bash
npx playwright show-report
```

---

## CI Integration (GitHub Actions)

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    timeout-minutes: 15
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright Browsers
        run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: npx playwright test

      - name: Upload test report
        uses: actions/upload-artifact@v4
        if: ${{ !cancelled() }}
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### Tips for CI

- Use `workers: 1` in CI config (containers have limited resources)
- Set `retries: 2` to handle flaky tests in CI
- Upload report as artifact for debugging failures
- Use `webServer` config to start your app automatically

---

## Trace Viewer for Debugging

The trace viewer records everything that happens during a test — screenshots at each step, DOM snapshots, network requests, and console logs.

### Enabling Traces

```typescript
// playwright.config.ts
use: {
  trace: "on-first-retry",      // Record trace only when retrying
  // trace: "on",               // Always record (more storage)
  // trace: "retain-on-failure", // Keep trace only for failed tests
}
```

### Viewing a Trace

```bash
npx playwright show-trace trace.zip
```

### What the Trace Shows

- **Timeline** — visual snapshot at each action
- **Actions** — every click, fill, navigation with timing
- **Network** — all HTTP requests/responses
- **Console** — browser console logs
- **DOM snapshot** — inspectable DOM at each point
- **Source** — which line of test code triggered each action

### Manual Trace Recording

```typescript
test("debug this test", async ({ page, context }) => {
  await context.tracing.start({ screenshots: true, snapshots: true });

  await page.goto("/");
  await page.getByRole("button").click();

  await context.tracing.stop({ path: "trace.zip" });
});
```

---

## Best Practices

1. **Use accessible locators** — `getByRole`, `getByLabel`, `getByText` over CSS selectors.
2. **Do not use hard waits** — never `page.waitForTimeout(5000)`. Use auto-waiting assertions.
3. **Keep E2E tests focused** — test critical user journeys, not every edge case.
4. **Use Page Object Model** for large suites — improves maintainability.
5. **Reuse authentication state** — log in once, share across tests via `storageState`.
6. **Test on multiple browsers** — use Playwright's projects feature.
7. **Run tests in parallel** — Playwright handles isolation per test.
8. **Use trace viewer** for debugging instead of adding `console.log`.
9. **Mask dynamic content** in visual tests — dates, animations, user avatars.
10. **Write tests that do not depend on order** — each test should start from a known state.
11. **Keep E2E tests separate from unit tests** — different folder, different run command.
12. **Use `webServer` config** — let Playwright start and stop your app automatically.

---

## Common Mistakes

| Mistake                              | Why It Is a Problem               | Fix                                              |
| ------------------------------------ | --------------------------------- | ------------------------------------------------ |
| Using `page.waitForTimeout()`        | Flaky, slow, arbitrary delays     | Use `await expect(...).toBeVisible()`            |
| Testing every small feature with E2E | Slow suite, expensive to maintain | Use E2E for critical flows only                  |
| CSS selectors like `.btn-primary`    | Break when styling changes        | Use `getByRole`, `getByText`                     |
| Not running in CI                    | E2E failures caught late          | Add to GitHub Actions                            |
| Sharing state between tests          | Flaky, order-dependent            | Each test starts from a clean state              |
| No trace on failure                  | Hard to debug CI failures         | Set `trace: "on-first-retry"`                    |
| Testing third-party widgets          | Fragile, not your responsibility  | Mock or skip external services                   |
| Logging in before every test         | Slow test suite                   | Use `storageState` to reuse auth                 |
| Ignoring mobile viewports            | Bugs only appear on mobile        | Add mobile project in config                     |
| No test data management              | Tests depend on production data   | Seed test data in `beforeEach` or use a test API |

---

## Summary

- **E2E tests** verify the full application from the user's perspective — browser, API, database together.
- **Playwright** offers cross-browser support, auto-waiting, parallel execution, and excellent debugging tools.
- Use **accessible locators** (`getByRole`, `getByLabel`) — same priority as React Testing Library.
- **Actions** (click, fill, check) auto-wait for elements to be ready — no manual sleeps needed.
- **Assertions** auto-retry until the condition passes or timeout is reached.
- Use the **Page Object Model** to organize large test suites and reduce duplication.
- **Reuse auth state** with `storageState` — log in once, share across tests.
- **Visual regression** testing catches unintended UI changes via screenshot comparison.
- Run tests in **CI (GitHub Actions)** and upload the report as an artifact for debugging.
- Use the **trace viewer** to debug failures — it records screenshots, DOM, network, and console at every step.
