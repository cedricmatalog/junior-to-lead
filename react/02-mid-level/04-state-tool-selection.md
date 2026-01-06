# State Tool Selection

> **Last reviewed**: 2026-01-06


## Week Four: Too Many Stores

The app now has Context, a small Zustand store, and a few reducers. Every new feature adds another state container.

"You don't need more tools," Sarah says. "You need boundaries."

Marcus adds, "Picking the right state tool is less about popularity and more about tradeoffs."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Local state** | A sticky note on your desk - private and quick |
| **Shared state** | A team whiteboard - visible and coordinated |
| **Server state** | A delivery tracker - remote, cached, and refreshed |
| **Derived state** | A calculator - computed from source data |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 03 (State at Scale) - Familiarity with Redux Toolkit, Zustand, and server state concepts.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Distinguish local, shared, and server state clearly
- [ ] Decide when state should live in URL, store, or component
- [ ] Choose a state tool using explicit tradeoffs
- [ ] Avoid duplicated or derived state bugs
- [ ] Design boundaries that prevent state sprawl
- [ ] Plan incremental migrations between tools

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Apply tradeoffs over 3-5 weeks

---

## Chapter 1: The State Inventory

Sarah asks you to list every piece of state in the feature.

"If you don't inventory it, you can't organize it."

```text
Local: modal open, input values, hover state
Shared: cart, auth, feature flags
Server: products, orders, pricing
URL: filters, pagination, search
```

"Each bucket wants a different tool," Marcus says.

---

## Chapter 2: Local vs Shared

Local state should be fast and private.

```jsx
function QuantityPicker() {
  const [count, setCount] = useState(1);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

Shared state should be explicit and stable.

```jsx
const useCartStore = create(set => ({
  items: [],
  addItem: item => set(state => ({ items: [...state.items, item] }))
}));
```

"If more than one part of the app depends on it, it stops being local," Sarah says.

---

## Chapter 3: Server State Is Its Own Category

Fetching data is different from UI state.

```jsx
const { data: products } = useQuery({
  queryKey: ['products'],
  queryFn: fetchProducts
});
```

"Server state needs caching, retries, and invalidation. That's not a local store's job."

---

## Chapter 4: Avoid Derived State Traps

Duplicating derived state creates bugs.

```jsx
const total = items.reduce((sum, item) => sum + item.price, 0);
```

"Keep a single source of truth, compute the rest."

---

## Chapter 5: The Decision Matrix

Marcus sketches a quick matrix:

```text
Is it remote data? -> Server state tool
Is it shared UI state? -> Store/context
Is it local to one component? -> useState
Does it need URL shareability? -> query params
```

"Choose the simplest tool that satisfies the requirements."

---

## Chapter 6: Migrations Without Pain

You don't have to rewrite everything.

"Wrap old state, migrate feature-by-feature, and keep the UI stable," Sarah says.

---

## Common Mistakes

1. **Using a store for everything** - Local state is faster and simpler.
2. **Duplicating server data** - Cache it once, derive UI state locally.
3. **Storing derived values** - Compute instead of syncing.
4. **Mixing responsibilities** - Separate UI state from server state.


Example:
```jsx
// ❌ Duplicate derived state
const [total, setTotal] = useState(0);
useEffect(() => setTotal(items.reduce((s, i) => s + i.price, 0)), [items]);

// ✅ Derive on render
const total = items.reduce((s, i) => s + i.price, 0);
```

---

## Practice Exercises

1. Classify a feature's state into local/shared/server/URL buckets
2. Refactor a component to remove derived state
3. Propose a migration plan from Context to a store

### Solutions

<details>
<summary>Exercise 1: State Classification</summary>

```text
Local: modal open, search input
Shared: cart count, auth user
Server: products list
URL: sort, filter, page
```

**Key points:**
- Keep categories explicit.
- URL state should be shareable.
- Server data should be cached.

</details>

<details>
<summary>Exercise 2: Remove Derived State</summary>

```jsx
// Before: derived state stored
const [total, setTotal] = useState(0);
useEffect(() => setTotal(items.reduce((s, i) => s + i.price, 0)), [items]);

// After: derived in render
const total = items.reduce((s, i) => s + i.price, 0);
```

**Key points:**
- Derived values should be computed.
- Avoid effects for pure calculations.
- Simpler and less error-prone.

</details>

<details>
<summary>Exercise 3: Migration Plan</summary>

```text
1) Wrap Context reads with a new store selector
2) Move one feature slice at a time
3) Keep old Context API stable until complete
4) Delete old provider after migration
```

**Key points:**
- Migrate gradually.
- Keep API stable during the transition.
- Remove legacy once unused.

</details>

---

## What You Learned

This module covered:

- **State categories**: Local, shared, server, URL
- **Boundaries**: Preventing state sprawl
- **Derived state**: Compute, don't duplicate
- **Decision matrix**: Pick tools with clear tradeoffs
- **Migrations**: Reduce risk with incremental moves

**Key takeaway**: The right state tool is the one that matches the state category and scope.

---

## Real-World Application

This week at work, you might use these concepts to:

- Move filter state into the URL for shareable links
- Replace a shared Context with a focused store
- Remove duplicated totals or counts
- Isolate server data into a query cache
- Draft a state map for a complex feature

---

## Further Reading

- [TanStack Query: Server State](https://tanstack.com/query/latest/docs/framework/react/overview)
- [Redux Style Guide](https://redux.js.org/style-guide/style-guide)
- [React Router: URL State](https://reactrouter.com/en/main/start/concepts)

---

**Navigation**: [← Previous Module](./03-state-at-scale.md) | [Next Module →](./05-api-design-integration.md)
