# Next.js Advanced

Master advanced Next.js patterns for production applications.

## Learning Objectives

By the end of this module, you will:
- Use Server Components effectively
- Implement streaming and Suspense
- Configure middleware and edge runtime
- Design advanced caching strategies

## Server Components

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

## Streaming and Suspense

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

## Middleware

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

## Edge Runtime

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

## Advanced Caching

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

## Server Actions

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

## Practice Exercises

1. Build a dashboard with streaming Suspense boundaries
2. Implement auth middleware with role-based routing
3. Design a caching strategy for an e-commerce product page

## Further Reading

- [Next.js App Router Documentation](https://nextjs.org/docs/app)
- [Server Components RFC](https://github.com/reactjs/rfcs/pull/188)
