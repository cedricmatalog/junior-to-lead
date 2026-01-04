# Testing Fundamentals

Write tests that catch bugs and give confidence in your code.

## Learning Objectives

By the end of this module, you will:
- Set up Jest and React Testing Library
- Write unit tests for components
- Test user interactions and async behavior
- Understand what to test and what to skip

## Testing Philosophy

**Test behavior, not implementation.**

```jsx
// Bad: Testing implementation details
expect(wrapper.state('count')).toBe(1);
expect(component.instance().handleClick).toHaveBeenCalled();

// Good: Testing what users see and do
expect(screen.getByText('Count: 1')).toBeInTheDocument();
fireEvent.click(screen.getByRole('button', { name: 'Increment' }));
expect(screen.getByText('Count: 2')).toBeInTheDocument();
```

## Setting Up Tests

```jsx
// Button.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import Button from './Button';

describe('Button', () => {
  it('renders children correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: 'Click me' })).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click</Button>);

    fireEvent.click(screen.getByRole('button'));

    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Disabled</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

## Querying Elements

Use queries that reflect how users find elements.

```jsx
// Priority (best to worst)
screen.getByRole('button', { name: 'Submit' }); // Accessible name
screen.getByLabelText('Email');                  // Form labels
screen.getByPlaceholderText('Enter email');      // Placeholder
screen.getByText('Welcome');                     // Visible text
screen.getByTestId('custom-element');            // Last resort
```

**Query variants:**
- `getBy` - Throws if not found (use for elements that should exist)
- `queryBy` - Returns null if not found (use for elements that might not exist)
- `findBy` - Async, waits for element (use for elements that appear after async operations)

```jsx
// Element should exist
expect(screen.getByRole('button')).toBeInTheDocument();

// Element should NOT exist
expect(screen.queryByRole('alert')).not.toBeInTheDocument();

// Wait for element to appear
const message = await screen.findByText('Success!');
```

## Testing User Interactions

```jsx
import userEvent from '@testing-library/user-event';

it('filters list when user types in search', async () => {
  const user = userEvent.setup();
  render(<SearchableList items={['Apple', 'Banana', 'Cherry']} />);

  await user.type(screen.getByRole('searchbox'), 'app');

  expect(screen.getByText('Apple')).toBeInTheDocument();
  expect(screen.queryByText('Banana')).not.toBeInTheDocument();
});

it('submits form with entered values', async () => {
  const user = userEvent.setup();
  const onSubmit = jest.fn();
  render(<LoginForm onSubmit={onSubmit} />);

  await user.type(screen.getByLabelText('Email'), 'test@example.com');
  await user.type(screen.getByLabelText('Password'), 'password123');
  await user.click(screen.getByRole('button', { name: 'Login' }));

  expect(onSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123',
  });
});
```

## Testing Async Behavior

```jsx
it('shows loading state then data', async () => {
  render(<UserProfile userId="1" />);

  // Initially shows loading
  expect(screen.getByText('Loading...')).toBeInTheDocument();

  // Wait for data to load
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
  expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
});

it('shows error message on fetch failure', async () => {
  // Mock failed fetch
  global.fetch = jest.fn(() => Promise.reject(new Error('Network error')));

  render(<UserProfile userId="1" />);

  expect(await screen.findByRole('alert')).toHaveTextContent('Failed to load');
});
```

## Mocking

```jsx
// Mock a module
jest.mock('./api', () => ({
  fetchUser: jest.fn(),
}));

import { fetchUser } from './api';

beforeEach(() => {
  fetchUser.mockResolvedValue({ id: 1, name: 'John' });
});

// Mock a hook
jest.mock('./useAuth', () => ({
  useAuth: () => ({ user: { name: 'Test User' }, isLoggedIn: true }),
}));
```

## What to Test

**Do test:**
- User interactions (clicks, typing, form submission)
- Conditional rendering (show/hide based on state)
- Edge cases (empty lists, error states, loading)
- Accessibility (roles, labels, focus management)

**Don't test:**
- Implementation details (internal state, private methods)
- Third-party libraries (they have their own tests)
- Styling (unless critical to functionality)
- Simple pass-through props

## Test Structure

```jsx
describe('ComponentName', () => {
  // Group related tests
  describe('when user is logged in', () => {
    it('shows dashboard link', () => { });
    it('shows logout button', () => { });
  });

  describe('when user is logged out', () => {
    it('shows login button', () => { });
  });
});
```

## Practice Exercises

1. Write tests for a Todo component (add, complete, delete)
2. Test a form with validation (show errors, prevent submit)
3. Test a component that fetches data on mount

## Further Reading

- [Testing Library Docs](https://testing-library.com/docs/)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Common Mistakes with RTL](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
