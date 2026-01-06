# Testing Fundamentals

> **Last reviewed**: 2026-01-06


## Week Ten: The Bug That Got Away

Six weeks of building. The travel platform looks great. Then Thursday morning, you get the Slack message every developer dreads.

"Production is down. Users are seeing a blank screen on checkout."

Sarah and Marcus gather around your desk. The bug? A typo in a conditional render. `isLoaading` instead of `isLoading`. The component tried to access properties on undefined.

"How did this ship?" you ask.

"Because you didn't test it," Marcus says. "No test caught this before production."

Sarah pulls up a chair. "Today, you learn testing. Not because it's fun—because bugs like this are preventable."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Unit tests** | Checking a single ingredient - fast, focused, isolated |
| **Integration tests** | Taste-testing the whole dish - verify pieces work together |
| **Mocking** | Fake ingredients for practice - isolates what you're testing |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Junior Module 07 (Styling Approaches) - Understanding components, state, and user interactions to test effectively.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Write tests that verify user-facing behavior rather than implementation details
- [ ] Query elements using accessible roles and labels like real users do
- [ ] Test user interactions with userEvent for realistic simulation
- [ ] Handle async operations in tests using findBy queries and waitFor
- [ ] Mock API calls and dependencies to isolate component logic
- [ ] Integrate automated accessibility testing with jest-axe

---

## Time Estimate

- **Reading**: 60-75 minutes
- **Exercises**: 4-5 hours
- **Mastery**: Write tests alongside every feature over 4-6 weeks to build the habit

---

## Chapter 1: The Testing Philosophy

"First, forget what you think testing means," Sarah starts.

Here's the test pyramid to guide your strategy:

```
┌──────────────────────────────────────────────────────────┐
│                 Testing Pyramid                          │
├──────────────────────────────────────────────────────────┤
│                                                          │
│                       /\                                 │
│                      /  \     E2E Tests                  │
│                     /    \    • Full user flows          │
│                    / Slow \   • Cross-browser            │
│                   /        \  • Few but critical         │
│                  /──────────\ • Cypress, Playwright      │
│                 /            \                           │
│                / Integration  \                          │
│               /     Tests      \                         │
│              /  • Components    \                        │
│             /    together        \                       │
│            /  • API + UI          \                      │
│           /   • React Testing      \                     │
│          /     Library              \                    │
│         /───────────────────────────\                    │
│        /                             \                   │
│       /         Unit Tests            \                  │
│      /  • Single functions             \                 │
│     /   • Pure logic                    \                │
│    /    • Fast, many tests               \               │
│   /     • Jest, Vitest                    \              │
│  /─────────────────────────────────────────\             │
│                                                          │
│  Write MORE tests at the bottom (fast, cheap)            │
│  Write FEWER tests at the top (slow, expensive)          │
│                                                          │
│  For React apps, focus on:                               │
│  • Integration tests (70%) - Component + user behavior   │
│  • Unit tests (20%) - Complex logic, utilities           │
│  • E2E tests (10%) - Critical user journeys              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

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

// Note: Matchers like toBeInTheDocument, toBeDisabled come from @testing-library/jest-dom
// Add this to your test setup file: import '@testing-library/jest-dom';

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

You start by querying the UI the way a real person would.

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

Here's the query priority flowchart - which query method to use:

```
┌──────────────────────────────────────────────────────────┐
│         Testing Library Query Priority                   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Start here: What are you testing?                       │
│       │                                                  │
│       ↓                                                  │
│  Can users interact with it?                             │
│       │                                                  │
│   ┌───┴───┐                                              │
│   │       │                                              │
│  YES     NO (text, images)                               │
│   │       │                                              │
│   ↓       └──→ screen.getByText('Welcome')               │
│  Is it a form element?                                   │
│   │                                                      │
│  YES ──→ screen.getByLabelText('Email')                  │
│   │      screen.getByPlaceholderText('Enter email')      │
│   │                                                      │
│  NO                                                      │
│   ↓                                                      │
│  Use role (BEST):                                        │
│  screen.getByRole('button', { name: 'Submit' })          │
│  screen.getByRole('heading', { name: 'Title' })          │
│  screen.getByRole('textbox', { name: 'Search' })         │
│                                                          │
│  Last resort:                                            │
│  screen.getByTestId('custom-element')                    │
│                                                          │
│  ─────────────────────────────────────────────────────   │
│                                                          │
│  Query Variants:                                         │
│                                                          │
│  getBy*    → Element must exist (throws if not)          │
│              Use for: Elements that should be there      │
│                                                          │
│  queryBy*  → Element may not exist (returns null)        │
│              Use for: Checking absence                   │
│              expect(queryByRole('alert')).not.toBe...    │
│                                                          │
│  findBy*   → Wait for element (async, returns promise)   │
│              Use for: Elements appearing after async     │
│              await findByText('Loaded!')                 │
│                                                          │
│  getAllBy*, queryAllBy*, findAllBy*                      │
│              Return arrays for multiple elements         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Chapter 4: Testing User Interactions

"Clicking is the easy part," Marcus says. "Typing, selecting, form submission—that's where it gets interesting."

You simulate real user actions, not just events.

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

"Notice you use `userEvent` instead of `fireEvent`. It simulates real user behavior—focus, typing character by character, blur."

---

## Chapter 5: The Async Dance

"Remember that checkout bug?" Sarah asks. "Loading states are where async testing saves you."

You wait for the UI to settle before you assert.

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
- Styling (unless it conveys meaning or is required for accessibility)
- Simple pass-through props

"If changing the internals breaks your test but not the behavior, your test is wrong."

---

## Chapter 8: The Banking Dashboard Tests

Time for a real challenge. The company is adding a banking dashboard—transactions, transfers, balances. Security and correctness are non-negotiable.

Sarah says, "Treat finance flows like production-critical code."

Because color conveys meaning in finance UIs, it's reasonable to assert the positive/negative styling here.

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

### Testing the Transfer Form

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

### Organizing Tests

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


Example:
```jsx
// ❌ Testing implementation details
expect(component.state.open).toBe(true);

// ✅ Testing user-visible behavior
expect(screen.getByRole('dialog')).toBeVisible();
```

## Practice Exercises

1. Write tests for a Todo component (add, complete, delete)
2. Test a form with validation (show errors, prevent submit)
3. Test a component that fetches data on mount

### Solutions

<details>
<summary>Exercise 1: Testing a Todo Component</summary>

```jsx
// TodoList.jsx
import { useState } from 'react';

export function TodoList() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');

  const addTodo = () => {
    if (!input.trim()) return;
    setTodos([...todos, { id: Date.now(), text: input, completed: false }]);
    setInput('');
  };

  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  return (
    <div>
      <h1>Todo List</h1>

      <div>
        <input
          type="text"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyDown={(e) => e.key === 'Enter' && addTodo()}
          placeholder="Add a todo..."
          aria-label="New todo"
        />
        <button onClick={addTodo}>Add</button>
      </div>

      {todos.length === 0 && <p>No todos yet. Add one above!</p>}

      <ul>
        {todos.map(todo => (
          <li
            key={todo.id}
            style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
          >
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
              aria-label={`Mark "${todo.text}" as ${todo.completed ? 'incomplete' : 'complete'}`}
            />
            <span>{todo.text}</span>
            <button onClick={() => deleteTodo(todo.id)}>
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

```jsx
// TodoList.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { TodoList } from './TodoList';

describe('TodoList', () => {
  it('renders with empty state', () => {
    render(<TodoList />);

    expect(screen.getByRole('heading', { name: 'Todo List' })).toBeInTheDocument();
    expect(screen.getByText('No todos yet. Add one above!')).toBeInTheDocument();
  });

  it('adds a new todo', async () => {
    const user = userEvent.setup();
    render(<TodoList />);

    const input = screen.getByLabelText('New todo');
    await user.type(input, 'Buy groceries');
    await user.click(screen.getByRole('button', { name: 'Add' }));

    expect(screen.getByText('Buy groceries')).toBeInTheDocument();
    expect(input).toHaveValue('');
  });

  it('adds todo when pressing Enter', async () => {
    const user = userEvent.setup();
    render(<TodoList />);

    const input = screen.getByLabelText('New todo');
    await user.type(input, 'Walk the dog{Enter}');

    expect(screen.getByText('Walk the dog')).toBeInTheDocument();
  });

  it('does not add empty todos', async () => {
    const user = userEvent.setup();
    render(<TodoList />);

    await user.click(screen.getByRole('button', { name: 'Add' }));

    expect(screen.getByText('No todos yet. Add one above!')).toBeInTheDocument();
  });

  it('toggles todo completion', async () => {
    const user = userEvent.setup();
    render(<TodoList />);

    // Add a todo
    await user.type(screen.getByLabelText('New todo'), 'Read book');
    await user.click(screen.getByRole('button', { name: 'Add' }));

    // Toggle it
    const checkbox = screen.getByRole('checkbox', {
      name: /Mark "Read book" as complete/
    });
    await user.click(checkbox);

    expect(checkbox).toBeChecked();

    // Toggle back
    await user.click(checkbox);
    expect(checkbox).not.toBeChecked();
  });

  it('deletes a todo', async () => {
    const user = userEvent.setup();
    render(<TodoList />);

    // Add a todo
    await user.type(screen.getByLabelText('New todo'), 'Learn React');
    await user.click(screen.getByRole('button', { name: 'Add' }));

    // Delete it
    await user.click(screen.getByRole('button', { name: 'Delete' }));

    expect(screen.queryByText('Learn React')).not.toBeInTheDocument();
    expect(screen.getByText('No todos yet. Add one above!')).toBeInTheDocument();
  });

  it('maintains multiple todos independently', async () => {
    const user = userEvent.setup();
    render(<TodoList />);

    // Add two todos
    await user.type(screen.getByLabelText('New todo'), 'First task{Enter}');
    await user.type(screen.getByLabelText('New todo'), 'Second task{Enter}');

    const checkboxes = screen.getAllByRole('checkbox');
    expect(checkboxes).toHaveLength(2);

    // Toggle only the first one
    await user.click(checkboxes[0]);

    expect(checkboxes[0]).toBeChecked();
    expect(checkboxes[1]).not.toBeChecked();
  });
});
```

**Key points:**
- Test user interactions, not implementation details
- Use accessible queries (getByRole, getByLabelText) for better tests
- Test edge cases like empty input
- userEvent simulates real user behavior better than fireEvent
- Each test is independent and sets up its own state

</details>

<details>
<summary>Exercise 2: Testing Form Validation</summary>

```jsx
// LoginForm.jsx
import { useState } from 'react';

export function LoginForm({ onSubmit }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const validate = () => {
    const newErrors = {};

    if (!email) {
      newErrors.email = 'Email is required';
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      newErrors.email = 'Invalid email address';
    }

    if (!password) {
      newErrors.password = 'Password is required';
    } else if (password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    if (!validate()) return;

    setIsSubmitting(true);

    try {
      await onSubmit({ email, password });
    } catch (error) {
      setErrors({ submit: error.message });
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          aria-invalid={errors.email ? 'true' : 'false'}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <span id="email-error" role="alert">
            {errors.email}
          </span>
        )}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          aria-invalid={errors.password ? 'true' : 'false'}
          aria-describedby={errors.password ? 'password-error' : undefined}
        />
        {errors.password && (
          <span id="password-error" role="alert">
            {errors.password}
          </span>
        )}
      </div>

      {errors.submit && (
        <div role="alert">{errors.submit}</div>
      )}

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

```jsx
// LoginForm.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('shows validation error for empty email', async () => {
    const user = userEvent.setup();
    const onSubmit = jest.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    await user.click(screen.getByRole('button', { name: 'Login' }));

    expect(screen.getByRole('alert', { name: /email is required/i })).toBeInTheDocument();
    expect(onSubmit).not.toHaveBeenCalled();
  });

  it('shows validation error for invalid email', async () => {
    const user = userEvent.setup();
    const onSubmit = jest.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText('Email'), 'not-an-email');
    await user.type(screen.getByLabelText('Password'), 'password123');
    await user.click(screen.getByRole('button', { name: 'Login' }));

    expect(screen.getByRole('alert', { name: /invalid email/i })).toBeInTheDocument();
    expect(onSubmit).not.toHaveBeenCalled();
  });

  it('shows validation error for short password', async () => {
    const user = userEvent.setup();
    const onSubmit = jest.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText('Email'), 'test@example.com');
    await user.type(screen.getByLabelText('Password'), 'short');
    await user.click(screen.getByRole('button', { name: 'Login' }));

    expect(screen.getByRole('alert', { name: /at least 8 characters/i })).toBeInTheDocument();
    expect(onSubmit).not.toHaveBeenCalled();
  });

  it('submits form with valid data', async () => {
    const user = userEvent.setup();
    const onSubmit = jest.fn().mockResolvedValue();
    render(<LoginForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText('Email'), 'test@example.com');
    await user.type(screen.getByLabelText('Password'), 'password123');
    await user.click(screen.getByRole('button', { name: 'Login' }));

    expect(onSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123',
    });
  });

  it('shows loading state while submitting', async () => {
    const user = userEvent.setup();
    const onSubmit = jest.fn(() => new Promise(resolve => setTimeout(resolve, 100)));
    render(<LoginForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText('Email'), 'test@example.com');
    await user.type(screen.getByLabelText('Password'), 'password123');
    await user.click(screen.getByRole('button', { name: 'Login' }));

    expect(screen.getByRole('button', { name: 'Logging in...' })).toBeDisabled();
  });

  it('handles submission error', async () => {
    const user = userEvent.setup();
    const onSubmit = jest.fn().mockRejectedValue(new Error('Invalid credentials'));
    render(<LoginForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText('Email'), 'test@example.com');
    await user.type(screen.getByLabelText('Password'), 'wrongpassword');
    await user.click(screen.getByRole('button', { name: 'Login' }));

    expect(await screen.findByText('Invalid credentials')).toBeInTheDocument();
  });

  it('sets aria-invalid on fields with errors', async () => {
    const user = userEvent.setup();
    const onSubmit = jest.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    await user.click(screen.getByRole('button', { name: 'Login' }));

    const emailInput = screen.getByLabelText('Email');
    expect(emailInput).toHaveAttribute('aria-invalid', 'true');
  });
});
```

**Key points:**
- Test validation logic thoroughly (empty, invalid, too short)
- Verify form doesn't submit when validation fails
- Test loading states and button disable behavior
- Check error messages are announced via role="alert"
- Test aria attributes for accessibility
- Mock async submission with both success and error cases

</details>

<details>
<summary>Exercise 3: Testing Data Fetching on Mount</summary>

```jsx
// UserProfile.jsx
import { useState, useEffect } from 'react';

export function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      try {
        setLoading(true);
        setError(null);

        const response = await fetch(`/api/users/${userId}`);

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        const data = await response.json();

        if (!cancelled) {
          setUser(data);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }

    fetchUser();

    return () => {
      cancelled = true;
    };
  }, [userId]);

  if (loading) return <p>Loading user...</p>;
  if (error) return <div role="alert">Error: {error}</div>;
  if (!user) return null;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
      <p>Bio: {user.bio}</p>
    </div>
  );
}
```

```jsx
// UserProfile.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import { UserProfile } from './UserProfile';

// Mock fetch globally
global.fetch = jest.fn();

describe('UserProfile', () => {
  beforeEach(() => {
    fetch.mockClear();
  });

  it('shows loading state initially', () => {
    fetch.mockImplementation(() => new Promise(() => {})); // Never resolves
    render(<UserProfile userId="1" />);

    expect(screen.getByText('Loading user...')).toBeInTheDocument();
  });

  it('displays user data after successful fetch', async () => {
    const mockUser = {
      id: '1',
      name: 'John Doe',
      email: 'john@example.com',
      bio: 'Software developer',
    };

    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockUser,
    });

    render(<UserProfile userId="1" />);

    expect(await screen.findByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('Email: john@example.com')).toBeInTheDocument();
    expect(screen.getByText('Bio: Software developer')).toBeInTheDocument();

    expect(fetch).toHaveBeenCalledWith('/api/users/1');
  });

  it('displays error message when fetch fails', async () => {
    fetch.mockRejectedValueOnce(new Error('Network error'));

    render(<UserProfile userId="1" />);

    expect(await screen.findByRole('alert')).toHaveTextContent('Error: Network error');
  });

  it('displays error for non-OK HTTP response', async () => {
    fetch.mockResolvedValueOnce({
      ok: false,
      status: 404,
    });

    render(<UserProfile userId="1" />);

    expect(await screen.findByRole('alert')).toHaveTextContent('HTTP error! status: 404');
  });

  it('refetches when userId changes', async () => {
    const user1 = { id: '1', name: 'User One', email: 'one@example.com', bio: 'Bio 1' };
    const user2 = { id: '2', name: 'User Two', email: 'two@example.com', bio: 'Bio 2' };

    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => user1,
    });

    const { rerender } = render(<UserProfile userId="1" />);

    expect(await screen.findByText('User One')).toBeInTheDocument();

    // Change userId
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => user2,
    });

    rerender(<UserProfile userId="2" />);

    expect(await screen.findByText('User Two')).toBeInTheDocument();
    expect(fetch).toHaveBeenCalledTimes(2);
    expect(fetch).toHaveBeenLastCalledWith('/api/users/2');
  });

  it('cancels previous request when userId changes quickly', async () => {
    const user1 = { id: '1', name: 'User One', email: 'one@example.com', bio: 'Bio 1' };
    const user2 = { id: '2', name: 'User Two', email: 'two@example.com', bio: 'Bio 2' };

    let resolveFirstFetch;
    fetch.mockImplementationOnce(() => new Promise(resolve => {
      resolveFirstFetch = () => resolve({
        ok: true,
        json: async () => user1,
      });
    }));

    const { rerender } = render(<UserProfile userId="1" />);

    // Immediately change to user 2 before first fetch completes
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => user2,
    });

    rerender(<UserProfile userId="2" />);

    // Now resolve the first fetch
    resolveFirstFetch();

    // Should only show user 2, not user 1
    await waitFor(() => {
      expect(screen.getByText('User Two')).toBeInTheDocument();
    });

    expect(screen.queryByText('User One')).not.toBeInTheDocument();
  });
});
```

**Key points:**
- Mock fetch globally before tests
- Test loading, success, and error states
- Use findBy queries for async elements
- Test race condition cancellation with controlled promises
- rerender allows testing prop changes
- waitFor handles async state updates
- Clear mocks between tests to avoid interference

</details>

---

## What You Learned

This module covered:

- **Testing Philosophy**: Test what users see and do, not internal implementation details
- **Accessible Queries**: Use getByRole, getByLabelText to query elements like assistive tech does
- **User Interactions**: userEvent simulates real user behavior better than fireEvent
- **Async Testing**: findBy and waitFor handle loading states and async operations
- **Mocking**: Isolate components by mocking API calls, hooks, and dependencies
- **Automated Accessibility**: jest-axe catches common accessibility violations

**Key takeaway**: Tests should break when behavior changes, not when implementation changes - test like a user, not like a developer.

---

## Real-World Application

This week at work, you might use these concepts to:

- Write integration tests for critical user flows like checkout and registration
- Catch accessibility issues automatically with jest-axe in your CI pipeline
- Test error states and loading spinners using async queries
- Mock API responses to test how components handle different data scenarios
- Build confidence that refactoring won't break user-facing functionality

---

## Further Reading

- [Testing Library Docs](https://testing-library.com/docs/)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Common Mistakes with RTL](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)

---

**Navigation**: [← Previous Module](./09-tooling-fundamentals.md) | [Next Module →](./11-debugging-techniques.md)
