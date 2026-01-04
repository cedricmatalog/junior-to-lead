# Debugging Techniques

Find and fix bugs efficiently using the right tools and methods.

## Learning Objectives

By the end of this module, you will:
- Use React DevTools to inspect components and state
- Debug effectively with browser developer tools
- Implement error boundaries to catch runtime errors
- Apply systematic debugging approaches

## React DevTools

Install the browser extension for Chrome or Firefox.

### Components Tab

- **Inspect component tree** - See props, state, hooks for any component
- **Search components** - Find by name or filter by type
- **Edit state/props** - Modify values live to test scenarios
- **Highlight updates** - See which components re-render

### Profiler Tab

- **Record rendering** - Identify slow components
- **Flame chart** - See render timing
- **Ranked view** - Find most expensive renders

```jsx
// Force a component to show in DevTools with a display name
const MyContext = createContext();
MyContext.displayName = 'MyContext';

// Named function for better DevTools display
const MemoizedComponent = memo(function MemoizedComponent(props) {
  return <div>{props.value}</div>;
});
```

## Console Methods

Beyond `console.log`:

```jsx
// Group related logs
console.group('User Data');
console.log('Name:', user.name);
console.log('Email:', user.email);
console.groupEnd();

// Tabular data
console.table(users);

// Timing
console.time('fetch');
await fetch('/api/data');
console.timeEnd('fetch'); // fetch: 234ms

// Conditional logging
console.assert(user.age > 0, 'Age should be positive', user);

// Stack trace
console.trace('How did we get here?');

// Styled output
console.log('%cImportant!', 'color: red; font-size: 20px');
```

## Browser DevTools

### Breakpoints

```jsx
function handleSubmit(data) {
  debugger; // Execution pauses here when DevTools is open

  // Or set breakpoints in Sources panel
  validateData(data);
  submitToAPI(data);
}
```

**Breakpoint types:**
- Line breakpoints - Pause at specific line
- Conditional breakpoints - Pause only when condition is true
- DOM breakpoints - Pause when DOM node changes
- XHR breakpoints - Pause on specific network requests

### Network Tab

- Inspect API requests and responses
- Throttle network to simulate slow connections
- Block requests to test error handling

### Application Tab

- View localStorage, sessionStorage, cookies
- Inspect service workers
- Clear storage for fresh state

## Error Boundaries

Catch JavaScript errors in component trees.

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log to error reporting service
    console.error('Error caught:', error, errorInfo);
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h2>Something went wrong</h2>
          <details>
            <summary>Error details</summary>
            <pre>{this.state.error?.message}</pre>
          </details>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage - wrap sections that might fail
<ErrorBoundary>
  <UserProfile />
</ErrorBoundary>
```

**Error boundaries don't catch:**
- Event handlers (use try/catch)
- Async code (use try/catch or .catch())
- Server-side rendering
- Errors in the error boundary itself

## Systematic Debugging

### 1. Reproduce the Bug
- Find the exact steps to trigger it
- Identify if it's consistent or intermittent

### 2. Isolate the Problem
- Remove unrelated code
- Create minimal reproduction
- Check if it happens with fresh state

### 3. Form a Hypothesis
- What do you think is causing it?
- What would prove or disprove your theory?

### 4. Test and Fix
- Make one change at a time
- Verify the fix doesn't break other things

### Common Bug Patterns

```jsx
// Stale closure
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1); // Bug: stale count
  }, 1000);
  return () => clearInterval(timer);
}, []); // Fix: use setCount(c => c + 1)

// Missing dependency
useEffect(() => {
  fetchData(userId); // Bug: doesn't refetch when userId changes
}, []); // Fix: add [userId]

// Mutating state
const handleAdd = (item) => {
  items.push(item); // Bug: mutating state
  setItems(items);  // Won't trigger re-render
}; // Fix: setItems([...items, item])
```

## Practice Exercises

1. Use React DevTools to find why a component re-renders unnecessarily
2. Debug a stale closure issue in a timer component
3. Add error boundaries to handle API failures gracefully

## Further Reading

- [React DevTools Tutorial](https://react.dev/learn/react-developer-tools)
- [Chrome DevTools Documentation](https://developer.chrome.com/docs/devtools/)
