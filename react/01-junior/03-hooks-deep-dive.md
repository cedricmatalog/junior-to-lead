# Hooks Deep Dive

## Week Three: The Notification System

The streaming dashboard is live. Users are browsing, watching, building their lists. But something's missing.

"Users want notifications," Sarah says in Monday's standup. "New releases, friend activity, continue watching reminders. And it needs to be real-time."

Real-time. That means WebSockets, subscriptions, things that live outside React's normal render cycle.

"This is where hooks really shine," Marcus adds. "You'll learn more about `useEffect` this week than the last two weeks combined."

He's not wrong.

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **useState** | A sticky note - remembers a value between renders, updates trigger re-renders |
| **useEffect** | A morning routine - runs after render, handles side tasks, has cleanup steps |
| **Custom hooks** | Your recipe collection - reusable logic you write once and use anywhere |

Keep these in mind. They'll click as we build.

---

## Chapter 1: State That Sticks Around

You start with a simple notification counter. Every time a notification arrives, the badge should update.

"Let me show you the basics first," Sarah says.

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

"useState gives you two things: the current value, and a function to change it. Every time you call the setter, React re-renders."

You nod. You've seen this before. But Sarah's not done.

"Here's where people get tripped up."

```jsx
// Problem: rapid clicks lose updates
function BadCounter() {
  const [count, setCount] = useState(0);

  function handleTripleClick() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // Result: count goes up by 1, not 3!
  }
}
```

"Each `setCount` sees the same `count` value. They all say 'change 0 to 1'."

"So how do you fix it?"

```jsx
// Solution: functional updates
function GoodCounter() {
  const [count, setCount] = useState(0);

  function handleTripleClick() {
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    // Result: count goes up by 3
  }
}
```

"When you pass a function, React gives you the *latest* value. Each update builds on the previous one."

**Key patterns you'll use constantly:**

```jsx
// Functional updates (when new state depends on previous)
setCount(prev => prev + 1);

// Object state (always spread existing state)
const [user, setUser] = useState({ name: '', email: '' });
setUser(prev => ({ ...prev, name: 'Alice' }));

// Lazy initialization (for expensive computations)
const [data, setData] = useState(() => computeExpensiveValue());
```

---

## Chapter 2: The Effect Hook

"Now for the interesting part," Marcus says. "Notifications need to connect to a WebSocket. That's a *side effect*—something outside React's rendering."

He pulls up `useEffect`:

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

"Three parts," Sarah explains. "The effect function, the cleanup function, and the dependency array."

"What's the `cancelled` flag for?"

"Race conditions. If the user switches profiles quickly, you might get responses out of order. The flag prevents updating state for a profile you've already navigated away from."

**The dependency array is crucial:**

```jsx
// Run once on mount
useEffect(() => {
  initializeAnalytics();
}, []);

// Run when dependencies change
useEffect(() => {
  fetchUser(userId);
}, [userId]);

// Run on every render (rarely what you want)
useEffect(() => {
  console.log('Every single render');
});
```

"Think of the dependency array as an answer to 'when should this run again?' If `userId` changes, fetch the new user. Otherwise, leave it alone."

---

## Chapter 3: The Ref Escape Hatch

"We need a reference to the WebSocket," Marcus says. "But we don't want it to trigger re-renders every time it changes."

"That's what `useRef` is for."

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
```

"A ref is like a box. You can put anything in `.current`, and React doesn't care. No re-renders."

You'll use it for:
- DOM element references
- WebSocket connections
- Timer IDs
- Any mutable value that shouldn't trigger updates

```jsx
// Storing values without re-rendering
const renderCount = useRef(0);
renderCount.current += 1; // No re-render happens
console.log(`Rendered ${renderCount.current} times`);
```

---

## Chapter 4: Context for Deep Props

The notification bell needs to know who's logged in. That information lives at the top of the app.

"I could pass it down through every component..." you start.

"Please don't," Sarah laughs. "That's prop drilling. Use Context."

```jsx
const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Header />        {/* Doesn't need theme */}
      <MainContent />   {/* Doesn't need theme */}
      <DeepComponent /> {/* Needs theme! */}
    </ThemeContext.Provider>
  );
}

function DeepComponent() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>Themed content</div>;
}
```

"Context lets any component reach up and grab values from above—no matter how deep it is."

---

## Chapter 5: The Rules That Can't Be Broken

Before you get too far, Sarah stops you.

"Two rules. Break them and React breaks."

**Rule 1: Only call hooks at the top level**

```jsx
// WRONG - conditional hook
function Profile({ isLoggedIn }) {
  if (isLoggedIn) {
    const [user, setUser] = useState(null); // React loses track!
  }
}

// RIGHT - conditional rendering
function Profile({ isLoggedIn }) {
  const [user, setUser] = useState(null);
  if (!isLoggedIn) return <LoginPrompt />;
  return <UserDashboard user={user} />;
}
```

"React tracks hooks by their order. If the first render has 3 hooks and the second has 2, everything breaks."

**Rule 2: Only call hooks from React functions**

```jsx
// WRONG - regular function
function fetchAndSetUser(setUser) {
  useEffect(() => { ... }); // Can't use hooks here!
}

// RIGHT - custom hook
function useUser(userId) {
  useEffect(() => { ... }); // This is fine
}
```

---

## Chapter 6: Your First Custom Hook

Time to build the notification system for real.

"Watch how I package this up," Marcus says.

```jsx
function useNotifications(userId) {
  const [notifications, setNotifications] = useState([]);
  const [unreadCount, setUnreadCount] = useState(0);
  const wsRef = useRef(null);

  useEffect(() => {
    // Connect to WebSocket for real-time updates
    wsRef.current = new WebSocket(`wss://api.example.com/notifications/${userId}`);

    wsRef.current.onmessage = (event) => {
      const notification = JSON.parse(event.data);
      setNotifications(prev => [notification, ...prev]);
      setUnreadCount(count => count + 1);
    };

    // Cleanup on unmount or userId change
    return () => {
      wsRef.current?.close();
    };
  }, [userId]);

  const markAsRead = (notificationId) => {
    setNotifications(prev =>
      prev.map(n => n.id === notificationId ? { ...n, read: true } : n)
    );
    setUnreadCount(count => Math.max(0, count - 1));
  };

  return { notifications, unreadCount, markAsRead };
}
```

"All the complexity—WebSocket management, state updates, cleanup—hidden behind one simple interface."

**Custom hook conventions:**
- Name starts with `use`
- Can call other hooks inside
- Returns what the component needs

---

## Chapter 7: Putting It All Together

By Friday, the notification bell is live:

```jsx
function NotificationBell() {
  const { user } = useContext(AuthContext);
  const { notifications, unreadCount, markAsRead } = useNotifications(user.id);
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef(null);

  // Close dropdown when clicking outside
  useEffect(() => {
    function handleClickOutside(event) {
      if (dropdownRef.current && !dropdownRef.current.contains(event.target)) {
        setIsOpen(false);
      }
    }

    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, []);

  return (
    <div ref={dropdownRef} className="notification-bell">
      <button onClick={() => setIsOpen(!isOpen)}>
        <BellIcon />
        {unreadCount > 0 && <span className="badge">{unreadCount}</span>}
      </button>

      {isOpen && (
        <div className="dropdown">
          {notifications.map(notification => (
            <NotificationItem
              key={notification.id}
              notification={notification}
              onRead={() => markAsRead(notification.id)}
            />
          ))}
        </div>
      )}
    </div>
  );
}
```

Look at how many hooks work together:
- `useContext` - Gets the logged-in user
- `useNotifications` - Custom hook for real-time data
- `useState` - Tracks dropdown open/closed
- `useRef` - References the dropdown for click-outside detection
- `useEffect` - Sets up the click-outside listener

"Clean, readable, testable," Sarah says. "Each hook handles one thing. The component just orchestrates."

---

## Chapter 8: The Traps to Avoid

Before you leave for the weekend, Marcus shares a war story.

"I once crashed production with an infinite loop. Took me hours to figure out."

```jsx
// INFINITE LOOP - options is new object each render
function BadComponent({ url }) {
  const options = { method: 'GET', headers: {} }; // New object every render!

  useEffect(() => {
    fetch(url, options);
  }, [url, options]); // options always "changes"
}
```

"Objects and arrays are compared by reference. A new object looks different even if the contents are the same."

**The fix:**

```jsx
// Solution 1: useMemo
const options = useMemo(() => ({ method: 'GET', headers: {} }), []);

// Solution 2: Move outside component
const OPTIONS = { method: 'GET', headers: {} };

function GoodComponent({ url }) {
  useEffect(() => {
    fetch(url, OPTIONS);
  }, [url]);
}
```

**Other common mistakes:**

1. **Missing dependencies** - Use ESLint's `react-hooks/exhaustive-deps` rule
2. **Stale closures** - Use functional updates or refs
3. **Forgetting cleanup** - WebSockets, timers, event listeners need cleanup

---

## Chapter 9: The Performance Toolkit

"Now that you understand hooks," Sarah says, "let's talk about when *not* to recompute things."

### React.memo — Memoize Components

```jsx
// Without memo: re-renders every time parent renders
function ExpensiveList({ items }) {
  console.log('Rendering list...'); // Logs on every parent render
  return (
    <ul>
      {items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}

// With memo: only re-renders when props change
const ExpensiveList = memo(function ExpensiveList({ items }) {
  console.log('Rendering list...'); // Only logs when items changes
  return (
    <ul>
      {items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
});
```

### useMemo — Memoize Expensive Calculations

```jsx
function ProductList({ products, filter }) {
  // BAD: Filters on every render, even if products/filter unchanged
  const filteredProducts = products.filter(p => p.category === filter);

  // GOOD: Only recalculates when dependencies change
  const filteredProducts = useMemo(
    () => products.filter(p => p.category === filter),
    [products, filter]
  );

  return <List items={filteredProducts} />;
}
```

### useCallback — Memoize Functions

```jsx
function SearchPage() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  // BAD: New function every render, causes child re-renders
  const handleSearch = (term) => {
    fetch(`/api/search?q=${term}`).then(r => r.json()).then(setResults);
  };

  // GOOD: Same function reference unless dependencies change
  const handleSearch = useCallback((term) => {
    fetch(`/api/search?q=${term}`).then(r => r.json()).then(setResults);
  }, []);

  return <SearchBox onSearch={handleSearch} />;
}
```

### When NOT to Optimize

"Here's the thing," Marcus warns. "Most components don't need this."

```jsx
// DON'T DO THIS - premature optimization
const greeting = useMemo(() => `Hello, ${name}!`, [name]);

// Just write it normally
const greeting = `Hello, ${name}!`;
```

**Only optimize when:**
- You've measured a performance problem
- The calculation is genuinely expensive (filtering thousands of items)
- Child components re-render unnecessarily due to new function/object references

**Rule of thumb:** Write code without memoization first. Add it when you see performance issues in React DevTools Profiler.

---

## Hooks Reference

| Hook | Purpose | When to Use |
|------|---------|-------------|
| `useState` | Reactive values | Component needs to remember something |
| `useEffect` | Side effects | Fetch data, subscriptions, DOM manipulation |
| `useRef` | Mutable values | DOM refs, values that shouldn't trigger renders |
| `useContext` | Access context | Need data from distant parent |
| `useMemo` | Memoize values | Expensive computations |
| `useCallback` | Memoize functions | Prevent child re-renders |

## Common Mistakes

1. **Missing dependencies in useEffect** - Use ESLint rules to catch these
2. **Stale closures** - Use functional updates or refs
3. **Infinite loops** - Object/array dependencies recreated each render

## Practice Exercises

1. Build a `useWindowSize` hook that returns current viewport dimensions
2. Create a `useFetch` hook with loading, error, and data states
3. Build a `usePrevious` hook that returns the previous value

## Further Reading

- [React.dev - useState](https://react.dev/reference/react/useState)
- [React.dev - useEffect](https://react.dev/reference/react/useEffect)
- [React.dev - Rules of Hooks](https://react.dev/warnings/invalid-hook-call-warning)

---

**Navigation**: [← Previous Module](./02-component-patterns.md) | [Next Module →](./04-state-management-basics.md)
