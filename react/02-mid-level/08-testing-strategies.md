# Testing Strategies

> **Last reviewed**: 2026-01-06


## Week Eight: The Quality Push

The team is shipping faster, but bugs are slipping through. Sarah wants more confidence before releases, and Marcus points out that unit tests alone aren't catching integration issues. You need a strategy that balances speed, coverage, and reliability. This week is about choosing the right mix of tests, writing meaningful integration tests, and wiring them into CI so quality keeps up with delivery.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Testing pyramid** | A budget - spend more where it pays off |
| **Integration tests** | Dress rehearsals - full flow with real components |
| **Mocking** | Stunt doubles - stand in for external systems |
| **CI pipelines** | Airport security - checks before boarding |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 07 (Internationalization) - Comfort with component composition and async data.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Design a testing strategy aligned with risk
- [ ] Write integration tests that mirror real user flows
- [ ] Mock APIs and external services safely
- [ ] Measure coverage without chasing vanity metrics
- [ ] Configure CI pipelines to enforce testing standards
- [ ] Test hooks and async behaviors reliably

---

## Time Estimate

- **Reading**: 60-90 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice testing strategies over 4-6 weeks

---

## Chapter 1: Testing Pyramid

You need a shared testing strategy before you write another test file.

```
         /\
        /  \
       / E2E\        Few, slow, expensive
      /------\
     /        \
    /Integration\    More, medium speed
   /--------------\
  /                \
 /      Unit        \  Many, fast, cheap
/--------------------\
```

**React testing typically inverts this:**
- Focus on integration tests (component + behavior)
- Use E2E for critical paths
- Unit test complex logic

## Chapter 2: Integration Testing

Sarah wants tests that match how users actually interact with the UI.

Test components as users interact with them.

```jsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { BrowserRouter } from 'react-router-dom';

// Mock server
const server = setupServer(
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' },
    ]);
  }),
  http.post('/api/users', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: 3, ...body });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Test wrapper with providers
function renderWithProviders(ui) {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });

  return render(
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        {ui}
      </BrowserRouter>
    </QueryClientProvider>
  );
}

// Integration test
describe('UserManagement', () => {
  it('displays users and allows adding new ones', async () => {
    const user = userEvent.setup();
    renderWithProviders(<UserManagement />);

    // Wait for users to load
    expect(await screen.findByText('Alice')).toBeInTheDocument();
    expect(screen.getByText('Bob')).toBeInTheDocument();

    // Add new user
    await user.click(screen.getByRole('button', { name: 'Add User' }));
    await user.type(screen.getByLabelText('Name'), 'Charlie');
    await user.click(screen.getByRole('button', { name: 'Save' }));

    // Verify new user appears
    await waitFor(() => {
      expect(screen.getByText('Charlie')).toBeInTheDocument();
    });
  });

  it('shows error when API fails', async () => {
    server.use(
      http.get('/api/users', () => HttpResponse.json([], { status: 500 }))
    );

    renderWithProviders(<UserManagement />);

    expect(await screen.findByRole('alert')).toHaveTextContent(
      'Failed to load users'
    );
  });
});
```

## Chapter 3: Mocking Strategies

External systems can't be part of every test. You need safe stand-ins.

### MSW for API Mocking

```jsx
// mocks/handlers.js
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/user', ({ request }) => {
    const isAuthenticated = request.headers.get('Authorization');
    if (!isAuthenticated) {
      return new HttpResponse(null, { status: 401 });
    }
    return HttpResponse.json({ id: 1, name: 'Test User' });
  }),
];

// Use in tests and development
// mocks/browser.js - for dev
import { setupWorker } from 'msw';
import { handlers } from './handlers';
export const worker = setupWorker(...handlers);

// mocks/server.js - for tests
import { setupServer } from 'msw/node';
import { handlers } from './handlers';
export const server = setupServer(...handlers);
```

### Mocking Modules

```jsx
// Mock a module
jest.mock('./analytics', () => ({
  trackEvent: jest.fn(),
}));

import { trackEvent } from './analytics';

it('tracks button click', async () => {
  const user = userEvent.setup();
  render(<TrackedButton />);

  await user.click(screen.getByRole('button'));

  expect(trackEvent).toHaveBeenCalledWith('button_clicked');
});

// Mock with implementation
jest.mock('./useAuth', () => ({
  useAuth: () => ({
    user: { id: 1, name: 'Test' },
    isAuthenticated: true,
  }),
}));
```

### Mocking Dates and Timers

```jsx
// Mock current date
beforeEach(() => {
  jest.useFakeTimers();
  jest.setSystemTime(new Date('2024-01-15'));
});

afterEach(() => {
  jest.useRealTimers();
});

it('shows relative time', () => {
  render(<TimeAgo date={new Date('2024-01-14')} />);
  expect(screen.getByText('1 day ago')).toBeInTheDocument();
});

// Advance timers
it('auto-saves after delay', async () => {
  const onSave = jest.fn();
  render(<AutoSaveForm onSave={onSave} />);

  await userEvent.type(screen.getByRole('textbox'), 'content');

  // Fast-forward debounce timer
  jest.advanceTimersByTime(1000);

  expect(onSave).toHaveBeenCalled();
});
```

## Chapter 4: Test Coverage

Coverage should highlight risk, not just inflate metrics.

```js
// jest.config.js
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
  ],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
};
```

**What to cover:**
- Critical business logic
- Edge cases and error states
- User interaction flows

**What not to chase:**
- 100% coverage for its own sake
- Simple presentational components
- Third-party library internals

## Chapter 5: CI/CD Testing

Marcus wants tests to run on every pull request, not just locally.

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linting
        run: npm run lint

      - name: Run type check
        run: npm run typecheck

      - name: Run unit tests
        run: npm run test:ci

      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

## Chapter 6: Testing Hooks

Custom hooks handle tricky logic and deserve direct tests too.

```jsx
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

describe('useFetch', () => {
  it('fetches data', async () => {
    const { result } = renderHook(() => useFetch('/api/data'));

    expect(result.current.isLoading).toBe(true);

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });

    expect(result.current.data).toEqual({ id: 1 });
  });
});
```

---

## Common Mistakes

1. **Testing implementation details** - Test behavior, not internal state.
2. **Over-mocking** - Excessive mocks can hide integration bugs.
3. **Ignoring flaky tests** - Flakiness erodes trust in the suite.
4. **Chasing 100% coverage** - Focus on risk and critical paths.


Example:
```ts
// ❌ Stub fetch in every test
global.fetch = jest.fn();

// ✅ Use MSW handlers for integration tests
import { http, HttpResponse } from 'msw';
```

## Practice Exercises

1. Write integration tests for a checkout flow
2. Set up MSW for a feature with complex API interactions
3. Configure CI to run tests on PRs

### Solutions

<details>
<summary>Exercise 1: Checkout Integration Test</summary>

```jsx
it('submits a checkout flow', async () => {
  const user = userEvent.setup();
  render(<Checkout />);

  await user.type(screen.getByLabelText(/card number/i), '4242 4242 4242 4242');
  await user.type(screen.getByLabelText(/email/i), 'user@example.com');
  await user.click(screen.getByRole('button', { name: /place order/i }));

  expect(await screen.findByRole('status')).toHaveTextContent(/order confirmed/i);
});
```

**Key points:**
- Interacts with the UI the way users do.
- Waits for async confirmation.
- Avoids internal implementation assertions.

</details>

<details>
<summary>Exercise 2: MSW Setup</summary>

```jsx
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';

export const server = setupServer(
  http.get('/api/products', () =>
    HttpResponse.json([{ id: 1, name: 'Headphones' }])
  ),
  http.post('/api/cart', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: 99, ...body });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Example override in a test
// server.use(
//   http.get('/api/products', () => HttpResponse.json([], { status: 500 }))
// );
```

**Key points:**
- Handlers mirror real endpoints.
- Tests stay fast and deterministic.
- Errors can be simulated per test.

</details>

<details>
<summary>Exercise 3: CI Test Workflow</summary>

```yaml
name: CI
on: [pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --ci
```

**Key points:**
- Tests run on every pull request.
- CI uses clean installs for consistency.
- Failures block merges automatically.

</details>

---

## What You Learned

This module covered:

- **Testing strategy**: Matching test depth to risk
- **Integration testing**: Verifying real user flows
- **Mocking**: Isolating external dependencies
- **Coverage**: Using metrics to guide, not mislead
- **CI**: Automating quality checks

**Key takeaway**: Confidence comes from the right mix of tests, not the most tests.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add integration tests for checkout and onboarding
- Use MSW to stabilize API-heavy tests
- Add PR checks that block failing test runs
- Focus coverage on high-risk paths
- Reduce flaky tests that slow down releases

---

## Further Reading

- [Testing Library Guiding Principles](https://testing-library.com/docs/guiding-principles)
- [MSW Documentation](https://mswjs.io/)

---

**Navigation**: [← Previous Module](./07-internationalization.md) | [Next Module →](./09-accessibility-testing-tooling.md)
