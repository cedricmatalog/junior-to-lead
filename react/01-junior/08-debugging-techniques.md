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

> **React 19 Note**: React 19 introduces functional error boundaries using the new `use` hook and `ErrorBoundary` component from `react-error-boundary`. The class-based approach above still works, but modern codebases increasingly use the library approach for cleaner syntax.

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

## Further Reading

- [React DevTools Tutorial](https://react.dev/learn/react-developer-tools)
- [Chrome DevTools Documentation](https://developer.chrome.com/docs/devtools/)
