# React Router Data APIs

> **Last reviewed**: 2026-01-06


## Week Sixteen: The Loading That Never Ended

A new settings page takes ten seconds to load. The UI spins, the URL changes, and users click refresh. The data fetch lives in a `useEffect`, and you can't tell if the route is actually ready.

"Routing isn't just about screens anymore," Sarah says. "It's about data." 

Marcus nods. "Data routers let the route own the fetch, not the component."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Loader** | A kitchen prep station - data ready before serving |
| **Action** | A checkout counter - submit data and get a response |
| **Defer** | A split order - serve what’s ready now, the rest later |
| **Route errors** | A safety net - catch failures at the route boundary |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Mid-Level Module 05 (API Design and Integration) - Comfort with async requests and error handling.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Use loaders to fetch data before routes render
- [ ] Handle mutations with actions and form submissions
- [ ] Stream partial data with `defer` and `Await`
- [ ] Add route-level error boundaries for failures
- [ ] Use fetchers for background updates
- [ ] Revalidate data intentionally after mutations

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Apply data router patterns over 3-5 weeks

---

## Chapter 1: Loaders Make Routes Responsible

Sarah points at a slow page.

"Stop fetching inside the component. Let the route do it."

```jsx
import { createBrowserRouter } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/settings',
    loader: async () => fetch('/api/settings').then(r => r.json()),
    element: <Settings />,
  },
]);
```

"Now the route owns data timing and errors."

---

## Chapter 2: useLoaderData for the Result

The component becomes simpler.

"Let the loader carry the data," Sarah says.

```jsx
import { useLoaderData } from 'react-router-dom';

function Settings() {
  const data = useLoaderData();
  return <SettingsForm initial={data} />;
}
```

"You render with data already in hand," Marcus says.

---

## Chapter 3: Actions for Mutations

Forms can submit directly to routes.

"Mutations belong next to the route," Marcus says.

```jsx
import { Form } from 'react-router-dom';

// Route
{
  path: '/settings',
  action: async ({ request }) => {
    const form = await request.formData();
    return saveSettings(Object.fromEntries(form));
  },
  element: <Settings />,
}

// Component
<Form method="post">
  <input name="email" />
  <button type="submit">Save</button>
</Form>
```

"The route handles the mutation. The component stays clean."

---

## Chapter 4: Defer and Stream

Some data is slow. Serve what you can now.

"Split slow data so the page can breathe," Sarah says.

```jsx
import { defer, Await } from 'react-router-dom';

loader: () => defer({
  summary: fetch('/api/summary').then(r => r.json()),
  history: fetch('/api/history').then(r => r.json()),
});
```

```jsx
<Await resolve={summary}>{data => <Summary data={data} />}</Await>
```

"Defer turns one slow response into two fast ones," Sarah says.

---

## Chapter 5: Error Boundaries at the Route

Handle failures where they happen.

"Contain errors at the route boundary," Marcus says.

```jsx
{
  path: '/settings',
  loader,
  element: <Settings />,
  errorElement: <RouteError />,
}
```

"A route-level error page prevents broken UI from cascading."

---

## Chapter 6: Fetchers and Revalidation

Update in the background without navigation.

"Background updates should not hijack the URL," Sarah says.

```jsx
import { useFetcher } from 'react-router-dom';

const fetcher = useFetcher();
fetcher.submit({ id }, { method: 'post', action: '/api/pin' });
```

"Use fetchers for side effects that shouldn't change the URL."

---

## Common Mistakes

1. **Fetching in components again** - You lose route-level control.
2. **No errorElement** - Errors fall through to blank screens.
3. **Mutating without revalidation** - UI stays stale.
4. **Overusing defer** - Only defer genuinely slow data.


Example:
```jsx
// ❌ Fetch inside component
useEffect(() => fetch('/api/settings'), []);

// ✅ Fetch in loader
export async function loader() {
  return fetch('/api/settings');
}
```

---

## Practice Exercises

1. Convert a settings page to use a loader
2. Add a form action for a profile update
3. Defer a slow analytics panel

### Solutions

<details>
<summary>Exercise 1: Loader Conversion</summary>

```jsx
export async function loader() {
  return fetch('/api/settings').then(r => r.json());
}

function Settings() {
  const data = useLoaderData();
  return <SettingsForm initial={data} />;
}
```

**Key points:**
- Keep data fetching in the route.
- Render with prepared data.
- Simplify component logic.

</details>

<details>
<summary>Exercise 2: Action Submission</summary>

```jsx
export async function action({ request }) {
  const form = await request.formData();
  return updateProfile(Object.fromEntries(form));
}
```

**Key points:**
- Use `Form` for submissions.
- Keep mutations near the route.
- Revalidate after success.

</details>

<details>
<summary>Exercise 3: Defer Panel</summary>

```jsx
export async function loader() {
  return defer({
    metrics: fetch('/api/metrics').then(r => r.json()),
  });
}
```

**Key points:**
- Defer slow data only.
- Wrap with `Await`.
- Keep UI responsive.

</details>

---

## What You Learned

This module covered:

- **Loaders**: Fetch data at the route boundary
- **Actions**: Handle mutations with route submissions
- **Defer**: Stream slow data
- **Errors**: Catch failures with `errorElement`
- **Fetchers**: Background updates without navigation

**Key takeaway**: When routes own data, UI stays predictable and fast.

---

## Real-World Application

This week at work, you might use these concepts to:

- Move a slow `useEffect` fetch into a loader
- Add route-based form submissions for profile edits
- Stream a large analytics panel with `defer`
- Add route-level error handling for 404s
- Revalidate data after a mutation

---

## Further Reading

- [React Router Data APIs](https://reactrouter.com/en/main/routers/picking-a-router)
- [Loaders and Actions](https://reactrouter.com/en/main/route/loader)
- [Defer and Await](https://reactrouter.com/en/main/utils/defer)

---

**Navigation**: [← Previous Module](./15-deployment-and-ci.md) | [Next Module →](./17-pwa-and-offline-first.md)
