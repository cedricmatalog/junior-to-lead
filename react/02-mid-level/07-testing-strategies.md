# Testing Strategies

Develop comprehensive testing approaches for real-world applications.

## Learning Objectives

By the end of this module, you will:
- Design effective testing strategies
- Write integration tests that provide confidence
- Mock external dependencies appropriately
- Set up CI/CD testing pipelines

## Testing Pyramid

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

## Integration Testing

Test components as users interact with them.

```jsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { rest } from 'msw';
import { setupServer } from 'msw/node';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { BrowserRouter } from 'react-router-dom';

// Mock server
const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json([
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' },
    ]));
  }),
  rest.post('/api/users', (req, res, ctx) => {
    return res(ctx.json({ id: 3, ...req.body }));
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
      rest.get('/api/users', (req, res, ctx) => {
        return res(ctx.status(500));
      })
    );

    renderWithProviders(<UserManagement />);

    expect(await screen.findByRole('alert')).toHaveTextContent(
      'Failed to load users'
    );
  });
});
```

## Mocking Strategies

### MSW for API Mocking

```jsx
// mocks/handlers.js
import { rest } from 'msw';

export const handlers = [
  rest.get('/api/user', (req, res, ctx) => {
    const isAuthenticated = req.headers.get('Authorization');
    if (!isAuthenticated) {
      return res(ctx.status(401));
    }
    return res(ctx.json({ id: 1, name: 'Test User' }));
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

## Test Coverage

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

## CI/CD Testing

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

## Testing Hooks

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

## Practice Exercises

1. Write integration tests for a checkout flow
2. Set up MSW for a feature with complex API interactions
3. Configure CI to run tests on PRs

## Further Reading

- [Testing Library Guiding Principles](https://testing-library.com/docs/guiding-principles)
- [MSW Documentation](https://mswjs.io/)
