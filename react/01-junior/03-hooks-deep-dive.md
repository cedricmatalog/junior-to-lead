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

## Prerequisites

Module 02 (Component Patterns) - Familiarity with props, component composition, and how data flows in React.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Use useState with functional updates to handle state based on previous values
- [ ] Implement useEffect for side effects with proper cleanup and dependency arrays
- [ ] Manage mutable values and DOM references using useRef without triggering re-renders
- [ ] Share global state across components using useContext
- [ ] Build custom hooks to extract and reuse stateful logic
- [ ] Apply performance optimizations with useMemo and useCallback when needed

---

## Time Estimate

- **Reading**: 50-70 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice hooks patterns over 3-4 weeks as you build real features

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

Here's the useEffect lifecycle visualized:

```
┌──────────────────────────────────────────────────────────┐
│              useEffect Lifecycle                         │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Component renders                                       │
│       ↓                                                  │
│  React commits to DOM                                    │
│       ↓                                                  │
│  Browser paints screen                                   │
│       ↓                                                  │
│  useEffect runs  ────────┐                               │
│       ↓                  │                               │
│  Side effect executes    │                               │
│  (fetch, subscribe, etc) │                               │
│       │                  │                               │
│       │                  │                               │
│  Component re-renders ───┤                               │
│       ↓                  │                               │
│  Cleanup runs ←──────────┘  (if dependencies changed)   │
│  (prev effect cleanup)                                   │
│       ↓                                                  │
│  New effect runs                                         │
│       │                                                  │
│       │                                                  │
│  Component unmounts                                      │
│       ↓                                                  │
│  Final cleanup runs  ← (always runs on unmount)         │
│                                                          │
│  Example:                                                │
│  useEffect(() => {                                       │
│    socket.connect();     ← Effect                       │
│    return () => {                                        │
│      socket.disconnect(); ← Cleanup                     │
│    };                                                    │
│  }, [userId]);           ← Dependencies                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
```"

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

Here's a decision tree for dependency arrays:

```
┌──────────────────────────────────────────────────────────┐
│         Dependency Array Decision Tree                   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Does your effect use props or state?                   │
│           │                                              │
│     ┌─────┴─────┐                                        │
│     │           │                                        │
│    NO          YES                                       │
│     │           │                                        │
│     │           ├─ Does it use ALL props/state?         │
│     │           │      │                                 │
│     │           │  ┌───┴───┐                             │
│     │           │  │       │                             │
│     │           │ YES     NO                             │
│     │           │  │       │                             │
│     ↓           │  │       ↓                             │
│  useEffect(     │  │   useEffect(                        │
│    () => {},    │  │     () => {},                       │
│    []  ← Empty  │  │     [specific] ← List them          │
│  )              │  │   )                                 │
│                 │  │                                     │
│  Run once       │  │   Re-run when those change         │
│  on mount       │  │                                     │
│                 │  ↓                                     │
│                 │  useEffect(                            │
│                 │    () => {},                           │
│                 │    [all, props, state] ← All of them  │
│                 │  )                                     │
│                 │                                        │
│                 │  Re-run when any change               │
│                 │                                        │
│  Common mistake:                                         │
│  useEffect(() => {                                       │
│    fetchUser(userId); ← Uses userId                     │
│  }, []); ← Missing dependency! Bug!                     │
│                                                          │
│  ESLint will warn you - listen to it!                   │
│                                                          │
└──────────────────────────────────────────────────────────┘
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

Here's what happens with race conditions:

```
┌──────────────────────────────────────────────────────────┐
│           Race Condition Timeline                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Without cancellation flag (BUG):                       │
│                                                          │
│  Time   User Action        Fetch Request    State       │
│  ────   ───────────        ────────────────  ──────     │
│   0s    View user A    →   Fetch A starts              │
│   1s    View user B    →   Fetch B starts   (A pending) │
│   2s                   ←   B returns        Set B ✓     │
│   3s                   ←   A returns        Set A ✗     │
│                                                          │
│  Result: Shows user A (wrong!)                          │
│  User clicked B but sees A's data                       │
│                                                          │
│  ─────────────────────────────────────────────────────  │
│                                                          │
│  With cancellation flag (FIX):                          │
│                                                          │
│  Time   User Action        Fetch Request    State       │
│  ────   ───────────        ────────────────  ──────     │
│   0s    View user A    →   Fetch A starts              │
│         Set cancelled=false                             │
│   1s    View user B    →   Cleanup: cancelled=true     │
│                        →   Fetch B starts              │
│         Set cancelled=false                             │
│   2s                   ←   B returns        Set B ✓     │
│   3s                   ←   A returns        Ignored! ✓  │
│                            (cancelled=true)             │
│                                                          │
│  Result: Shows user B (correct!)                        │
│  Stale response ignored                                 │
│                                                          │
│  Code pattern:                                          │
│  useEffect(() => {                                      │
│    let cancelled = false;                               │
│    fetchData().then(data => {                           │
│      if (!cancelled) setState(data);                   │
│    });                                                  │
│    return () => { cancelled = true; };                 │
│  }, [id]);                                              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

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
import { memo } from 'react';

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

---

## Debug This Code

Before moving to the exercises, test your debugging skills. Each snippet has bugs - can you spot and fix them?

<details>
<summary>Challenge 1: Infinite Loop with useEffect</summary>

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId, user]);

  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

Check the dependency array. What happens when `user` changes?

</details>

<details>
<summary>Solution</summary>

**Bug**: `user` in the dependency array causes an infinite loop. When user state updates, the effect runs again, fetches data, updates user, triggers effect again, etc.

**Fixed code:**

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId]); // Remove 'user' from dependencies

  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

**Why it matters**: Including state that the effect updates in the dependency array creates infinite re-render loops. Only include external values the effect depends on.

</details>

</details>

<details>
<summary>Challenge 2: Stale Closure in Timer</summary>

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    if (!isRunning) return;

    const interval = setInterval(() => {
      setCount(count + 1); // Bug: stale count value
    }, 1000);

    return () => clearInterval(interval);
  }, [isRunning]);

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? 'Pause' : 'Start'}
      </button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

The interval captures the `count` value. What happens when count should update?

</details>

<details>
<summary>Solution</summary>

**Bug**: Stale closure - the interval callback captures the initial `count` value (0) and never sees updates.

**Fixed code (using functional update):**

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    if (!isRunning) return;

    const interval = setInterval(() => {
      setCount(prevCount => prevCount + 1); // Use functional update
    }, 1000);

    return () => clearInterval(interval);
  }, [isRunning]);

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? 'Pause' : 'Start'}
      </button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

**Alternative fix (using useRef):**

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  const countRef = useRef(count);

  useEffect(() => {
    countRef.current = count;
  }, [count]);

  useEffect(() => {
    if (!isRunning) return;

    const interval = setInterval(() => {
      setCount(countRef.current + 1);
    }, 1000);

    return () => clearInterval(interval);
  }, [isRunning]);

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? 'Pause' : 'Start'}
      </button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

**Why it matters**: Closures in effects capture values at creation time. Use functional updates or refs to access current values in async callbacks.

</details>

</details>

<details>
<summary>Challenge 3: Missing Cleanup in useEffect</summary>

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true);

    fetch(`/api/search?q=${query}`)
      .then(res => res.json())
      .then(data => {
        setResults(data);
        setLoading(false);
      });
  }, [query]);

  if (loading) return <div>Searching...</div>;

  return (
    <ul>
      {results.map(item => (
        <li key={item.id}>{item.title}</li>
      ))}
    </ul>
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

What happens if the user types quickly and `query` changes before the fetch completes?

</details>

<details>
<summary>Solution</summary>

**Bug**: No cleanup means stale requests can complete after newer ones, showing outdated results. Also causes "Can't perform a React state update on an unmounted component" warnings.

**Fixed code:**

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    let cancelled = false;

    setLoading(true);

    fetch(`/api/search?q=${query}`)
      .then(res => res.json())
      .then(data => {
        if (!cancelled) {
          setResults(data);
          setLoading(false);
        }
      });

    return () => {
      cancelled = true;
    };
  }, [query]);

  if (loading) return <div>Searching...</div>;

  return (
    <ul>
      {results.map(item => (
        <li key={item.id}>{item.title}</li>
      ))}
    </ul>
  );
}
```

**Even better with AbortController:**

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    const abortController = new AbortController();

    setLoading(true);

    fetch(`/api/search?q=${query}`, { signal: abortController.signal })
      .then(res => res.json())
      .then(data => {
        setResults(data);
        setLoading(false);
      })
      .catch(error => {
        if (error.name !== 'AbortError') {
          console.error('Search failed:', error);
        }
      });

    return () => {
      abortController.abort();
    };
  }, [query]);

  if (loading) return <div>Searching...</div>;

  return (
    <ul>
      {results.map(item => (
        <li key={item.id}>{item.title}</li>
      ))}
    </ul>
  );
}
```

**Why it matters**: Async operations that complete after component unmount or dependency change cause memory leaks and race conditions. Always cleanup effects.

</details>

</details>

---

## Practice Exercises

1. Build a `useWindowSize` hook that returns current viewport dimensions
2. Create a `useFetch` hook with loading, error, and data states
3. Build a `usePrevious` hook that returns the previous value

### Solutions

<details>
<summary>Exercise 1: useWindowSize Hook</summary>

```jsx
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    function handleResize() {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []); // Empty deps - only set up listener once

  return size;
}

// Usage
function ResponsiveComponent() {
  const { width, height } = useWindowSize();

  return (
    <div>
      <p>Window size: {width} x {height}</p>
      {width < 768 ? <MobileView /> : <DesktopView />}
    </div>
  );
}
```

**Key points:**
- Initialize state with current window dimensions to avoid flash of incorrect values
- Use cleanup function to remove event listener and prevent memory leaks
- Empty dependency array ensures listener is added once, not on every render
- Consider debouncing resize events in production for better performance

</details>

<details>
<summary>Exercise 2: useFetch Hook</summary>

```jsx
function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchData() {
      try {
        setLoading(true);
        setError(null);

        const response = await fetch(url, options);

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        const json = await response.json();

        if (!cancelled) {
          setData(json);
          setLoading(false);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message);
          setLoading(false);
        }
      }
    }

    fetchData();

    return () => {
      cancelled = true;
    };
  }, [url]); // Re-fetch when URL changes

  return { data, loading, error };
}

// Usage
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!user) return null;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

**Key points:**
- Cancelled flag prevents setting state after component unmounts (race condition)
- Error handling for both network errors and HTTP errors
- Loading state tracks fetch progress for better UX
- URL in dependency array triggers re-fetch when it changes
- For production, consider using TanStack Query instead for caching and revalidation

</details>

---

## What You Learned

This module covered:

- **useState**: Manages component state with functional updates to avoid stale closures
- **useEffect**: Handles side effects (data fetching, subscriptions, DOM manipulation) with cleanup functions and dependencies
- **useRef**: Stores mutable values and DOM references without triggering re-renders
- **useContext**: Accesses shared state from Context providers without prop drilling
- **Custom Hooks**: Extracts reusable stateful logic following the "use" naming convention
- **Performance Hooks**: useMemo and useCallback prevent unnecessary recalculations and re-renders

**Key takeaway**: Hooks let you add state and side effects to function components while keeping logic reusable and testable through custom hooks.

---

## Real-World Application

This week at work, you might use these concepts to:

- Build a notification system with WebSocket connections managed via useEffect cleanup
- Create a useFetch custom hook to handle loading, error, and data states across multiple components
- Implement a theme switcher using useContext to avoid passing theme props through every component
- Optimize expensive filtering or sorting operations with useMemo
- Build a useLocalStorage hook that syncs state with browser storage

---

<details>
<summary>Exercise 3: usePrevious Hook</summary>

```jsx
function usePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  });

  return ref.current;
}

// Usage
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <p>
        {count > prevCount
          ? 'Increased'
          : count < prevCount
          ? 'Decreased'
          : 'No change'}
      </p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
    </div>
  );
}
```

**Key points:**
- useRef stores values without triggering re-renders
- The effect runs after render, so ref.current holds the previous value during render
- No dependency array means effect runs after every render
- Useful for comparing previous and current values in animations or transitions
- First render returns undefined since there's no previous value yet

</details>

## Further Reading

- [React.dev - useState](https://react.dev/reference/react/useState)
- [React.dev - useEffect](https://react.dev/reference/react/useEffect)
- [React.dev - Rules of Hooks](https://react.dev/warnings/invalid-hook-call-warning)

---

**Navigation**: [← Previous Module](./02-component-patterns.md) | [Next Module →](./04-state-management-basics.md)
