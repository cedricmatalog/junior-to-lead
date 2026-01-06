# Modern React APIs

> **Last reviewed**: 2026-01-06


## Week Eighteen: The Hook You Haven't Met Yet

A library update breaks your subscription logic. The old pattern still "works," but React warns it's unsafe under concurrent rendering.

"React added new APIs for a reason," Sarah says. "They're guardrails for modern rendering."

Marcus adds, "If you skip these, you end up fighting the framework."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **External store** | A radio station - you tune in, but you don't control it |
| **Insertion effect** | A backstage pass - run before the paint happens |
| **Action hooks** | A submission desk - server work with UI feedback |
| **Optimistic UI** | A reserved seat - show it before the ticket is confirmed |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Senior Module 16 (Next.js App Router Deep Dive) - Familiarity with server/client boundaries and streaming.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Use `useSyncExternalStore` for safe subscriptions
- [ ] Apply `useInsertionEffect` in styling contexts
- [ ] Use `useActionState` and `useFormStatus` for server actions
- [ ] Apply `useOptimistic` for optimistic UI updates
- [ ] Decide when these APIs are necessary
- [ ] Avoid common concurrency pitfalls

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Apply modern APIs over 4-6 weeks

---

## Chapter 1: useSyncExternalStore

Subscriptions need to be concurrent-safe.

"If you're listening to something outside React, use the right hook," Sarah says.

```tsx
import { useSyncExternalStore } from 'react';

function useAuthStore() {
  return useSyncExternalStore(
    authStore.subscribe,
    authStore.getSnapshot
  );
}
```

"If you subscribe to something outside React, use the official hook," Sarah says.

---

## Chapter 2: useInsertionEffect

Some styling libraries need to inject CSS before layout.

"Only use it for styles," Marcus says. "Nothing else."

```tsx
import { useInsertionEffect } from 'react';

useInsertionEffect(() => {
  insertStyles();
}, []);
```

"It's specialized. Use it only for style injection," Marcus says.

---

## Chapter 3: useActionState and useFormStatus

Server actions need UI feedback.

"Keep server work close to the UI," Sarah says.

```tsx
'use client';
import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';

function SaveButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Saving...' : 'Save'}
    </button>
  );
}

function SaveProfile() {
  const [state, formAction] = useActionState(saveProfile, { error: null });
  return (
    <form action={formAction}>
      <SaveButton />
      {state.error && <p>{state.error}</p>}
    </form>
  );
}
```

"Actions keep server work close to the UI," Sarah says.

---

## Chapter 4: useOptimistic

Show updates before confirmation.

"Optimism needs a rollback plan," Marcus says.

```tsx
'use client';
import { useOptimistic } from 'react';

const [optimisticItems, addOptimistic] = useOptimistic(
  items,
  (state, newItem) => [...state, newItem]
);
```

"Optimistic UI should still handle rollback when the server rejects."

---

## Chapter 5: When to Use These

Don't reach for advanced hooks unless you need them.

"If useState works, keep it," Sarah says.

"If `useState` works, keep it. Use modern APIs when concurrency or server actions require it."

---

## Chapter 6: Migration Notes

Upgrade in small steps:

"Small migrations beat big rewrites," Marcus says.

- Replace custom subscriptions with `useSyncExternalStore`
- Keep `useInsertionEffect` limited to CSS-in-JS
- Move critical forms to `useActionState`

---

## Common Mistakes

1. **Using insertion effect for data** - It blocks paint.
2. **Skipping snapshots** - External stores must provide stable snapshots.
3. **Optimistic without rollback** - Users see false success.
4. **Using action hooks everywhere** - They’re for server actions, not local UI.


Example:
```tsx
// ❌ useInsertionEffect for data fetch
useInsertionEffect(() => fetch('/api/data'));

// ✅ useEffect for data, insertion only for styles
useEffect(() => fetch('/api/data'));
```

---

## Practice Exercises

1. Convert a custom store subscription to `useSyncExternalStore`
2. Add `useActionState` to a settings form
3. Implement optimistic UI for a like button

### Solutions

<details>
<summary>Exercise 1: External Store</summary>

```tsx
const state = useSyncExternalStore(store.subscribe, store.getSnapshot);
```

**Key points:**
- Provide stable snapshots.
- Use official hook for concurrency.
- Avoid custom subscription hacks.

</details>

<details>
<summary>Exercise 2: Action State</summary>

```tsx
const [state, action] = useActionState(updateSettings, { error: null });
```

**Key points:**
- Keep server action near form.
- Render state feedback.
- Handle errors explicitly.

</details>

<details>
<summary>Exercise 3: Optimistic Like</summary>

```tsx
const [optimisticLikes, addOptimisticLike] = useOptimistic(
  likes,
  (state, delta) => state + delta
);
```

**Key points:**
- Update UI immediately.
- Roll back on error.
- Keep source of truth on server.

</details>

---

## What You Learned

This module covered:

- **useSyncExternalStore**: Safe external subscriptions
- **useInsertionEffect**: Style injection before paint
- **Action hooks**: Server actions with `useActionState` and `useFormStatus`
- **useOptimistic**: Fast UI with rollback

**Key takeaway**: Modern React APIs exist to keep concurrency safe and user feedback crisp.

---

## Real-World Application

This week at work, you might use these concepts to:

- Replace a custom store subscription
- Add server actions to a profile form
- Implement optimistic updates for reactions
- Fix a CSS-in-JS timing issue
- Reduce concurrency bugs in complex state

---

## Further Reading

- [React useSyncExternalStore](https://react.dev/reference/react/useSyncExternalStore)
- [React useInsertionEffect](https://react.dev/reference/react/useInsertionEffect)
- [React useOptimistic](https://react.dev/reference/react/useOptimistic)

---

**Navigation**: [← Previous Module](./17-framework-comparison.md) | [Next Module →](./19-web-workers-off-main-thread.md)
