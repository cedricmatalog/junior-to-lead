# State at Scale

Manage complex state effectively as applications grow.

## Learning Objectives

By the end of this module, you will:
- Choose appropriate state management for your needs
- Use Redux Toolkit effectively
- Implement lightweight alternatives (Zustand, Jotai)
- Separate server state from client state

## State Categories

| Type | Examples | Solution |
|------|----------|----------|
| Local UI | Form inputs, toggles, modals | useState |
| Shared UI | Theme, sidebar state | Context or Zustand |
| Server | User data, products, orders | TanStack Query |
| URL | Filters, pagination, tabs | URL params |

## Redux Toolkit

Modern Redux with less boilerplate.

```jsx
// store/slices/cartSlice.js
import { createSlice } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    total: 0,
  },
  reducers: {
    addItem: (state, action) => {
      // RTK uses Immer - can "mutate" directly
      state.items.push(action.payload);
      state.total += action.payload.price;
    },
    removeItem: (state, action) => {
      const index = state.items.findIndex(i => i.id === action.payload);
      if (index !== -1) {
        state.total -= state.items[index].price;
        state.items.splice(index, 1);
      }
    },
    clearCart: (state) => {
      state.items = [];
      state.total = 0;
    },
  },
});

export const { addItem, removeItem, clearCart } = cartSlice.actions;
export default cartSlice.reducer;

// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import cartReducer from './slices/cartSlice';

export const store = configureStore({
  reducer: {
    cart: cartReducer,
  },
});

// Usage in component
import { useSelector, useDispatch } from 'react-redux';
import { addItem } from './store/slices/cartSlice';

function ProductCard({ product }) {
  const dispatch = useDispatch();
  const cartItems = useSelector(state => state.cart.items);

  return (
    <button onClick={() => dispatch(addItem(product))}>
      Add to Cart
    </button>
  );
}
```

## Zustand

Minimal state management with hooks.

```jsx
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

const useCartStore = create(
  persist(
    (set, get) => ({
      items: [],
      total: 0,

      addItem: (product) => set((state) => ({
        items: [...state.items, product],
        total: state.total + product.price,
      })),

      removeItem: (productId) => set((state) => {
        const item = state.items.find(i => i.id === productId);
        return {
          items: state.items.filter(i => i.id !== productId),
          total: state.total - (item?.price || 0),
        };
      }),

      clearCart: () => set({ items: [], total: 0 }),

      // Computed values
      get itemCount() {
        return get().items.length;
      },
    }),
    {
      name: 'cart-storage', // localStorage key
    }
  )
);

// Usage - no Provider needed!
function CartButton() {
  const itemCount = useCartStore(state => state.items.length);
  return <button>Cart ({itemCount})</button>;
}

function AddToCartButton({ product }) {
  const addItem = useCartStore(state => state.addItem);
  return <button onClick={() => addItem(product)}>Add</button>;
}
```

## Jotai

Atomic state management - bottom-up approach.

```jsx
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';

// Create atoms
const countAtom = atom(0);
const doubleCountAtom = atom((get) => get(countAtom) * 2);

// Async atoms
const userAtom = atom(async () => {
  const response = await fetch('/api/user');
  return response.json();
});

// Writable derived atom
const temperatureAtom = atom(
  (get) => get(celsiusAtom),
  (get, set, newValue) => {
    set(celsiusAtom, (newValue - 32) * 5 / 9);
  }
);

// Usage
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const doubleCount = useAtomValue(doubleCountAtom);

  return (
    <div>
      <p>Count: {count}, Double: {doubleCount}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
}
```

## Server State with TanStack Query

Separate concerns: let a library handle caching, refetching, and synchronization.

```jsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function UserProfile({ userId }) {
  const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(r => r.json()),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });

  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  return <div>{user.name}</div>;
}

function UpdateUserButton({ userId }) {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newData) =>
      fetch(`/api/users/${userId}`, {
        method: 'PATCH',
        body: JSON.stringify(newData),
      }),
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['user', userId] });
    },
  });

  return (
    <button
      onClick={() => mutation.mutate({ name: 'New Name' })}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? 'Saving...' : 'Update'}
    </button>
  );
}
```

## State Machines (XState)

For complex state logic with defined transitions.

```jsx
import { createMachine } from 'xstate';
import { useMachine } from '@xstate/react';

const fetchMachine = createMachine({
  id: 'fetch',
  initial: 'idle',
  states: {
    idle: {
      on: { FETCH: 'loading' }
    },
    loading: {
      invoke: {
        src: 'fetchData',
        onDone: { target: 'success', actions: 'setData' },
        onError: { target: 'error', actions: 'setError' }
      }
    },
    success: {
      on: { FETCH: 'loading' }
    },
    error: {
      on: { RETRY: 'loading' }
    }
  }
});

function DataFetcher() {
  const [state, send] = useMachine(fetchMachine);

  if (state.matches('loading')) return <Spinner />;
  if (state.matches('error')) return <button onClick={() => send('RETRY')}>Retry</button>;
  return <Data data={state.context.data} />;
}
```

## Choosing the Right Tool

| Scenario | Recommendation |
|----------|----------------|
| Small app, simple state | useState + Context |
| Need DevTools, middleware | Redux Toolkit |
| Want minimal boilerplate | Zustand |
| Atomic, bottom-up approach | Jotai |
| Server data caching | TanStack Query |
| Complex state transitions | XState |

## Practice Exercises

1. Migrate a Context-based app to Zustand
2. Implement optimistic updates with TanStack Query
3. Build a multi-step form wizard with XState

## Further Reading

- [Redux Toolkit](https://redux-toolkit.js.org/)
- [Zustand](https://github.com/pmndrs/zustand)
- [TanStack Query](https://tanstack.com/query/latest)
