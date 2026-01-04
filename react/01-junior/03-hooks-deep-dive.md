# Hooks Deep Dive

Understand how hooks work and when to use each one.

## Learning Objectives

By the end of this module, you will:
- Use useState, useEffect, useRef, and useContext correctly
- Understand the rules of hooks and why they exist
- Build custom hooks to share logic
- Debug common hook-related issues

## Core Hooks

### useState

Adds state to functional components.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

**Key patterns:**

```jsx
// Functional updates (when new state depends on previous)
setCount(prev => prev + 1);

// Object state (always spread existing state)
const [user, setUser] = useState({ name: '', email: '' });
setUser(prev => ({ ...prev, name: 'Alice' }));

// Lazy initialization (for expensive computations)
const [data, setData] = useState(() => computeExpensiveValue());
```

### useEffect

Synchronize with external systems (APIs, DOM, subscriptions).

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();
      if (!cancelled) setUser(data);
    }

    fetchUser();

    // Cleanup function
    return () => { cancelled = true; };
  }, [userId]); // Dependency array

  return user ? <div>{user.name}</div> : <p>Loading...</p>;
}
```

**Dependency array rules:**
- `[]` - Run once on mount
- `[dep1, dep2]` - Run when dependencies change
- No array - Run on every render (rarely needed)

### useRef

Hold mutable values that don't trigger re-renders.

```jsx
function TextInput() {
  const inputRef = useRef(null);

  function focusInput() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}

// Storing values without re-rendering
const renderCount = useRef(0);
renderCount.current += 1; // Doesn't cause re-render
```

### useContext

Access context values without prop drilling.

```jsx
const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <DeepComponent />
    </ThemeContext.Provider>
  );
}

function DeepComponent() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>Themed content</div>;
}
```

## Rules of Hooks

1. **Only call hooks at the top level** - Not inside loops, conditions, or nested functions
2. **Only call hooks from React functions** - Components or custom hooks

```jsx
// Bad - conditional hook
if (isLoggedIn) {
  const [user, setUser] = useState(null); // Error!
}

// Good - conditional rendering
const [user, setUser] = useState(null);
if (!isLoggedIn) return <LoginForm />;
```

## Custom Hooks

Extract and share logic between components.

```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const saved = localStorage.getItem(key);
    return saved ? JSON.parse(saved) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}

// Usage
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  // ...
}
```

**Custom hook conventions:**
- Name starts with `use`
- Can call other hooks
- Returns values/functions the component needs

## Common Mistakes

1. **Missing dependencies in useEffect** - Use ESLint rules to catch these
2. **Stale closures** - Use functional updates or refs
3. **Infinite loops** - Object/array dependencies recreated each render

```jsx
// Infinite loop! options is new object each render
useEffect(() => {
  fetch(url, options);
}, [options]);

// Fix with useMemo or move outside component
const memoizedOptions = useMemo(() => ({ method: 'GET' }), []);
```

## Practice Exercises

1. Build a `useWindowSize` hook that returns current viewport dimensions
2. Create a `useFetch` hook with loading, error, and data states
3. Build a `usePrevious` hook that returns the previous value

## Further Reading

- [React.dev - useState](https://react.dev/reference/react/useState)
- [React.dev - useEffect](https://react.dev/reference/react/useEffect)
- [React.dev - Rules of Hooks](https://react.dev/warnings/invalid-hook-call-warning)
