# End-to-End Testing with Playwright

> **Last reviewed**: 2026-01-06


## Week Thirteen: The Checkout That Broke at Midnight

The checkout flow passed unit tests and integration tests, yet a production hotfix went out at midnight. The payment modal never opened on Safari.

"This is why you need E2E," Sarah says. "Unit tests protect logic. Integration tests protect components. E2E protects the real user journey."

Marcus adds, "Think of E2E as a smoke alarm. You don't want it to go off every day, but you want it there when it counts."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **E2E test** | A rehearsal - the full performance, start to finish |
| **Locator** | A spotlight - find the exact element on stage |
| **Fixture data** | Props and costumes - controlled inputs for repeatable scenes |
| **Flake control** | Stage timing - wait for cues, not guesses |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Mid-Level Module 08 (Testing Strategies) - Comfort with integration testing and test structure.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Set up Playwright and run E2E tests locally
- [ ] Write stable locators for user flows
- [ ] Seed and reset test data for repeatable runs
- [ ] Handle authentication in E2E tests
- [ ] Reduce flakiness with smart waits and assertions
- [ ] Run E2E tests in CI with artifacts

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 4-5 hours
- **Mastery**: Build E2E coverage over 4-6 weeks

---

## Chapter 1: The First E2E Test

Sarah starts with the simplest flow: open the app and see the dashboard.

```ts
import { test, expect } from '@playwright/test';

test('loads the dashboard', async ({ page }) => {
  await page.goto('http://localhost:5173');
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
});
```

"If this fails, the app is down. That's the whole point," Marcus says.

---

## Chapter 2: Locators That Survive Refactors

Fragile selectors make tests brittle.

"Pick selectors that match the user view," Marcus says.

"Prefer role and accessible names," Sarah says.

```ts
await page.getByRole('button', { name: 'Add to cart' }).click();
await expect(page.getByRole('status')).toHaveText('Added');
```

"Roles survive CSS changes. Classes do not."

---

## Chapter 3: Data Setup and Reset

The checkout test needs a known cart state.

"Reset data every run or chase ghosts," Sarah says.

```ts
test.beforeEach(async ({ request }) => {
  await request.post('/test/reset');
  await request.post('/test/seed', { data: { cart: ['sku-1'] } });
});
```

"These requests use the Playwright `baseURL`, so set it once in your config."

```ts
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: { baseURL: 'http://127.0.0.1:5173' },
});
```

"Keep `/test/*` endpoints behind auth or only in test environments."

"Treat test data like fixtures in a studio," Marcus says. "Controlled and repeatable."

---

## Chapter 4: Auth Without Pain

Login flows slow tests. You can reuse state.

"Reuse auth state so tests stay fast," Marcus says.

```ts
test.use({ storageState: 'storage/auth.json' });

test('opens account page', async ({ page }) => {
  await page.goto('/account');
  await expect(page.getByRole('heading', { name: 'Account' })).toBeVisible();
});
```

"Authenticate once, reuse the session," Sarah says.

---

## Chapter 5: Reducing Flakiness

Flaky tests destroy trust.

"If it flakes, it fails," Sarah says.

```ts
await page.getByRole('button', { name: 'Pay' }).click();
await expect(page.getByRole('status')).toHaveText('Payment complete');
```

"Avoid `waitForTimeout`. Wait for the UI you care about."

---

## Chapter 6: CI Runs and Artifacts

You wire tests into CI with screenshots and traces. Let Playwright start the dev server so your pipeline doesn't hang.

"Artifacts tell the story," Marcus says.

```ts
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: { baseURL: 'http://127.0.0.1:5173' },
  webServer: {
    command: 'npm run dev -- --host',
    url: 'http://127.0.0.1:5173',
    reuseExistingServer: !process.env.CI,
  },
});
```

```yaml
- name: Run Playwright
  run: npx playwright test
- name: Upload artifacts
  uses: actions/upload-artifact@v4
  with:
    name: playwright-report
    path: playwright-report
```

"Artifacts turn failures into evidence," Marcus says.

---

## Common Mistakes

1. **Testing too many flows** - Focus on critical paths first.
2. **Using brittle selectors** - Prefer role-based locators.
3. **Relying on timeouts** - Wait for UI state, not time.
4. **Ignoring data resets** - Tests should start clean every run.


Example:
```ts
// ❌ Brittle selector
await page.click('.btn-primary');

// ✅ Accessible locator
await page.getByRole('button', { name: 'Checkout' }).click();
```

---

## Practice Exercises

1. Write a signup flow test from start to finish
2. Create a test that seeds data and verifies a list view
3. Add a CI job that runs E2E tests on pull requests

### Solutions

<details>
<summary>Exercise 1: Signup Flow</summary>

```ts
test('signs up a new user', async ({ page }) => {
  await page.goto('/signup');
  await page.getByLabel('Email').fill('new@user.com');
  await page.getByLabel('Password').fill('safe-password');
  await page.getByRole('button', { name: 'Create account' }).click();
  await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible();
});
```

**Key points:**
- Fill fields the way a user does.
- Assert on visible outcomes.
- Keep the flow short and focused.

</details>

<details>
<summary>Exercise 2: Seeded List View</summary>

```ts
test.beforeEach(async ({ request }) => {
  await request.post('/test/reset');
  await request.post('/test/seed', { data: { items: ['A', 'B'] } });
});

test('shows seeded items', async ({ page }) => {
  await page.goto('/items');
  await expect(page.getByText('A')).toBeVisible();
  await expect(page.getByText('B')).toBeVisible();
});
```

**Key points:**
- Reset state before each test.
- Seed only what you need.
- Verify UI output, not internals.

</details>

<details>
<summary>Exercise 3: CI Job</summary>

```yaml
name: E2E
on: [pull_request]

jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
```

**Key points:**
- Install browser deps in CI.
- Start the app with Playwright `webServer`.
- Run in a clean environment for confidence.

</details>

---

## What You Learned

This module covered:

- **E2E scope**: Protect real user journeys
- **Locators**: Stable selectors with roles
- **Data**: Seed and reset for repeatability
- **Auth**: Reuse storage state for speed
- **Flake control**: Wait for UI, not time
- **CI**: Capture artifacts for fast debugging

**Key takeaway**: E2E tests are expensive but critical when they cover the right flows.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add a checkout smoke test that runs on every PR
- Seed database state for a complex list view
- Reduce flaky tests by switching to role-based locators
- Record Playwright traces for hard-to-repro bugs
- Gate releases on a small E2E suite

---

## Further Reading

- [Playwright Docs](https://playwright.dev/docs/intro)
- [Playwright Locators](https://playwright.dev/docs/locators)
- [Test Isolation](https://playwright.dev/docs/test-isolation)

---

**Navigation**: [← Previous Module](./12-nextjs-fundamentals.md) | [Next Module →](./14-animations-and-motion.md)
