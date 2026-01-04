# Testing Fundamentals

## Week Seven: The Bug That Got Away

Six weeks of building. The travel platform looks great. Then Thursday morning, you get the Slack message every developer dreads.

"Production is down. Users are seeing a blank screen on checkout."

Sarah and Marcus gather around your desk. The bug? A typo in a conditional render. `isLoaading` instead of `isLoading`. The component tried to access properties on undefined.

"How did this ship?" you ask.

"Because we didn't test it," Marcus says. "No test caught this before production."

Sarah pulls up a chair. "Today, you learn testing. Not because it's fun—because bugs like this are preventable."

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Unit tests** | Checking a single ingredient - fast, focused, isolated |
| **Integration tests** | Taste-testing the whole dish - verify pieces work together |
| **Mocking** | Fake ingredients for practice - isolates what you're testing |

Keep these in mind. They'll click as we build.

---

## Chapter 1: The Testing Philosophy

"First, forget what you think testing means," Sarah starts.

She shows you two approaches:

```jsx
// Bad: Testing implementation details
expect(wrapper.state('count')).toBe(1);
expect(component.instance().handleClick).toHaveBeenCalled();

// Good: Testing what users see and do
expect(screen.getByText('Count: 1')).toBeInTheDocument();
fireEvent.click(screen.getByRole('button', { name: 'Increment' }));
expect(screen.getByText('Count: 2')).toBeInTheDocument();
```

"The first approach breaks when you refactor. The second keeps working as long as the *behavior* stays the same."

The mantra: **Test behavior, not implementation.**

---

## Chapter 2: Your First Test

Marcus sets up a simple button test.

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

"Three tests. Three behaviors. If any of these break, you know exactly what went wrong."

---

## Chapter 3: Finding Elements Like Users Do

"Users don't inspect the DOM," Sarah says. "They look for buttons, labels, text. Your tests should too."

```jsx
// Priority (best to worst)
screen.getByRole('button', { name: 'Submit' }); // Accessible name
screen.getByLabelText('Email');                  // Form labels
screen.getByPlaceholderText('Enter email');      // Placeholder
screen.getByText('Welcome');                     // Visible text
screen.getByTestId('custom-element');            // Last resort
```

"Start with `getByRole`. It tests accessibility for free."

**Query variants matter too:**

```jsx
// Element should exist
expect(screen.getByRole('button')).toBeInTheDocument();

// Element should NOT exist
expect(screen.queryByRole('alert')).not.toBeInTheDocument();

// Wait for element to appear (async)
const message = await screen.findByText('Success!');
```

"Use `getBy` when it must exist. `queryBy` when checking absence. `findBy` when waiting for async."

---

## Chapter 4: Testing User Interactions

"Clicking is the easy part," Marcus says. "Typing, selecting, form submission—that's where it gets interesting."

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

"Notice we use `userEvent` instead of `fireEvent`. It simulates real user behavior—focus, typing character by character, blur."

---

## Chapter 5: The Async Dance

"Remember that checkout bug?" Sarah asks. "Loading states are where async testing saves you."

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

"The `findBy` queries wait automatically. No manual timers, no flaky tests."

---

## Chapter 6: Mocking Dependencies

"You can't hit real APIs in tests," Marcus explains. "That's where mocking comes in."

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

"Mocks let you control the world your component lives in. Network always fast. User always logged in. Whatever you need."

---

## Chapter 7: What to Test (And What Not To)

Sarah draws a line in the sand.

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

"If changing the internals breaks your test but not the behavior, your test is wrong."

---

## Chapter 8: The Banking Dashboard Tests

Time for a real challenge. The company is adding a banking dashboard—transactions, transfers, balances. Security and correctness are non-negotiable.

```jsx
// TransactionList.test.jsx
import { render, screen, within } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { TransactionList } from './TransactionList';

const mockTransactions = [
  { id: '1', date: '2024-01-15', description: 'Coffee Shop', amount: -4.50, category: 'food' },
  { id: '2', date: '2024-01-15', description: 'Salary Deposit', amount: 3500.00, category: 'income' },
  { id: '3', date: '2024-01-14', description: 'Electric Bill', amount: -125.00, category: 'utilities' },
];

describe('TransactionList', () => {
  it('displays transactions grouped by date', () => {
    render(<TransactionList transactions={mockTransactions} />);

    expect(screen.getByText('January 15, 2024')).toBeInTheDocument();
    expect(screen.getByText('January 14, 2024')).toBeInTheDocument();
    expect(screen.getByText('Coffee Shop')).toBeInTheDocument();
    expect(screen.getByText('Salary Deposit')).toBeInTheDocument();
  });

  it('formats positive amounts in green with + sign', () => {
    render(<TransactionList transactions={mockTransactions} />);

    const salaryRow = screen.getByText('Salary Deposit').closest('tr');
    const amountCell = within(salaryRow).getByText('+$3,500.00');

    expect(amountCell).toHaveClass('text-green-600');
  });

  it('formats negative amounts in red with - sign', () => {
    render(<TransactionList transactions={mockTransactions} />);

    const coffeeRow = screen.getByText('Coffee Shop').closest('tr');
    const amountCell = within(coffeeRow).getByText('-$4.50');

    expect(amountCell).toHaveClass('text-red-600');
  });

  it('filters transactions by category', async () => {
    const user = userEvent.setup();
    render(<TransactionList transactions={mockTransactions} />);

    await user.selectOptions(screen.getByLabelText('Filter by category'), 'food');

    expect(screen.getByText('Coffee Shop')).toBeInTheDocument();
    expect(screen.queryByText('Salary Deposit')).not.toBeInTheDocument();
  });

  it('shows empty state when no transactions match filter', async () => {
    const user = userEvent.setup();
    render(<TransactionList transactions={mockTransactions} />);

    await user.selectOptions(screen.getByLabelText('Filter by category'), 'travel');

    expect(screen.getByText('No transactions found')).toBeInTheDocument();
  });
});
```

---

## Chapter 9: Testing the Transfer Form

The transfer form is critical. Money moves. Mistakes cost.

```jsx
describe('TransferForm', () => {
  const mockAccounts = [
    { id: 'checking', name: 'Checking', balance: 5000 },
    { id: 'savings', name: 'Savings', balance: 10000 },
  ];

  it('prevents transfer exceeding available balance', async () => {
    const user = userEvent.setup();
    const onTransfer = jest.fn();

    render(<TransferForm accounts={mockAccounts} onTransfer={onTransfer} />);

    await user.selectOptions(screen.getByLabelText('From'), 'checking');
    await user.selectOptions(screen.getByLabelText('To'), 'savings');
    await user.type(screen.getByLabelText('Amount'), '6000');
    await user.click(screen.getByRole('button', { name: 'Transfer' }));

    expect(screen.getByRole('alert')).toHaveTextContent('Insufficient funds');
    expect(onTransfer).not.toHaveBeenCalled();
  });

  it('prevents transfer to same account', async () => {
    const user = userEvent.setup();
    render(<TransferForm accounts={mockAccounts} onTransfer={jest.fn()} />);

    await user.selectOptions(screen.getByLabelText('From'), 'checking');

    const toSelect = screen.getByLabelText('To');
    expect(within(toSelect).queryByText('Checking')).not.toBeInTheDocument();
  });

  it('submits valid transfer and shows confirmation', async () => {
    const user = userEvent.setup();
    const onTransfer = jest.fn().mockResolvedValue({ success: true });

    render(<TransferForm accounts={mockAccounts} onTransfer={onTransfer} />);

    await user.selectOptions(screen.getByLabelText('From'), 'checking');
    await user.selectOptions(screen.getByLabelText('To'), 'savings');
    await user.type(screen.getByLabelText('Amount'), '500');
    await user.click(screen.getByRole('button', { name: 'Transfer' }));

    expect(await screen.findByText('Transfer successful!')).toBeInTheDocument();
    expect(onTransfer).toHaveBeenCalledWith({
      from: 'checking',
      to: 'savings',
      amount: 500,
    });
  });

  it('shows loading state during transfer', async () => {
    const user = userEvent.setup();
    const onTransfer = jest.fn().mockImplementation(
      () => new Promise(resolve => setTimeout(() => resolve({ success: true }), 100))
    );

    render(<TransferForm accounts={mockAccounts} onTransfer={onTransfer} />);

    await user.selectOptions(screen.getByLabelText('From'), 'checking');
    await user.selectOptions(screen.getByLabelText('To'), 'savings');
    await user.type(screen.getByLabelText('Amount'), '100');
    await user.click(screen.getByRole('button', { name: 'Transfer' }));

    expect(screen.getByRole('button', { name: 'Processing...' })).toBeDisabled();
    expect(await screen.findByText('Transfer successful!')).toBeInTheDocument();
  });
});
```

"These tests would have caught your checkout bug," Sarah says. "Loading states, error handling, validation—all covered."

---

## Chapter 10: Organizing Tests

```jsx
describe('ComponentName', () => {
  describe('when user is logged in', () => {
    it('shows dashboard link', () => { });
    it('shows logout button', () => { });
  });

  describe('when user is logged out', () => {
    it('shows login button', () => { });
  });
});
```

"Group by scenario. When tests fail, the description tells you exactly what broke."

---

## Testing Checklist

| Scenario | Test It? |
|----------|----------|
| Button click calls handler | Yes |
| Form validates input | Yes |
| Loading state appears | Yes |
| Error message on failure | Yes |
| Internal state value | No |
| CSS class names | Usually no |
| Third-party library behavior | No |

## Common Mistakes

1. **Testing implementation** - Don't check state, check what users see
2. **Too many mocks** - If you mock everything, you're testing nothing
3. **Not testing error cases** - Happy path isn't enough
4. **Brittle selectors** - Use roles and labels, not class names

## Practice Exercises

1. Write tests for a Todo component (add, complete, delete)
2. Test a form with validation (show errors, prevent submit)
3. Test a component that fetches data on mount

## Further Reading

- [Testing Library Docs](https://testing-library.com/docs/)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Common Mistakes with RTL](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
