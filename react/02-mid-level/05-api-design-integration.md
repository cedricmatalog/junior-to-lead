# API Design and Integration

> **Last reviewed**: 2026-01-06


## Week Five: The Data Contract

Marketing wants a new personalization API, design wants instant updates, and customer support wants reliable error messages. Sarah tells you the UI can't be stable unless the data layer is. Marcus adds that "fetch in components" is why errors are leaking into the UI. This week you'll design a reusable API layer, integrate caching, and handle real-world failures without spreading messy logic across every screen.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **API layer** | A receptionist - one place to handle requests |
| **Caching** | A fridge - grab what's fresh without reordering |
| **Optimistic updates** | A promise - show the result before the confirmation |
| **Loading states** | A queue sign - set expectations while waiting |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 03 (State at Scale) - Familiarity with client vs server state and caching tools.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Structure a reusable API layer with consistent errors
- [ ] Integrate TanStack Query for caching and retries
- [ ] Implement optimistic updates safely
- [ ] Work with REST and GraphQL APIs confidently
- [ ] Design loading and error states that guide users
- [ ] Choose patterns that reduce duplicate fetch logic

---

## Time Estimate

- **Reading**: 60-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice data integration over 4-6 weeks

---

## Chapter 1: API Layer Architecture

You need a single place to handle auth, errors, and request defaults.

Separate API logic from components.

```jsx
// api/client.js
const BASE_URL = process.env.REACT_APP_API_URL;

class ApiError extends Error {
  constructor(message, status, data) {
    super(message);
    this.status = status;
    this.data = data;
  }
}

async function request(endpoint, options = {}) {
  const url = `${BASE_URL}${endpoint}`;
  const config = {
    headers: {
      'Content-Type': 'application/json',
      ...options.headers,
    },
    ...options,
  };

  // Add auth token if available
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  const response = await fetch(url, config);

  if (!response.ok) {
    const data = await response.json().catch(() => ({}));
    throw new ApiError(
      data.message || 'An error occurred',
      response.status,
      data
    );
  }

  return response.json();
}

export const api = {
  get: (endpoint) => request(endpoint),
  post: (endpoint, data) => request(endpoint, { method: 'POST', body: JSON.stringify(data) }),
  put: (endpoint, data) => request(endpoint, { method: 'PUT', body: JSON.stringify(data) }),
  patch: (endpoint, data) => request(endpoint, { method: 'PATCH', body: JSON.stringify(data) }),
  delete: (endpoint) => request(endpoint, { method: 'DELETE' }),
};

// api/users.js
import { api } from './client';

export const usersApi = {
  getAll: () => api.get('/users'),
  getById: (id) => api.get(`/users/${id}`),
  create: (data) => api.post('/users', data),
  update: (id, data) => api.patch(`/users/${id}`, data),
  delete: (id) => api.delete(`/users/${id}`),
};
```

## Chapter 2: Using with TanStack Query

With a clean API layer, you can now cache data and avoid refetching on every render.

```jsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { usersApi } from './api/users';

// Custom hooks for data fetching
function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: usersApi.getAll,
  });
}

function useUser(id) {
  return useQuery({
    queryKey: ['users', id],
    queryFn: () => usersApi.getById(id),
    enabled: !!id, // Don't fetch if no id
  });
}

function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: usersApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

// Usage in component
function UserList() {
  const { data: users, isLoading, error } = useUsers();
  const createUser = useCreateUser();

  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <div>
      {users.map(user => <UserCard key={user.id} user={user} />)}
      <button onClick={() => createUser.mutate({ name: 'New User' })}>
        Add User
      </button>
    </div>
  );
}
```

## Chapter 3: Optimistic Updates

Sarah wants instant feedback when users favorite a show, even if the server is slow.

Update UI immediately, rollback on error.

```jsx
function useUpdateTodo() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, ...data }) => todosApi.update(id, data),

    onMutate: async (newTodo) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['todos'] });

      // Snapshot current state
      const previousTodos = queryClient.getQueryData(['todos']);

      // Optimistically update
      queryClient.setQueryData(['todos'], (old) =>
        old.map(todo =>
          todo.id === newTodo.id ? { ...todo, ...newTodo } : todo
        )
      );

      // Return context for rollback
      return { previousTodos };
    },

    onError: (err, newTodo, context) => {
      // Rollback on error
      queryClient.setQueryData(['todos'], context.previousTodos);
    },

    onSettled: () => {
      // Always refetch to ensure consistency
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
}
```

## Chapter 4: Error Handling Patterns

Real users hit real failures. You need consistent, reusable error paths.

```jsx
function ErrorMessage({ error }) {
  if (error.status === 401) {
    return <Navigate to="/login" replace />;
  }

  if (error.status === 403) {
    return <p>You don't have permission to view this.</p>;
  }

  if (error.status === 404) {
    return <p>Not found.</p>;
  }

  if (error.status >= 500) {
    return (
      <div>
        <p>Server error. Please try again later.</p>
        <button onClick={() => window.location.reload()}>Retry</button>
      </div>
    );
  }

  return <p>{error.message}</p>;
}

// Retry logic
const { data } = useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  retry: 3,
  retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
});
```

## Chapter 5: GraphQL with Apollo

The analytics team ships a GraphQL endpoint. Your UI still needs the same discipline.

```jsx
import { gql, useQuery, useMutation } from '@apollo/client';

const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
      email
    }
  }
`;

const CREATE_USER = gql`
  mutation CreateUser($input: CreateUserInput!) {
    createUser(input: $input) {
      id
      name
      email
    }
  }
`;

function UserList() {
  const { loading, error, data } = useQuery(GET_USERS);
  const [createUser] = useMutation(CREATE_USER, {
    refetchQueries: [{ query: GET_USERS }],
  });

  if (loading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <div>
      {data.users.map(user => <UserCard key={user.id} user={user} />)}
    </div>
  );
}
```

## Chapter 6: Loading States

Loading spinners and skeletons are UX decisions, not afterthoughts.

```jsx
function UserProfile({ userId }) {
  const { data, isLoading, isFetching, isError } = useUser(userId);

  // Initial load
  if (isLoading) {
    return <ProfileSkeleton />;
  }

  if (isError) {
    return <ErrorMessage />;
  }

  return (
    <div>
      {/* Show subtle indicator for background refetch */}
      {isFetching && <RefetchIndicator />}
      <h1>{data.name}</h1>
    </div>
  );
}

// Skeleton loading
function ProfileSkeleton() {
  return (
    <div className="animate-pulse">
      <div className="h-8 bg-gray-200 rounded w-1/4 mb-4" />
      <div className="h-4 bg-gray-200 rounded w-1/2 mb-2" />
      <div className="h-4 bg-gray-200 rounded w-1/3" />
    </div>
  );
}
```

---

## Common Mistakes

1. **Fetching inside every component** - Centralize requests to keep behavior consistent.
2. **Ignoring non-200 errors** - Always handle error shapes and status codes.
3. **Skipping retries and caching** - Users feel slowness if you refetch too often.
4. **Hiding loading states** - Make waiting explicit to prevent confusion.


Example:
```jsx
// ❌ Ignore error states
const res = await fetch('/api/orders');
const data = await res.json();

// ✅ Handle errors explicitly
const res = await fetch('/api/orders');
if (!res.ok) throw new Error('Request failed');
const data = await res.json();
```

## Practice Exercises

1. Build an API layer with error handling and auth
2. Implement optimistic updates for a todo list
3. Add pagination with infinite scroll

### Solutions

<details>
<summary>Exercise 1: API Layer</summary>

```jsx
const BASE_URL = process.env.REACT_APP_API_URL;

class ApiError extends Error {
  constructor(message, status, data) {
    super(message);
    this.status = status;
    this.data = data;
  }
}

async function request(path, options = {}) {
  const token = localStorage.getItem('token');
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), 10000);
  const response = await fetch(`${BASE_URL}${path}`, {
    ...options,
    signal: options.signal ?? controller.signal,
    headers: {
      'Content-Type': 'application/json',
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...options.headers
    }
  });
  clearTimeout(timeout);

  if (!response.ok) {
    const data = await response.json().catch(() => ({}));
    throw new ApiError(data.message ?? 'Request failed', response.status, data);
  }

  if (response.status === 204) return null;
  const contentType = response.headers.get('Content-Type') ?? '';
  return contentType.includes('application/json') ? response.json() : response.text();
}

export const api = {
  get: (path, options) => request(path, options),
  post: (path, body, options) =>
    request(path, { method: 'POST', body: JSON.stringify(body), ...options })
};
```

**Key points:**
- Centralized error handling keeps components clean.
- Auth headers are injected once.
- Typed error shape makes UI decisions easier.

</details>

<details>
<summary>Exercise 2: Optimistic Todos</summary>

```jsx
function useAddTodo() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (todo) => api.post('/api/todos', todo),
    onMutate: async (newTodo) => {
      await queryClient.cancelQueries(['todos']);
      const previous = queryClient.getQueryData(['todos']);
      const optimisticId = `temp-${Date.now()}`;

      queryClient.setQueryData(['todos'], (old = []) => [
        ...old,
        { ...newTodo, id: optimisticId, optimistic: true }
      ]);

      return { previous, optimisticId };
    },
    onError: (_err, _todo, context) => {
      if (context?.previous) {
        queryClient.setQueryData(['todos'], context.previous);
      }
    },
    onSuccess: (savedTodo, _todo, context) => {
      queryClient.setQueryData(['todos'], (old = []) =>
        old.map((item) =>
          item.id === context.optimisticId ? savedTodo : item
        )
      );
    },
    onSettled: () => {
      queryClient.invalidateQueries(['todos']);
    }
  });
}
```

**Key points:**
- Cache updates keep the UI responsive.
- Rollbacks avoid permanent bad state.
- Invalidations ensure true server data.

</details>

<details>
<summary>Exercise 3: Infinite Scroll Pagination</summary>

```jsx
function useProducts() {
  return useInfiniteQuery({
    queryKey: ['products'],
    queryFn: ({ pageParam = 1 }) =>
      fetch(`/api/products?cursor=${pageParam}`).then((r) => r.json()),
    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined
  });
}

function ProductFeed() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useProducts();
  const items = data?.pages.flatMap((page) => page.items) ?? [];
  const sentinelRef = useRef(null);

  useEffect(() => {
    if (!hasNextPage || !sentinelRef.current) return;
    const observer = new IntersectionObserver((entries) => {
      if (entries[0].isIntersecting) fetchNextPage();
    });
    observer.observe(sentinelRef.current);
    return () => observer.disconnect();
  }, [fetchNextPage, hasNextPage]);

  return (
    <>
      <ProductList items={items} />
      <div ref={sentinelRef}>
        {hasNextPage && (
          <button onClick={() => fetchNextPage()} disabled={isFetchingNextPage}>
            {isFetchingNextPage ? 'Loading...' : 'Load more'}
          </button>
        )}
      </div>
    </>
  );
}
```

**Key points:**
- Infinite queries manage page state for you.
- Flatten pages into a single list for rendering.
- UI stays responsive while fetching more data.

</details>

---

## What You Learned

This module covered:

- **API layers**: Centralized request and error handling
- **Caching**: Reusing data efficiently with TanStack Query
- **Optimistic updates**: Fast UI with safe rollbacks
- **GraphQL integration**: Consistent data access patterns
- **Loading states**: UX-first feedback for async work

**Key takeaway**: A clean data layer makes UI predictable and resilient.

---

## Real-World Application

This week at work, you might use these concepts to:

- Refactor direct fetch calls into a shared API client
- Add caching and retries to slow endpoints
- Ship optimistic updates for a like or vote feature
- Implement infinite scrolling for search results
- Build consistent error banners across pages

---

## Further Reading

- [TanStack Query - Optimistic Updates](https://tanstack.com/query/latest/docs/react/guides/optimistic-updates)
- [Apollo Client](https://www.apollographql.com/docs/react/)

---

**Navigation**: [← Previous Module](./04-state-tool-selection.md) | [Next Module →](./06-error-handling-patterns.md)
