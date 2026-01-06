# Next.js Advanced

> **Last reviewed**: 2026-01-06


## Week Fifteen: The Server-Client Split

The team adopted Next.js, but performance and developer experience are still uneven. Sarah asks you to define clear patterns for Server Components, streaming, and caching. Marcus warns that misuse of client components is driving up bundle size. This week is about mastering advanced Next.js features so you can ship fast pages without sacrificing flexibility.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Server components** | Prep kitchen - data work before the table |
| **Streaming** | A buffet line - serve what is ready first |
| **Middleware** | A checkpoint - modify requests before they enter |
| **Edge runtime** | A roadside stand - compute close to users |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Mid-Level Module 12 (Next.js Fundamentals) - Familiarity with Next.js routing, data fetching, and deployment basics.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Use Server Components effectively
- [ ] Implement streaming with Suspense
- [ ] Configure middleware and the Edge Runtime
- [ ] Design advanced caching strategies
- [ ] Apply Server Actions safely
- [ ] Reduce client bundle size with server-first patterns

---

## Time Estimate

- **Reading**: 70-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice advanced Next.js patterns over 6-8 weeks

---

## Chapter 1: Server Components

You need a mental model for what runs on the server vs the client.

Sarah says, "Know where the code runs before you ship it."

### Mental Model

```
┌─────────────────────────────────────────────┐
│                   Server                     │
│  ┌────────────────────────────────────────┐ │
│  │         Server Components               │ │
│  │  - Run on server only                   │ │
│  │  - Can access DB, filesystem            │ │
│  │  - No hooks, no event handlers          │ │
│  │  - Stream to client as HTML             │ │
│  └────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│                   Client                     │
│  ┌────────────────────────────────────────┐ │
│  │         Client Components               │ │
│  │  - 'use client' directive               │ │
│  │  - useState, useEffect                  │ │
│  │  - Event handlers                       │ │
│  │  - Browser APIs                         │ │
│  └────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

### Composition Pattern

```tsx
// Server Component (default)
async function ProductPage({ params }) {
  const product = await db.products.findUnique({ id: params.id });

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>

      {/* Client component for interactivity */}
      <AddToCartButton productId={product.id} price={product.price} />

      {/* Server component for data */}
      <Suspense fallback={<ReviewsSkeleton />}>
        <Reviews productId={product.id} />
      </Suspense>
    </div>
  );
}

// Client Component
'use client';

function AddToCartButton({ productId, price }) {
  const [isAdding, setIsAdding] = useState(false);

  const handleClick = async () => {
    setIsAdding(true);
    await addToCart(productId);
    setIsAdding(false);
  };

  return (
    <button onClick={handleClick} disabled={isAdding}>
      {isAdding ? 'Adding...' : `Add to Cart - $${price}`}
    </button>
  );
}
```

### Data Fetching Patterns

```tsx
// Parallel data fetching
async function Dashboard() {
  // These fetch in parallel
  const [user, stats, notifications] = await Promise.all([
    getUser(),
    getStats(),
    getNotifications(),
  ]);

  return (
    <div>
      <UserHeader user={user} />
      <Stats data={stats} />
      <Notifications items={notifications} />
    </div>
  );
}

// Sequential (when needed)
async function OrderDetails({ orderId }) {
  const order = await getOrder(orderId);
  // Need order to get user
  const user = await getUser(order.userId);

  return <OrderView order={order} user={user} />;
}
```

## Chapter 2: Streaming and Suspense

Streaming keeps the UI responsive while data loads.

Marcus says, "Perceived speed is real speed."

### Progressive Loading

```tsx
export default async function Page() {
  return (
    <main>
      {/* Renders immediately */}
      <Header />

      {/* Streams when ready */}
      <Suspense fallback={<ProductGridSkeleton />}>
        <ProductGrid />
      </Suspense>

      {/* Independent streaming */}
      <Suspense fallback={<RecommendationsSkeleton />}>
        <Recommendations />
      </Suspense>

      <Footer />
    </main>
  );
}
```

### Loading UI

```tsx
// app/products/loading.tsx
export default function Loading() {
  return (
    <div className="grid grid-cols-3 gap-4">
      {Array.from({ length: 9 }).map((_, i) => (
        <div key={i} className="animate-pulse">
          <div className="bg-gray-200 h-48 rounded" />
          <div className="bg-gray-200 h-4 mt-2 rounded w-3/4" />
        </div>
      ))}
    </div>
  );
}
```

## Chapter 3: Middleware

Middleware enforces rules before the request reaches your app.

Sarah says, "Put guardrails at the edge, not deep in the tree."

```tsx
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Auth check
  const token = request.cookies.get('auth-token');
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Geolocation-based routing
  const country = request.geo?.country || 'US';
  if (request.nextUrl.pathname === '/') {
    return NextResponse.rewrite(new URL(`/${country}`, request.url));
  }

  // Add headers
  const response = NextResponse.next();
  response.headers.set('x-request-id', crypto.randomUUID());

  return response;
}

export const config = {
  matcher: ['/', '/dashboard/:path*', '/api/:path*'],
};
```

## Chapter 4: Edge Runtime

Move computation closer to the user for lower latency.

Marcus says, "Distance shows up as latency."

```tsx
// app/api/location/route.ts
export const runtime = 'edge';

export async function GET(request: Request) {
  const { geo } = request;

  return Response.json({
    country: geo?.country,
    city: geo?.city,
    region: geo?.region,
  });
}
```

### When to Use Edge

| Use Case | Runtime |
|----------|---------|
| Auth/redirect logic | Edge |
| Geolocation | Edge |
| A/B testing | Edge |
| Database queries | Node.js |
| Heavy computation | Node.js |
| File system access | Node.js |

## Chapter 5: Advanced Caching

Caching strategy is the difference between fast and stale.

Sarah says, "Caching is part of the product contract."

### Request Memoization

```tsx
// Same request is deduplicated within a render
async function Page() {
  // These become ONE request
  const user = await getUser(userId);
  const posts = await getUserPosts(userId); // also calls getUser internally
}

// Opt out with cache: 'no-store'
const data = await fetch(url, { cache: 'no-store' });
```

### Data Cache

```tsx
// Cache for 1 hour
const data = await fetch(url, { next: { revalidate: 3600 } });

// Cache indefinitely (default)
const data = await fetch(url);

// Never cache
const data = await fetch(url, { cache: 'no-store' });
```

### Cache Tags

```tsx
// Tag cached data
async function getProduct(id: string) {
  const res = await fetch(`/api/products/${id}`, {
    next: { tags: [`product-${id}`] },
  });
  return res.json();
}

// Revalidate by tag
import { revalidateTag } from 'next/cache';

export async function updateProduct(id: string, data: ProductData) {
  await db.products.update({ id, data });
  revalidateTag(`product-${id}`);
}
```

### Full Route Cache

```tsx
// Force dynamic rendering
export const dynamic = 'force-dynamic';

// Or force static
export const dynamic = 'force-static';

// Revalidate entire route
export const revalidate = 60; // seconds
```

## Chapter 6: Server Actions

Server Actions simplify mutations without custom endpoints.

Marcus says, "Actions simplify flows, but they still need guardrails."

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function createPost(formData: FormData) {
  const title = formData.get('title');
  const content = formData.get('content');

  await db.posts.create({ title, content });

  revalidatePath('/posts');
}

// Usage in Server Component
export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" required />
      <textarea name="content" required />
      <button type="submit">Create</button>
    </form>
  );
}

// Usage in Client Component
'use client';

import { createPost } from './actions';
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? 'Creating...' : 'Create'}</button>;
}
```

---

## Common Mistakes

1. **Overusing client components** - It bloats bundles and loses server benefits.
2. **Missing Suspense boundaries** - Streaming has no effect without them.
3. **Caching without invalidation** - Stale data hurts trust.
4. **Edge runtime limitations** - Some APIs are not available at the edge.


Example:
```tsx
// ❌ Fetch in a client effect
useEffect(() => {
  fetch('/api/account');
}, []);

// ✅ Fetch in a Server Component
export default async function Page() {
  const data = await fetch('/api/account');
  return <Account data={data} />;
}
```

## Practice Exercises

1. Build a dashboard with streaming Suspense boundaries
2. Implement auth middleware with role-based routing
3. Design a caching strategy for an e-commerce product page

### Solutions

<details>
<summary>Exercise 1: Streaming Dashboard</summary>

```tsx
import { Suspense } from 'react';

export default function Dashboard() {
  return (
    <>
      <Summary />
      <Suspense fallback={<ChartsSkeleton />}>
        <Charts />
      </Suspense>
      <Suspense fallback={<ActivitySkeleton />}>
        <RecentActivity />
      </Suspense>
    </>
  );
}
```

**Key points:**
- Each section streams independently.
- Fast content appears first.
- Slow data does not block the entire page.

</details>

<details>
<summary>Exercise 2: Auth Middleware</summary>

```ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  const role = request.cookies.get('role')?.value;
  const url = new URL(request.url);

  if (url.pathname.startsWith('/admin') && role !== 'admin') {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/admin/:path*']
};
```

**Key points:**
- Middleware runs before routing.
- Role checks block unauthorized access.
- Navigation keeps users on a safe path.

</details>

<details>
<summary>Exercise 3: Product Caching</summary>

```tsx
export const revalidate = 300;

export async function ProductPage({ params }) {
  const product = await fetch(`https://api.example.com/products/${params.id}`, {
    next: { tags: [`product-${params.id}`], revalidate: 300 }
  }).then((r) => r.json());

  return <ProductDetails product={product} />;
}

// Later in a server action or webhook:
// revalidateTag(`product-${id}`);
```

**Key points:**
- ISR keeps data fresh on a schedule.
- Cache tags allow targeted invalidation.
- Server rendering keeps bundles small.

</details>

---

## What You Learned

This module covered:

- **Server components**: Server-first rendering with smaller bundles
- **Streaming**: Faster perceived performance with Suspense
- **Middleware/Edge**: Request-level control close to users
- **Caching**: Predictable freshness strategies
- **Server actions**: Mutations without extra endpoints

**Key takeaway**: Advanced Next.js patterns let you ship faster pages with fewer client costs.

---

## Real-World Application

This week at work, you might use these concepts to:

- Move data fetching into Server Components
- Add streaming to a slow dashboard
- Build role-based middleware for admin routes
- Implement cache tags for product pages
- Reduce client bundle size by removing unnecessary hooks

---

## Further Reading

- [Next.js App Router Documentation](https://nextjs.org/docs/app)
- [Server Components RFC](https://github.com/reactjs/rfcs/pull/188)

---

**Navigation**: [← Previous Module](./14-mentoring-juniors.md) | [Next Module →](./16-nextjs-app-router-deep-dive.md)
