# Error Handling Patterns

> **Last reviewed**: 2026-01-06


## Week Six: The Outage Drill

The API is flaky this week. Users are hitting broken screens, and the support inbox is filling up. Sarah asks for a resilience plan: "If something fails, you need to recover without losing trust." Marcus adds that errors shouldn't blow up the whole app. This week is about designing predictable recovery paths -- from component crashes to failed requests -- so the UI fails gracefully and guides users back on track.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Error boundaries** | Circuit breakers - stop a failure from spreading |
| **Fallback UI** | A detour sign - keep users moving |
| **Async errors** | Dropped calls - handle retries and timeouts |
| **Logging** | Flight recorder - capture what went wrong |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 04 (API Design and Integration) - Familiarity with async requests and loading states.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Implement error boundaries for component crashes
- [ ] Handle async errors and retries consistently
- [ ] Build fallback UIs with recovery actions
- [ ] Classify error types and map them to UX
- [ ] Display form and validation errors clearly
- [ ] Instrument logging and monitoring hooks

---

## Time Estimate

- **Reading**: 60-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice error-handling patterns over 4-6 weeks

---

## Chapter 1: Error Boundaries

When a component crashes, the rest of the page shouldn't go down with it.

Catch JavaScript errors in component trees.

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({ errorInfo });
    // Log to error reporting service
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <ErrorFallback
          error={this.state.error}
          resetError={() => this.setState({ hasError: false })}
        />
      );
    }
    return this.props.children;
  }
}

// Reusable hook version using react-error-boundary
import { ErrorBoundary } from 'react-error-boundary';

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={(error, info) => logErrorToService(error, info)}
      onReset={() => {
        // Reset app state
      }}
    >
      <MainApp />
    </ErrorBoundary>
  );
}
```

## Chapter 2: Error Fallback Components

Sarah wants every failure to show a friendly recovery path, not a dead end.

```jsx
function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert" className="error-container">
      <h2>Something went wrong</h2>
      <pre className="error-message">{error.message}</pre>
      <div className="error-actions">
        <button onClick={resetErrorBoundary}>Try again</button>
        <button onClick={() => window.location.href = '/'}>
          Go to homepage
        </button>
      </div>
    </div>
  );
}

// Different fallbacks for different sections
function App() {
  return (
    <ErrorBoundary FallbackComponent={PageError}>
      <Header />
      <ErrorBoundary FallbackComponent={SidebarError}>
        <Sidebar />
      </ErrorBoundary>
      <ErrorBoundary FallbackComponent={ContentError}>
        <MainContent />
      </ErrorBoundary>
    </ErrorBoundary>
  );
}
```

## Chapter 3: Async Error Handling

Network failures are inevitable. You need consistent retry and timeout strategies.

```jsx
// With try-catch
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) {
      throw new ApiError('Failed to fetch', response.status);
    }
    return await response.json();
  } catch (error) {
    if (error instanceof ApiError) {
      // Handle API errors
      throw error;
    }
    // Handle network errors
    throw new NetworkError('Network request failed');
  }
}

// In components with TanStack Query
function DataComponent() {
  const { data, error, isError, refetch } = useQuery({
    queryKey: ['data'],
    queryFn: fetchData,
    retry: (failureCount, error) => {
      // Don't retry on 4xx errors
      if (error.status >= 400 && error.status < 500) return false;
      return failureCount < 3;
    },
  });

  if (isError) {
    return <ErrorDisplay error={error} onRetry={refetch} />;
  }

  return <DataDisplay data={data} />;
}
```

## Chapter 4: Error Types and Handling

Different errors need different UX. Not every failure is fatal.

```jsx
// Define error types
class AppError extends Error {
  constructor(message, code, isOperational = true) {
    super(message);
    this.code = code;
    this.isOperational = isOperational;
  }
}

class ValidationError extends AppError {
  constructor(message, fields) {
    super(message, 'VALIDATION_ERROR');
    this.fields = fields;
  }
}

class NetworkError extends AppError {
  constructor(message) {
    super(message, 'NETWORK_ERROR');
  }
}

class AuthError extends AppError {
  constructor(message) {
    super(message, 'AUTH_ERROR');
  }
}

// Handle by type
function handleError(error) {
  if (error instanceof ValidationError) {
    return { type: 'validation', fields: error.fields };
  }

  if (error instanceof NetworkError) {
    return { type: 'network', message: 'Please check your connection' };
  }

  if (error instanceof AuthError) {
    redirectToLogin();
    return null;
  }

  // Unknown error - log and show generic message
  logErrorToService(error);
  return { type: 'unknown', message: 'Something went wrong' };
}
```

## Chapter 5: User-Friendly Error Display

The message matters. Users need clarity, not stack traces.

```jsx
function ErrorDisplay({ error, onRetry }) {
  const errorInfo = useMemo(() => {
    switch (error.code) {
      case 'NETWORK_ERROR':
        return {
          title: 'Connection Problem',
          message: 'Please check your internet connection and try again.',
          icon: <WifiOffIcon />,
          canRetry: true,
        };
      case 'NOT_FOUND':
        return {
          title: 'Not Found',
          message: "We couldn't find what you're looking for.",
          icon: <SearchOffIcon />,
          canRetry: false,
        };
      case 'PERMISSION_DENIED':
        return {
          title: 'Access Denied',
          message: "You don't have permission to view this.",
          icon: <LockIcon />,
          canRetry: false,
        };
      default:
        return {
          title: 'Something Went Wrong',
          message: 'An unexpected error occurred. Please try again.',
          icon: <AlertIcon />,
          canRetry: true,
        };
    }
  }, [error.code]);

  return (
    <div className="error-display">
      {errorInfo.icon}
      <h3>{errorInfo.title}</h3>
      <p>{errorInfo.message}</p>
      {errorInfo.canRetry && (
        <button onClick={onRetry}>Try Again</button>
      )}
    </div>
  );
}
```

## Chapter 6: Form Error Handling

Validation errors need to be specific and immediate.

```jsx
function RegistrationForm() {
  const [errors, setErrors] = useState({});
  const [submitError, setSubmitError] = useState(null);

  async function handleSubmit(data) {
    setErrors({});
    setSubmitError(null);

    try {
      await registerUser(data);
    } catch (error) {
      if (error instanceof ValidationError) {
        // Field-level errors
        setErrors(error.fields);
      } else if (error.status === 409) {
        // Email already exists
        setErrors({ email: 'This email is already registered' });
      } else {
        // General error
        setSubmitError('Registration failed. Please try again.');
      }
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      {submitError && (
        <div role="alert" className="error-banner">
          {submitError}
        </div>
      )}

      <Field name="email" error={errors.email} />
      <Field name="password" error={errors.password} />

      <button type="submit">Register</button>
    </form>
  );
}
```

## Chapter 7: Logging and Monitoring

If errors aren't captured, you can't fix what you can't see.

```jsx
// Error logging service
function logErrorToService(error, context = {}) {
  // Send to Sentry, LogRocket, etc.
  if (process.env.NODE_ENV === 'production') {
    Sentry.captureException(error, {
      extra: context,
    });
  } else {
    console.error('Error:', error, context);
  }
}

// Global error handler
window.addEventListener('unhandledrejection', (event) => {
  logErrorToService(event.reason, { type: 'unhandledRejection' });
});
```

---

## Common Mistakes

1. **Catching everything in one place** - Different layers need different recovery strategies.
2. **Showing raw errors to users** - Translate failures into actionable messages.
3. **Ignoring retries and timeouts** - Network errors need explicit handling.
4. **Logging too late** - Capture errors where they happen, not after the fact.


Example:
```jsx
// ❌ Swallow errors silently
try { await saveProfile(); } catch {}

// ✅ Capture and surface errors
try { await saveProfile(); } catch (err) { setError(err.message); }
```

## Practice Exercises

1. Implement tiered error boundaries for different app sections
2. Create a toast notification system for errors
3. Build retry logic with exponential backoff

### Solutions

<details>
<summary>Exercise 1: Tiered Error Boundaries</summary>

```jsx
function AppShell() {
  const [routeKey, setRouteKey] = useState(0);

  return (
    <ErrorBoundary
      FallbackComponent={AppFallback}
      onReset={() => setRouteKey((key) => key + 1)}
      resetKeys={[routeKey]}
    >
      <Header />
      <main>
        <ErrorBoundary FallbackComponent={DashboardFallback}>
          <Dashboard />
        </ErrorBoundary>
        <ErrorBoundary FallbackComponent={BillingFallback}>
          <Billing />
        </ErrorBoundary>
      </main>
    </ErrorBoundary>
  );
}
```

**Key points:**
- Critical sections get their own boundaries.
- A dashboard crash doesn't take down billing.
- App-level fallback covers the full page.

</details>

<details>
<summary>Exercise 2: Error Toasts</summary>

```jsx
const ToastContext = createContext(null);

function ToastProvider({ children }) {
  const [toasts, setToasts] = useState([]);

  const pushToast = (message) => {
    const id = Date.now();
    setToasts((prev) => [...prev, { id, message }]);
    setTimeout(() => dismissToast(id), 5000);
  };

  const dismissToast = (id) => {
    setToasts((prev) => prev.filter((toast) => toast.id !== id));
  };

  return (
    <ToastContext.Provider value={{ pushToast }}>
      {children}
      <div className="toast-stack" role="status" aria-live="polite">
        {toasts.map((toast) => (
          <div key={toast.id} className="toast" onClick={() => dismissToast(toast.id)}>
            {toast.message}
          </div>
        ))}
      </div>
    </ToastContext.Provider>
  );
}

function useToast() {
  return useContext(ToastContext);
}
```

**Key points:**
- Centralized toast state avoids prop drilling.
- Click-to-dismiss keeps UX predictable.
- Any component can report errors via `useToast`.

</details>

<details>
<summary>Exercise 3: Exponential Backoff</summary>

```jsx
async function fetchWithBackoff(url, retries = 3, delay = 500, signal) {
  try {
    const response = await fetch(url, { signal });
    if (!response.ok) throw new Error('Request failed');
    return response.json();
  } catch (error) {
    if (retries <= 0) throw error;
    const jitter = Math.random() * 200;
    await new Promise((resolve) => setTimeout(resolve, delay + jitter));
    return fetchWithBackoff(url, retries - 1, delay * 2, signal);
  }
}
```

**Key points:**
- Delay doubles each retry to reduce server load.
- Jitter avoids retry spikes.
- Errors bubble after the final attempt.
- Works with existing query or fetch utilities.

</details>

---

## What You Learned

This module covered:

- **Error boundaries**: Isolating crashes to a subtree
- **Fallback UI**: Keeping users moving during failures
- **Async errors**: Retries, timeouts, and network recovery
- **Error taxonomy**: Matching error types to UX
- **Logging**: Capturing actionable context

**Key takeaway**: Resilience comes from predictable recovery, not perfect code.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add fallback UIs to critical pages
- Improve error messaging on failed API calls
- Add retry logic to flaky endpoints
- Surface field-level form errors clearly
- Instrument error reporting with context

---

## Further Reading

- [React Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)
- [react-error-boundary](https://github.com/bvaughn/react-error-boundary)

---

**Navigation**: [← Previous Module](./05-api-design-integration.md) | [Next Module →](./07-internationalization.md)
