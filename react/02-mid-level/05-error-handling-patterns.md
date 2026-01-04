# Error Handling Patterns

Build resilient applications that fail gracefully.

## Learning Objectives

By the end of this module, you will:
- Implement error boundaries for component crashes
- Handle async errors effectively
- Display user-friendly error messages
- Create fallback UIs and recovery mechanisms

## Error Boundaries

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

## Error Fallback Components

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

## Async Error Handling

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

## Error Types and Handling

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

## User-Friendly Error Display

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

## Form Error Handling

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

## Logging and Monitoring

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

## Practice Exercises

1. Implement tiered error boundaries for different app sections
2. Create a toast notification system for errors
3. Build retry logic with exponential backoff

## Further Reading

- [React Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)
- [react-error-boundary](https://github.com/bvaughn/react-error-boundary)
