# API Design and Integration

Build robust data layers that handle real-world complexity.

## Learning Objectives

By the end of this module, you will:
- Structure API calls and error handling
- Implement caching and optimistic updates
- Work with REST and GraphQL APIs
- Handle loading and error states gracefully

## API Layer Architecture

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

## Using with TanStack Query

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

## Optimistic Updates

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

## Error Handling Patterns

```jsx
function ErrorMessage({ error }) {
  if (error.status === 401) {
    return <Redirect to="/login" />;
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

## GraphQL with Apollo

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

## Loading States

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

## Practice Exercises

1. Build an API layer with error handling and auth
2. Implement optimistic updates for a todo list
3. Add pagination with infinite scroll

## Further Reading

- [TanStack Query - Optimistic Updates](https://tanstack.com/query/latest/docs/react/guides/optimistic-updates)
- [Apollo Client](https://www.apollographql.com/docs/react/)
