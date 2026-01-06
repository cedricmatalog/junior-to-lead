# State at Scale

> **Last reviewed**: 2026-01-06


## Week Three: The Data Explosion

The product team wants personalized feeds, cross-device carts, and real-time updates. Sarah warns you that "state sprawl" is creeping in: some data lives in local components, some in global stores, and some on the server. Marcus adds that bugs are showing up because nobody knows where the source of truth lives. This week is about choosing the right state tool for each problem and keeping the data model sane as the app grows.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Client state** | A whiteboard - quick changes you control |
| **Server state** | A kitchen ticket - cached, synced with the backend |
| **State machines** | Traffic lights - strict, predictable transitions |
| **Atom stores** | Sticky notes - small, independent pieces |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 02 (Performance Optimization) - Comfort with hooks, effects, and performance tradeoffs.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Categorize state and choose the right storage approach
- [ ] Build scalable stores with Redux Toolkit
- [ ] Implement lightweight stores with Zustand or Jotai
- [ ] Separate server state using TanStack Query
- [ ] Model complex flows with state machines
- [ ] Evaluate tradeoffs between simplicity and tooling

---

## Time Estimate

- **Reading**: 60-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice these patterns over 4-6 weeks

---

## Chapter 1: State Categories

You need a shared language for state before you pick any tooling.

| Type | Examples | Solution |
|------|----------|----------|
| Local UI | Form inputs, toggles, modals | useState |
| Shared UI | Theme, sidebar state | Context or Zustand |
| Server | User data, products, orders | TanStack Query |
| URL | Filters, pagination, tabs | URL params |

## Chapter 2: Redux Toolkit

The cart needs global visibility, DevTools, and predictable updates. Redux Toolkit handles the heavy lifting.

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

## Chapter 3: Zustand

Some UI state is too small for Redux. You need a lighter store for quick wins.

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

## Chapter 4: Jotai

For modular features, atom-based state keeps dependencies clean.

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

// Writable derived atom (Fahrenheit backed by Celsius)
const celsiusAtom = atom(0);
const fahrenheitAtom = atom(
  (get) => get(celsiusAtom) * 9 / 5 + 32,
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

## Chapter 5: Server State with TanStack Query

Product data is slow and stale. You need caching, retries, and background updates.

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

## Chapter 6: State Machines (XState)

The checkout flow keeps breaking because transitions are implicit. A state machine makes them explicit.

For complex state logic with defined transitions.

```jsx
import { createMachine, assign } from 'xstate';
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

## Chapter 7: Choosing the Right Tool

Sarah expects you to justify why each slice of state lives where it does.

| Scenario | Recommendation |
|----------|----------------|
| Small app, simple state | useState + Context |
| Need DevTools, middleware | Redux Toolkit |
| Want minimal boilerplate | Zustand |
| Atomic, bottom-up approach | Jotai |
| Server data caching | TanStack Query |
| Complex state transitions | XState |

---

## Common Mistakes

1. **Globalizing everything** - Not all state belongs in a store; keep local state local.
2. **Mixing server and client state** - Cache server data separately from UI state to avoid bugs.
3. **Overusing Context** - Context is not a replacement for state management tools.
4. **Skipping state modeling** - Complex flows benefit from explicit machines or reducers.


Example:
```jsx
// ❌ Treat server data as local UI state
const [projects, setProjects] = useState([]);

// ✅ Use a server-state cache
const { data: projects } = useQuery(['projects'], fetchProjects);
```

## Practice Exercises

1. Migrate a Context-based app to Zustand
2. Implement optimistic updates with TanStack Query
3. Build a multi-step form wizard with XState

### Solutions

<details>
<summary>Exercise 1: Context to Zustand</summary>

```jsx
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

const useCartStore = create(
  persist(
    (set, get) => ({
      items: [],
      addItem: (item) =>
        set((state) => {
          const existing = state.items.find((i) => i.id === item.id);
          if (existing) {
            return {
              items: state.items.map((i) =>
                i.id === item.id ? { ...i, qty: i.qty + 1 } : i
              )
            };
          }
          return { items: [...state.items, { ...item, qty: 1 }] };
        }),
      removeItem: (id) =>
        set((state) => ({ items: state.items.filter((i) => i.id !== id) })),
      clear: () => set({ items: [] }),
      getTotal: () =>
        get().items.reduce((sum, item) => sum + item.price * item.qty, 0)
    }),
    { name: 'cart-store' }
  )
);

function CartButton({ product }) {
  const addItem = useCartStore((state) => state.addItem);
  return <button onClick={() => addItem(product)}>Add to cart</button>;
}

function CartCount() {
  const count = useCartStore((state) => state.items.length);
  return <span>{count}</span>;
}
```

**Key points:**
- Zustand replaces context provider plumbing.
- Selectors keep components re-rendering only when needed.
- Persistence keeps cart state across refreshes.
- Derived state can be calculated with selectors or helpers.

</details>

<details>
<summary>Exercise 2: Optimistic Updates</summary>

```jsx
function useToggleFavorite() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id) => fetch(`/api/favorites/${id}`, { method: 'POST' }),
    onMutate: async (id) => {
      await queryClient.cancelQueries(['favorites']);
      const previous = queryClient.getQueryData(['favorites']);
      const optimisticId = `optimistic-${id}`;

      queryClient.setQueryData(['favorites'], (old = []) => [
        ...old,
        { id, optimistic: true, optimisticId }
      ]);

      return { previous, optimisticId };
    },
    onError: (_err, _id, context) => {
      if (context?.previous) {
        queryClient.setQueryData(['favorites'], context.previous);
      }
    },
    onSuccess: (_data, id, context) => {
      queryClient.setQueryData(['favorites'], (old = []) =>
        old.map((item) =>
          item.optimisticId === context?.optimisticId
            ? { ...item, id, optimistic: false }
            : item
        )
      );
    },
    onSettled: () => {
      queryClient.invalidateQueries(['favorites']);
    }
  });
}
```

**Key points:**
- Update cache immediately for a fast UI.
- Roll back on error using stored previous data.
- Invalidate to sync with the server.

</details>

<details>
<summary>Exercise 3: XState Form Wizard</summary>

```jsx
import { createMachine } from 'xstate';
import { useMachine } from '@xstate/react';

const formMachine = createMachine({
  id: 'signup',
  initial: 'account',
  context: { data: { email: '', name: '' } },
  states: {
    account: {
      on: {
        UPDATE: { actions: 'updateData' },
        NEXT: { target: 'profile', guard: 'accountValid' }
      }
    },
    profile: {
      on: {
        UPDATE: { actions: 'updateData' },
        NEXT: 'confirm',
        BACK: 'account'
      }
    },
    confirm: { on: { SUBMIT: 'submitting', BACK: 'profile' } },
    submitting: { on: { SUCCESS: 'done', ERROR: 'confirm' } },
    done: {}
  }
}, {
  guards: {
    accountValid: ({ context }) => Boolean(context.data.email)
  },
  actions: {
    updateData: assign({
      data: ({ context, event }) => ({ ...context.data, ...event.data })
    })
  }
});

function SignupWizard() {
  const [state, send] = useMachine(formMachine);

  return (
    <div>
      {state.matches('account') && (
        <AccountStep
          data={state.context.data}
          onChange={(data) => send({ type: 'UPDATE', data })}
          onNext={() => send('NEXT')}
        />
      )}
      {state.matches('profile') && (
        <ProfileStep onBack={() => send('BACK')} onNext={() => send('NEXT')} />
      )}
      {state.matches('confirm') && (
        <ConfirmStep onBack={() => send('BACK')} onSubmit={() => send('SUBMIT')} />
      )}
      {state.matches('done') && <Success />}
    </div>
  );
}
```

**Key points:**
- Transitions are explicit and testable.
- Steps are guarded by states, not boolean flags.
- Easy to add validation transitions later.

</details>

---

## What You Learned

This module covered:

- **State categories**: Matching state type to the right tool
- **Redux Toolkit**: Structured global state with DevTools support
- **Zustand & Jotai**: Lightweight, focused stores
- **Server state**: Caching and syncing with TanStack Query
- **State machines**: Predictable transitions for complex flows

**Key takeaway**: The right state tool depends on the state's job, not the team's habits.

---

## Real-World Application

This week at work, you might use these concepts to:

- Separate cart UI state from server-backed inventory data
- Add optimistic updates to a "like" feature
- Replace prop drilling with a focused store
- Stabilize a multi-step onboarding flow
- Justify state choices during a design review

---

## Further Reading

- [Redux Toolkit](https://redux-toolkit.js.org/)
- [Zustand](https://github.com/pmndrs/zustand)
- [TanStack Query](https://tanstack.com/query/latest)

---

**Navigation**: [← Previous Module](./02-performance-optimization.md) | [Next Module →](./04-state-tool-selection.md)
