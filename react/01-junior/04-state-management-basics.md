# State Management Basics

Learn when and how to manage state effectively in React applications.

## Learning Objectives

By the end of this module, you will:
- Understand different types of state and where they belong
- Use lifting state up to share data between components
- Apply Context API for global state
- Know when simple state vs external libraries are appropriate

## Types of State

### Local State
State that belongs to a single component.

```jsx
function SearchInput() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

### Shared State
State needed by multiple components - lift it up.

```jsx
function Parent() {
  const [filter, setFilter] = useState('all');

  return (
    <>
      <FilterButtons filter={filter} onChange={setFilter} />
      <ItemList filter={filter} />
    </>
  );
}
```

### Global State
State needed throughout the app - use Context.

```jsx
const UserContext = createContext(null);

function App() {
  const [user, setUser] = useState(null);
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Router />
    </UserContext.Provider>
  );
}
```

### Server State
Data from external sources - consider specialized libraries.

```jsx
// TanStack Query handles caching, refetching, etc.
const { data, isLoading } = useQuery(['user', id], () => fetchUser(id));
```

## Lifting State Up

When sibling components need the same data, move state to their common parent.

```jsx
// Before: Each input manages its own state
function TemperatureInputs() {
  return (
    <>
      <CelsiusInput />    {/* has own state */}
      <FahrenheitInput /> {/* has own state */}
    </>
  );
}

// After: Parent manages shared state
function TemperatureInputs() {
  const [celsius, setCelsius] = useState(0);
  const fahrenheit = (celsius * 9/5) + 32;

  return (
    <>
      <CelsiusInput
        value={celsius}
        onChange={setCelsius}
      />
      <FahrenheitInput
        value={fahrenheit}
        onChange={f => setCelsius((f - 32) * 5/9)}
      />
    </>
  );
}
```

## Context API

### Creating Context

```jsx
// Create context with default value
const ThemeContext = createContext('light');

// Provider component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggle = () => setTheme(t => t === 'light' ? 'dark' : 'light');

  return (
    <ThemeContext.Provider value={{ theme, toggle }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

### Consuming Context

```jsx
function ThemeToggle() {
  const { theme, toggle } = useContext(ThemeContext);
  return (
    <button onClick={toggle}>
      Current: {theme}
    </button>
  );
}
```

### Context Best Practices

1. **Split contexts by domain** - Don't put everything in one context
2. **Memoize provider values** - Prevent unnecessary re-renders
3. **Create custom hooks** - Better DX and encapsulation

```jsx
// Custom hook for cleaner consumption
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}
```

## When to Use External Libraries

Use simple state (useState + Context) when:
- App is small to medium sized
- State updates are straightforward
- Team is new to React

Consider Redux/Zustand when:
- Complex state with many interconnected updates
- Need middleware (logging, persistence)
- Large team needing strict patterns
- DevTools and time-travel debugging needed

```jsx
// Zustand - simpler alternative to Redux
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set(state => ({ count: state.count + 1 })),
}));

function Counter() {
  const { count, increment } = useStore();
  return <button onClick={increment}>{count}</button>;
}
```

## Common Mistakes

1. **Using state when not needed** - Derived values don't need state
2. **Too much global state** - Keep state as local as possible
3. **Prop drilling vs premature Context** - A few levels of props is fine

## Practice Exercises

1. Build a shopping cart with Context (add, remove, update quantity)
2. Create a form with multiple inputs sharing validation state
3. Implement a notification system with Context and reducer

## Further Reading

- [React.dev - Sharing State Between Components](https://react.dev/learn/sharing-state-between-components)
- [React.dev - Scaling Up with Reducer and Context](https://react.dev/learn/scaling-up-with-reducer-and-context)
