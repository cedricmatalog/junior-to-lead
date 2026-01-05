# Debugging Techniques

## Week Eight: The Mystery of the Missing Messages

Week eight starts with chaos.

"Messages are disappearing," the QA lead reports. "Users switch conversations, and sometimes the new messages don't show up."

You check the code. The API returns the right data. The component receives it. But somewhere between fetch and render, messages vanish into thin air.

"Time to put on your detective hat," Marcus says. "Debugging isn't about luck—it's about method."

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **React DevTools** | X-ray vision - see inside any component's props and state |
| **Error Boundaries** | Safety nets - catch crashes so the whole app doesn't collapse |
| **Breakpoints** | A pause button - freeze execution and examine everything |

Keep these in mind. They'll click as we build.

---

## Prerequisites

Module 07 (Testing Fundamentals) - Understanding component behavior and state to debug effectively.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Use React DevTools to inspect component props, state, and hooks in real-time
- [ ] Debug with advanced console methods beyond console.log (table, time, trace)
- [ ] Set conditional breakpoints and use debugger statement to pause execution
- [ ] Identify and fix race conditions in async code with cancellation patterns
- [ ] Implement error boundaries to catch and handle render errors gracefully
- [ ] Apply systematic debugging methodology to isolate and resolve issues

---

## Time Estimate

- **Reading**: 55-70 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Debug issues consistently over 4-6 weeks to internalize the process

---

## Chapter 1: Your New Best Friend — React DevTools

"First things first," Sarah says, pulling up Chrome. "Install React DevTools."

The extension adds two tabs to your browser's developer tools: **Components** and **Profiler**.

### Components Tab

"This is X-ray vision for React," Marcus explains.

- **Inspect component tree** - See props, state, hooks for any component
- **Search components** - Find by name or filter by type
- **Edit state/props** - Modify values live to test scenarios
- **Highlight updates** - See which components re-render

You click on the ChatWindow component. There it is—the messages array, right in the state. Empty when it shouldn't be.

"Give your components good names," Sarah adds. "Anonymous functions are impossible to find."

```jsx
// Hard to find in DevTools
const MyContext = createContext();

// Easy to find
const MyContext = createContext();
MyContext.displayName = 'MyContext';

// Named function for better DevTools display
const MemoizedComponent = memo(function MemoizedComponent(props) {
  return <div>{props.value}</div>;
});
```

Here's what the React DevTools component tree looks like:

```
┌──────────────────────────────────────────────────────────┐
│         React DevTools Component Tree                    │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Components Tab:                                        │
│                                                          │
│  ▼ App                                                   │
│    ├─ props: {}                                          │
│    ├─ hooks:                                             │
│    │   └─ State: { user: {...} }                        │
│    │                                                     │
│    ├─▼ ChatWindow                                        │
│    │   ├─ props: { conversationId: "abc123" }           │
│    │   ├─ hooks:                                         │
│    │   │   ├─ State: { messages: [...] }                │
│    │   │   ├─ Effect                                     │
│    │   │   └─ Ref: { current: null }                    │
│    │   │                                                 │
│    │   ├─▼ MessageList                                   │
│    │   │   ├─ props: { messages: [...] }                │
│    │   │   └─▼ MessageItem (×3)                          │
│    │   │       ├─ props: { message: {...} }             │
│    │   │       └─ key: "msg-1"                           │
│    │   │                                                 │
│    │   └─► MessageInput                                  │
│    │       ├─ props: { onSend: ƒ }                       │
│    │       └─ hooks:                                     │
│    │           └─ State: { text: "typing..." }          │
│    │                                                     │
│    └─► Sidebar                                           │
│        └─ props: { user: {...} }                         │
│                                                          │
│  Features:                                              │
│  • Click any component to see props/state              │
│  • Edit values live to test scenarios                  │
│  • Search by component name                            │
│  • Highlight updates (see what re-renders)             │
│  • View component source code location                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Chapter 2: Beyond console.log

"Everyone starts with `console.log`," Marcus says. "But there's so much more."

```jsx
// Group related logs
console.group('User Data');
console.log('Name:', user.name);
console.log('Email:', user.email);
console.groupEnd();

// Tabular data - perfect for arrays of objects
console.table(users);

// Timing
console.time('fetch');
await fetch('/api/data');
console.timeEnd('fetch'); // fetch: 234ms

// Conditional logging
console.assert(user.age > 0, 'Age should be positive', user);

// Stack trace - how did we get here?
console.trace('How did we get here?');

// Styled output - make important things stand out
console.log('%cImportant!', 'color: red; font-size: 20px');
```

You add `console.time` around the message fetch. The timing looks normal. The bug is somewhere else.

---

## Chapter 3: The Power of Breakpoints

"Stop guessing," Sarah says. "Freeze time and look."

```jsx
function handleSubmit(data) {
  debugger; // Execution pauses here when DevTools is open

  validateData(data);
  submitToAPI(data);
}
```

The `debugger` statement is a hard pause. But there's more control in the Sources panel:

**Breakpoint types:**
- **Line breakpoints** - Pause at specific line
- **Conditional breakpoints** - Pause only when condition is true
- **DOM breakpoints** - Pause when DOM node changes
- **XHR breakpoints** - Pause on specific network requests

You set a conditional breakpoint: pause only when `conversationId === 'abc123'`. The culprit reveals itself—you're setting state for the wrong conversation.

---

## Chapter 4: Finding the Race Condition

The bug is clear now. When users switch conversations fast, responses arrive out of order.

```jsx
// BUG: Race condition in message loading
function ChatWindow({ conversationId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    async function loadMessages() {
      const data = await fetchMessages(conversationId);
      setMessages(data); // Bug: might set messages for wrong conversation
    }
    loadMessages();
  }, [conversationId]);
}
```

"The fetch for conversation A is still in flight when you switch to B," Marcus explains. "A's response arrives after B's and overwrites it."

The fix: cancel stale requests.

```jsx
// FIX: Cancel stale requests
function ChatWindow({ conversationId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    let cancelled = false;

    async function loadMessages() {
      const data = await fetchMessages(conversationId);
      if (!cancelled) {
        setMessages(data);
      }
    }

    loadMessages();
    return () => { cancelled = true; };
  }, [conversationId]);
}
```

**Debugging steps you used:**
1. Added `console.log('Loading messages for:', conversationId)` to see request order
2. Used Network tab to see request/response timing
3. Found responses arriving out of order
4. Added cancellation flag to ignore stale responses

---

## Chapter 5: The Typing Indicator That Wouldn't Stop

Another bug report: "User is typing..." shows indefinitely.

```jsx
// BUG: WebSocket event handler with stale closure
function TypingIndicator({ conversationId }) {
  const [typingUsers, setTypingUsers] = useState([]);

  useEffect(() => {
    socket.on('typing', (data) => {
      if (data.conversationId === conversationId) { // Bug: stale conversationId
        setTypingUsers(prev => [...prev, data.userId]);
      }
    });

    socket.on('stopped_typing', (data) => {
      setTypingUsers(prev => prev.filter(id => id !== data.userId));
    });
  }, []); // Missing conversationId dependency!
}
```

"Stale closure," Sarah identifies immediately. "The handler captures the old `conversationId` and never updates."

```jsx
// FIX: Include dependency and proper cleanup
function TypingIndicator({ conversationId }) {
  const [typingUsers, setTypingUsers] = useState([]);

  useEffect(() => {
    function handleTyping(data) {
      if (data.conversationId === conversationId) {
        setTypingUsers(prev => [...prev, data.userId]);
      }
    }

    function handleStoppedTyping(data) {
      if (data.conversationId === conversationId) {
        setTypingUsers(prev => prev.filter(id => id !== data.userId));
      }
    }

    socket.on('typing', handleTyping);
    socket.on('stopped_typing', handleStoppedTyping);

    return () => {
      socket.off('typing', handleTyping);
      socket.off('stopped_typing', handleStoppedTyping);
      setTypingUsers([]); // Clear on conversation change
    };
  }, [conversationId]);
}
```

"ESLint's `react-hooks/exhaustive-deps` would have caught this," Marcus points out. "Don't ignore those warnings."

---

## Chapter 6: Error Boundaries — The Safety Net

Friday afternoon. A user uploads a malformed message, and the entire chat app crashes.

"One bad component shouldn't take down the whole page," Sarah says. "That's what error boundaries are for."

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

// Wrap sections that might fail
<ErrorBoundary>
  <UserProfile />
</ErrorBoundary>
```

**Error boundaries don't catch:**
- Event handlers (use try/catch)
- Async code (use try/catch or .catch())
- Server-side rendering
- Errors in the error boundary itself

> **Note**: React still requires class components for error boundaries. If you want a function-component-friendly API, use a library like `react-error-boundary`, which wraps a class boundary for you.

---

## Chapter 7: The Double Fetch Mystery

"Why are we loading messages twice?" QA asks. The Network tab shows duplicate requests.

```jsx
// BUG: Effect runs twice due to object dependency
function MessageList({ filters }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    fetchMessages(filters).then(setMessages);
  }, [filters]); // Bug: filters object is recreated each render
}

// Parent component recreating filters every render
function ChatWindow() {
  const [search, setSearch] = useState('');

  return (
    <MessageList filters={{ search, limit: 50 }} /> // New object each render!
  );
}
```

The parent creates a new `filters` object on every render. Objects are compared by reference, so React thinks the dependency changed.

```jsx
// FIX: Memoize the filters object
function ChatWindow() {
  const [search, setSearch] = useState('');
  const filters = useMemo(() => ({ search, limit: 50 }), [search]);

  return <MessageList filters={filters} />;
}
```

**Debugging steps:**
1. Used React DevTools Profiler to see component re-renders
2. Added `console.log('Effect running')` to count executions
3. Found root cause: parent recreating props object

---

## Chapter 8: The Systematic Approach

Marcus pins a checklist to the wall:

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

---

## Chapter 9: The Common Bug Patterns

Sarah writes the patterns you'll see over and over:

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

---

## Chapter 10: Your Debugging Toolkit

```jsx
// Add to your debugging toolkit
const useDebugRender = (componentName) => {
  const renderCount = useRef(0);
  renderCount.current += 1;

  useEffect(() => {
    console.log(`[${componentName}] Render #${renderCount.current}`);
  });
};

// Usage
function ProblematicComponent() {
  useDebugRender('ProblematicComponent');
  // ... component code
}
```

**The debugging checklist for React apps:**

1. **Check the console** - Look for warnings about keys, effects, state updates
2. **React DevTools Components** - Inspect props and state values
3. **React DevTools Profiler** - Find unnecessary re-renders
4. **Network tab** - Verify API calls happen correctly
5. **Add strategic console.logs** - Track execution order and values
6. **Check ESLint warnings** - Especially react-hooks rules

By Friday, the chat app is stable. Messages arrive in order. The typing indicator behaves. Errors are caught before they crash the page.

"Debugging is a skill," Sarah says. "The more bugs you fix, the faster you spot them."

---

## Chapter 11: The Error Handling Playbook

Before you leave, Marcus pulls together everything you've learned about errors into one unified approach.

"You've seen errors everywhere this week—in effects, in handlers, in renders. Let me show you the complete picture."

### The Error Landscape

```
┌─────────────────────────────────────────────────────────────────┐
│                     React Error Handling                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────┐ │
│  │  Render Errors  │    │  Event Errors   │    │ Async Errors│ │
│  │                 │    │                 │    │             │ │
│  │ Error Boundary  │    │   try/catch     │    │  try/catch  │ │
│  │                 │    │                 │    │  .catch()   │ │
│  │                 │    │                 │    │  cancelled  │ │
│  └─────────────────┘    └─────────────────┘    └─────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Pattern 1: Event Handler Errors

```jsx
// Event handlers need explicit try/catch
function SubmitButton({ onSubmit }) {
  const handleClick = async () => {
    try {
      await onSubmit();
    } catch (error) {
      // Error boundaries won't catch this!
      console.error('Submit failed:', error);
      showToast('Something went wrong. Please try again.');
    }
  };

  return <button onClick={handleClick}>Submit</button>;
}
```

### Pattern 2: Race Condition Prevention

```jsx
// The cancelled flag pattern for async operations
function DataFetcher({ id }) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchData() {
      try {
        setError(null);
        const result = await api.fetch(id);
        if (!cancelled) {
          setData(result);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message);
        }
      }
    }

    fetchData();
    return () => { cancelled = true; };
  }, [id]);

  if (error) return <ErrorMessage message={error} />;
  if (!data) return <Loading />;
  return <DataDisplay data={data} />;
}
```

### Pattern 3: API Error Handling

```jsx
// Centralized API error handling
async function apiCall(endpoint, options = {}) {
  try {
    const response = await fetch(endpoint, options);

    if (!response.ok) {
      // Handle HTTP errors
      const error = new Error(`HTTP ${response.status}`);
      error.status = response.status;

      if (response.status === 401) {
        // Redirect to login
        window.location.href = '/login';
      }

      throw error;
    }

    return await response.json();
  } catch (error) {
    // Handle network errors
    if (error.name === 'TypeError') {
      throw new Error('Network error. Please check your connection.');
    }
    throw error;
  }
}

// Usage in component
const [error, setError] = useState(null);

const loadData = async () => {
  try {
    const data = await apiCall('/api/messages');
    setMessages(data);
  } catch (err) {
    setError(err.message);
  }
};
```

### Pattern 4: Graceful Degradation

```jsx
// Show fallback UI when features fail
function UserAvatar({ userId }) {
  const [imageError, setImageError] = useState(false);
  const [user, setUser] = useState(null);

  if (imageError || !user?.avatar) {
    return (
      <div className="avatar-fallback">
        {user?.name?.charAt(0) || '?'}
      </div>
    );
  }

  return (
    <img
      src={user.avatar}
      alt={user.name}
      onError={() => setImageError(true)}
    />
  );
}
```

### The Complete Error Strategy

"Here's how it all fits together," Marcus summarizes:

| Error Type | Where It Happens | How to Handle |
|------------|------------------|---------------|
| Render errors | During component render | Error Boundary |
| Event errors | onClick, onSubmit, etc. | try/catch in handler |
| Effect errors | useEffect async code | try/catch + cancelled flag |
| API errors | fetch/axios calls | Centralized error handler |
| Image/media errors | Resource loading | onError + fallback UI |

"The key insight: different errors need different solutions. Error boundaries are powerful, but they only catch render errors. Everything else needs explicit handling."

---

## Debugging Decision Tree

| Symptom | Check First |
|---------|-------------|
| Component not updating | State mutation? Missing key? |
| Effect running too often | Object/array dependency? |
| Stale data in handler | Stale closure? Missing dependency? |
| White screen of death | Check console for errors |
| Wrong data displayed | Check Network tab response |

## Common Mistakes

1. **Not reading the console** - Warnings are there for a reason
2. **Changing multiple things at once** - One change per test
3. **Ignoring ESLint warnings** - They catch real bugs
4. **Not using DevTools** - Stop guessing, start inspecting

## Practice Exercises

1. Use React DevTools to find why a component re-renders unnecessarily
2. Debug a stale closure issue in a timer component
3. Add error boundaries to handle API failures gracefully

### Solutions

<details>
<summary>Exercise 1: Finding Unnecessary Re-renders</summary>

**Problem Code:**

```jsx
// Parent component recreating objects on every render
function Dashboard() {
  const [count, setCount] = useState(0);

  // BUG: New object on every render!
  const user = {
    name: 'John Doe',
    email: 'john@example.com'
  };

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Clicked {count} times
      </button>
      <UserProfile user={user} />
    </div>
  );
}

const UserProfile = memo(function UserProfile({ user }) {
  console.log('UserProfile rendered');
  return <div>{user.name} ({user.email})</div>;
});
```

**Debugging Steps:**

1. Open React DevTools
2. Go to Components tab
3. Click the settings icon (gear)
4. Enable "Highlight updates when components render"
5. Click the button in the Dashboard
6. Notice UserProfile flashes (re-renders) even though user data hasn't changed
7. Select UserProfile in the tree
8. Look at props in the right panel
9. Notice `user` object reference changes every render

**Fixed Code:**

```jsx
function Dashboard() {
  const [count, setCount] = useState(0);

  // FIX: Memoize the object
  const user = useMemo(() => ({
    name: 'John Doe',
    email: 'john@example.com'
  }), []); // Empty deps = never changes

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Clicked {count} times
      </button>
      <UserProfile user={user} />
    </div>
  );
}

const UserProfile = memo(function UserProfile({ user }) {
  console.log('UserProfile rendered');
  return <div>{user.name} ({user.email})</div>;
});
```

**Alternative Fix - Move Outside Component:**

```jsx
// Even better: if data is static, move it outside
const USER_DATA = {
  name: 'John Doe',
  email: 'john@example.com'
};

function Dashboard() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Clicked {count} times
      </button>
      <UserProfile user={USER_DATA} />
    </div>
  );
}
```

**Key points:**
- Objects/arrays created in render are new instances each time
- React DevTools highlight feature makes re-renders visible
- useMemo prevents creating new object references
- Static data should live outside components when possible
- Use Profiler tab to see which components render and why

</details>

<details>
<summary>Exercise 2: Debugging Stale Closure in Timer</summary>

**Problem Code:**

```jsx
function Timer() {
  const [count, setCount] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    if (!isRunning) return;

    // BUG: Stale closure! count never updates inside interval
    const timer = setInterval(() => {
      console.log('Count:', count); // Always logs 0
      setCount(count + 1); // Always sets to 1
    }, 1000);

    return () => clearInterval(timer);
  }, [isRunning]); // Missing count dependency!

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

**Debugging Steps:**

1. Add console.log inside interval callback
2. Notice it always logs the same value
3. Check ESLint warnings - should show missing dependency
4. Add count to dependency array - but this recreates interval every second!
5. Solution: use functional update form

**Fixed Code - Solution 1 (Functional Updates):**

```jsx
function Timer() {
  const [count, setCount] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    if (!isRunning) return;

    // FIX: Use functional update to avoid stale closure
    const timer = setInterval(() => {
      setCount(prevCount => {
        console.log('Previous count:', prevCount); // Shows correct value
        return prevCount + 1;
      });
    }, 1000);

    return () => clearInterval(timer);
  }, [isRunning]); // Only isRunning needed now

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

**Fixed Code - Solution 2 (useRef):**

```jsx
function Timer() {
  const [count, setCount] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  const countRef = useRef(count);

  // Keep ref in sync with state
  useEffect(() => {
    countRef.current = count;
  }, [count]);

  useEffect(() => {
    if (!isRunning) return;

    const timer = setInterval(() => {
      const current = countRef.current;
      console.log('Count:', current); // Always current value
      setCount(current + 1);
    }, 1000);

    return () => clearInterval(timer);
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

**Key points:**
- Closures capture variables from their creation time
- Functional updates `setState(prev => prev + 1)` always get latest value
- useRef can store mutable values that don't trigger re-renders
- ESLint exhaustive-deps rule catches these issues
- console.log in callbacks helps identify stale values

</details>

<details>
<summary>Exercise 3: Error Boundaries for API Failures</summary>

```jsx
// ErrorBoundary.jsx
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log to error reporting service
    console.error('Error caught by boundary:', error, errorInfo);

    // Could send to Sentry, LogRocket, etc.
    // logErrorToService(error, errorInfo);

    this.setState({ errorInfo });
  }

  resetError = () => {
    this.setState({ hasError: false, error: null, errorInfo: null });
  };

  render() {
    if (this.state.hasError) {
      return (
        <div style={{
          padding: '2rem',
          textAlign: 'center',
          backgroundColor: '#fee',
          border: '1px solid #fcc',
          borderRadius: '4px',
        }}>
          <h2>Something went wrong</h2>
          <p style={{ color: '#c00' }}>{this.state.error?.message}</p>

          {process.env.NODE_ENV === 'development' && this.state.errorInfo && (
            <details style={{ textAlign: 'left', marginTop: '1rem' }}>
              <summary>Error Details (dev only)</summary>
              <pre style={{ fontSize: '0.875rem', overflow: 'auto' }}>
                {this.state.errorInfo.componentStack}
              </pre>
            </details>
          )}

          <button
            onClick={this.resetError}
            style={{
              marginTop: '1rem',
              padding: '0.5rem 1rem',
              backgroundColor: '#0066cc',
              color: 'white',
              border: 'none',
              borderRadius: '4px',
              cursor: 'pointer',
            }}
          >
            Try Again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// UserData.jsx - Component that fetches data
function UserData({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);

        if (!response.ok) {
          throw new Error(`Failed to fetch user: ${response.status}`);
        }

        const data = await response.json();

        if (!cancelled) {
          setUser(data);
          setError(null);
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
    return () => { cancelled = true; };
  }, [userId]);

  // Throw error to be caught by error boundary
  if (error) {
    throw new Error(error);
  }

  if (loading) return <p>Loading...</p>;
  if (!user) return null;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// App.jsx - Wrapping components with error boundaries
function App() {
  return (
    <div>
      <h1>User Dashboard</h1>

      {/* Separate error boundary per section */}
      <ErrorBoundary>
        <section>
          <h2>User Profile</h2>
          <UserData userId="123" />
        </section>
      </ErrorBoundary>

      <ErrorBoundary>
        <section>
          <h2>User Settings</h2>
          <UserSettings userId="123" />
        </section>
      </ErrorBoundary>

      {/* Rest of app still works if one section fails */}
      <section>
        <h2>Navigation</h2>
        <nav>Always visible even if sections above fail</nav>
      </section>
    </div>
  );
}

// Alternative: Async error handling without error boundary
function UserDataAlternative({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      try {
        setLoading(true);
        setError(null); // Clear previous errors

        const response = await fetch(`/api/users/${userId}`);

        if (!response.ok) {
          throw new Error(`Failed to fetch user: ${response.status}`);
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
    return () => { cancelled = true; };
  }, [userId]);

  if (loading) return <p>Loading...</p>;

  // Show error inline instead of throwing
  if (error) {
    return (
      <div role="alert" style={{ color: 'red', padding: '1rem', border: '1px solid red' }}>
        <strong>Error:</strong> {error}
        <button onClick={() => window.location.reload()}>
          Retry
        </button>
      </div>
    );
  }

  if (!user) return null;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

**Key points:**
- Error boundaries catch render errors in child components
- They don't catch event handler or async errors (use try/catch)
- Wrap each major section separately to isolate failures
- Reset functionality allows recovery without full page reload
- Show stack trace in development, hide in production
- Alternative: handle errors in state without throwing

</details>

---

## What You Learned

This module covered:

- **React DevTools**: Inspect component trees, props, state, and hooks; highlight re-renders visually
- **Console Methods**: table, time, trace, assert provide richer debugging info than console.log
- **Breakpoints**: Pause execution at specific lines or conditions to inspect state
- **Race Conditions**: Cancel stale async operations with cleanup flags to prevent bugs
- **Error Boundaries**: Catch render errors without crashing the entire app
- **Systematic Debugging**: Reproduce, isolate, hypothesize, test - a methodical approach

**Key takeaway**: Debugging is a skill built on method, not luck - use tools systematically and fix the cause, not just the symptom.

---

## Real-World Application

This week at work, you might use these concepts to:

- Track down why a component re-renders too often using React DevTools Profiler
- Fix race conditions in data fetching when users navigate quickly
- Add error boundaries to isolate failures in third-party widgets
- Debug stale closures in event handlers with breakpoints and console logs
- Identify memory leaks from uncleaned subscriptions and timers

---

## Further Reading

- [React DevTools Tutorial](https://react.dev/learn/react-developer-tools)
- [Chrome DevTools Documentation](https://developer.chrome.com/docs/devtools/)

---

**Navigation**: [← Previous Module](./07-testing-fundamentals.md) | [Next Module →](./09-git-workflows.md)
