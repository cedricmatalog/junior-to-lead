# Next.js App Router Deep Dive

> **Last reviewed**: 2026-01-06


## Week Sixteen: The Server Component Surprise

The team upgraded to the App Router and suddenly a client-only library broke. A page that used to fetch on the client now runs on the server by default.

"The App Router changes the mental model," Sarah says. "You need to know where your code runs."

Marcus adds, "Once you understand boundaries, the new model is powerful."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Server Component** | A kitchen - cook data before serving |
| **Client Component** | The dining room - interactive and immediate |
| **Caching** | A pantry - reuse prepared ingredients |
| **Route Handler** | A service window - lightweight APIs next to UI |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 15 (Next.js Advanced) - Familiarity with App Router concepts and streaming.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Distinguish server and client component boundaries
- [ ] Control caching with Next.js fetch options
- [ ] Use route handlers for lightweight APIs
- [ ] Build loading and error boundaries per route
- [ ] Decide between edge and node runtimes
- [ ] Avoid common App Router migration pitfalls

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 4-5 hours
- **Mastery**: Practice App Router patterns over 4-6 weeks

---

## Chapter 1: Server vs Client Components

By default, components are server components.

```tsx
// app/products/page.tsx (server component)
export default async function Page() {
  const products = await fetch('https://api.example.com/products').then(r => r.json());
  return <ProductGrid products={products} />;
}
```

"Add `'use client'` only when you need browser APIs or interactivity," Sarah says.

---

## Chapter 2: Caching and Revalidation

Caching is now built into `fetch`.

```tsx
await fetch('https://api.example.com/products', {
  next: { revalidate: 60 }
});
```

"Think of revalidation as a timer for freshness," Marcus says.

---

## Chapter 3: Route Handlers

Use `route.ts` for small APIs.

```ts
// app/api/health/route.ts
export async function GET() {
  return Response.json({ status: 'ok' });
}
```

"Keep handlers small and close to the UI they serve."

---

## Chapter 4: Loading and Error Boundaries

App Router gives per-route `loading.tsx` and `error.tsx`.

```tsx
// app/products/loading.tsx
export default function Loading() {
  return <ProductGridSkeleton />;
}
```

"Granular boundaries make the app feel fast even when data is slow."

---

## Chapter 5: Edge vs Node

Edge is fast but limited. Node is flexible but slower.

"Choose edge for latency-sensitive reads. Choose node for complex dependencies."

---

## Chapter 6: Migration Pitfalls

Common mistakes:

- Using client-only libraries in server components
- Fetching in both server and client
- Forgetting `'use client'` for hooks

---

## Common Mistakes

1. **Making everything client** - You lose server performance wins.
2. **Ignoring cache settings** - Defaults might be too stale or too fresh.
3. **Mixing server and client concerns** - Keep boundaries clean.
4. **Using heavy logic in edge runtime** - Edge has limits.


Example:
```tsx
// ❌ Client component for data fetching
'use client';
useEffect(() => fetch('/api/products'), []);

// ✅ Server component fetch
export default async function Page() {
  const data = await fetch('/api/products').then(r => r.json());
  return <Products data={data} />;
}
```

---

## Practice Exercises

1. Convert a client-fetched page to a server component
2. Add a route handler for a health check
3. Add `loading.tsx` and `error.tsx` to a route

### Solutions

<details>
<summary>Exercise 1: Server Component Fetch</summary>

```tsx
export default async function Page() {
  const data = await fetch('https://api.example.com/items').then(r => r.json());
  return <Items data={data} />;
}
```

**Key points:**
- Server components can fetch directly.
- Avoid duplicating fetches in client code.
- Keep client components for interactivity only.

</details>

<details>
<summary>Exercise 2: Route Handler</summary>

```ts
export async function GET() {
  return Response.json({ status: 'ok' });
}
```

**Key points:**
- Keep handlers small.
- Use for lightweight APIs.
- Pair with UI routes as needed.

</details>

<details>
<summary>Exercise 3: Loading and Error</summary>

```tsx
// app/items/loading.tsx
export default function Loading() {
  return <ItemsSkeleton />;
}
```

```tsx
// app/items/error.tsx
'use client';
export default function Error({ error }) {
  return <p>Something went wrong.</p>;
}
```

**Key points:**
- `loading.tsx` is server by default.
- `error.tsx` must be client.
- Keep boundaries per route.

</details>

---

## What You Learned

This module covered:

- **Boundaries**: Server vs client components
- **Caching**: Revalidation controls freshness
- **Route handlers**: Lightweight APIs
- **Loading/error**: Per-route boundaries
- **Runtime**: Edge vs node tradeoffs

**Key takeaway**: The App Router is powerful when you treat server and client as different environments.

---

## Real-World Application

This week at work, you might use these concepts to:

- Move data fetching to the server
- Add cached API calls with revalidation
- Create route handlers for small APIs
- Improve perceived performance with loading boundaries
- Choose the right runtime for latency-sensitive routes

---

## Further Reading

- [Next.js App Router](https://nextjs.org/docs/app)
- [Next.js Caching](https://nextjs.org/docs/app/building-your-application/caching)
- [Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)

---

**Navigation**: [← Previous Module](./15-nextjs-advanced.md) | [Next Module →](./17-framework-comparison.md)
