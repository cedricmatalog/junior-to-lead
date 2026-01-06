# Next.js Fundamentals

> **Last reviewed**: 2026-01-06


## Week 20: The Framework Shift

The team wants server-rendered pages, faster initial loads, and a path to production deployments. Sarah suggests moving the newest features to Next.js, while Marcus wants to avoid surprises with the new App Router. This week is about understanding the framework's mental model, how data flows on the server and the client, and how to ship a production-ready app without getting lost in the routing and deployment details.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Routing** | A filing cabinet - path maps to a folder |
| **Data fetching** | A kitchen order - prepare before serving |
| **Server components** | A prep kitchen - work happens before the table |
| **Deployment** | A launch checklist - steps before takeoff |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 09 (Code Quality) - Comfort with project structure and production readiness.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Compare Pages Router and App Router tradeoffs
- [ ] Implement SSR, SSG, and ISR data fetching
- [ ] Build dynamic routes and layouts
- [ ] Create API routes with proper handling
- [ ] Add loading and error states in the App Router
- [ ] Configure metadata and deployment settings

---

## Time Estimate

- **Reading**: 70-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice Next.js fundamentals over 4-6 weeks

---

## Chapter 1: Pages Router vs App Router

You need to choose the right routing model before you start migrating features.

| Feature | Pages Router | App Router |
|---------|-------------|------------|
| Directory | `pages/` | `app/` |
| Routing | File-based | Folder-based |
| Data fetching | getServerSideProps, getStaticProps | Server Components, fetch |
| Layouts | Manual `_app.js` | Nested layouts |
| Loading states | Manual | `loading.js` |
| Status | Stable, established | Newer, more features |

## Chapter 2: App Router Basics

Marcus wants a clean routing structure that the whole team can follow.

```
app/
├── layout.tsx          # Root layout
├── page.tsx            # Home page (/)
├── about/
│   └── page.tsx        # About page (/about)
├── blog/
│   ├── page.tsx        # Blog list (/blog)
│   └── [slug]/
│       └── page.tsx    # Blog post (/blog/my-post)
└── api/
    └── users/
        └── route.ts    # API route (/api/users)
```

### Root Layout

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <nav>
          <Link href="/">Home</Link>
          <Link href="/about">About</Link>
        </nav>
        <main>{children}</main>
        <footer>© 2024</footer>
      </body>
    </html>
  );
}
```

### Page Component

```tsx
// app/page.tsx
export default function HomePage() {
  return (
    <div>
      <h1>Welcome</h1>
      <p>This is the home page.</p>
    </div>
  );
}
```

## Chapter 3: Data Fetching

You need to decide where data lives and when it loads.

### Server Components (Default)

```tsx
// app/users/page.tsx
// This runs on the server - no "use client"
async function getUsers() {
  const res = await fetch('https://api.example.com/users', {
    cache: 'no-store', // SSR - fetch every request
    // cache: 'force-cache', // SSG - cache indefinitely
    // next: { revalidate: 60 }, // ISR - revalidate every 60s
  });
  return res.json();
}

export default async function UsersPage() {
  const users = await getUsers();

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### Client Components

```tsx
// app/components/Counter.tsx
'use client'; // Marks this as a Client Component

import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}
```

### When to Use Each

| Server Components | Client Components |
|-------------------|-------------------|
| Fetch data | Interactive UI (onClick, onChange) |
| Access backend directly | useState, useEffect |
| Keep sensitive info server-side | Browser APIs |
| Large dependencies | Real-time updates |

## Chapter 4: Dynamic Routes

Marketing needs blog posts and product pages that scale with content.

```tsx
// app/blog/[slug]/page.tsx
interface Props {
  params: { slug: string };
}

export default async function BlogPost({ params }: Props) {
  const post = await getPost(params.slug);

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}

// Generate static paths at build time
export async function generateStaticParams() {
  const posts = await getPosts();
  return posts.map(post => ({ slug: post.slug }));
}
```

## Chapter 5: API Routes

Some backend functionality can live alongside the UI for speed and simplicity.

```tsx
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const users = await db.users.findMany();
  return NextResponse.json(users);
}

export async function POST(request: NextRequest) {
  const body = await request.json();
  const user = await db.users.create({ data: body });
  return NextResponse.json(user, { status: 201 });
}

// app/api/users/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const user = await db.users.findUnique({
    where: { id: params.id },
  });

  if (!user) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }

  return NextResponse.json(user);
}
```

## Chapter 6: Loading and Error States

Fast navigation still needs clear feedback when data is loading or broken.

```tsx
// app/users/loading.tsx
export default function Loading() {
  return <div>Loading users...</div>;
}

// app/users/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

## Chapter 7: Shipping and Configuration

Before launch, you need consistent metadata, environment configuration, and deployment steps.

### Metadata

```tsx
// app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    template: '%s | My Site',
    default: 'My Site',
  },
  description: 'Welcome to my site',
};

// app/about/page.tsx
export const metadata: Metadata = {
  title: 'About', // Becomes "About | My Site"
};
```

### Environment Variables

```bash
# .env.local
DATABASE_URL=postgresql://...
NEXT_PUBLIC_API_URL=https://api.example.com
```

```tsx
// Server-side only
const dbUrl = process.env.DATABASE_URL;

// Available in browser (prefixed with NEXT_PUBLIC_)
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

### Deployment

```bash
# Build for production
npm run build

# Start production server
npm start

# Or deploy to Vercel
npx vercel
```

---

## Common Mistakes

1. **Mixing router paradigms** - Don't combine Pages and App Router patterns in the same feature.
2. **Over-fetching on the client** - Prefer server data fetching when possible.
3. **Missing loading states** - App Router routes need explicit `loading.tsx`.
4. **Leaking secrets** - Only expose variables prefixed with `NEXT_PUBLIC_`.


Example:
```tsx
// ❌ Fetch on the client for first paint data
useEffect(() => {
  fetch('/api/products');
}, []);

// ✅ Fetch on the server with getServerSideProps
export async function getServerSideProps() {
  return { props: { /* data */ } };
}
```

## Practice Exercises

1. Create a blog with dynamic routes and static generation
2. Build an API route with CRUD operations
3. Implement loading states and error boundaries

### Solutions

<details>
<summary>Exercise 1: Blog with Dynamic Routes</summary>

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetch('https://example.com/api/posts').then((r) => r.json());
  return posts.map((post) => ({ slug: post.slug }));
}

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await fetch(`https://example.com/api/posts/${params.slug}`, {
    next: { revalidate: 300 }
  }).then((r) => r.json());

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
    </article>
  );
}
```

**Key points:**
- `generateStaticParams` drives SSG for dynamic routes.
- Data is fetched on the server for fast initial paint.
- Each slug maps to a static page.

</details>

<details>
<summary>Exercise 2: CRUD API Route</summary>

```ts
// app/api/items/route.ts
import { NextResponse } from 'next/server';

let items = [{ id: 1, name: 'Sample' }];

export async function GET() {
  return NextResponse.json(items);
}

export async function POST(request: Request) {
  const body = await request.json();
  if (!body?.name) {
    return NextResponse.json({ message: 'Name is required' }, { status: 400 });
  }
  const item = { id: Date.now(), ...body };
  items.push(item);
  return NextResponse.json(item, { status: 201 });
}

export async function DELETE(request: Request) {
  const { id } = await request.json();
  items = items.filter((item) => item.id !== id);
  return NextResponse.json({ ok: true });
}
```

**Key points:**
- Route handlers map to HTTP verbs.
- Response helpers keep output consistent.
- Simple in-memory data works for demos.

</details>

<details>
<summary>Exercise 3: Loading and Error States</summary>

```tsx
// app/products/loading.tsx
export default function Loading() {
  return <p>Loading products...</p>;
}

// app/products/error.tsx
'use client';

export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div role="alert">
      <p>Something went wrong.</p>
      <pre>{error.message}</pre>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

**Key points:**
- `loading.tsx` shows while data loads.
- `error.tsx` catches route errors.
- `reset` allows a retry without full reload.

</details>

---

## What You Learned

This module covered:

- **Routing models**: Choosing Pages vs App Router
- **Data fetching**: SSR, SSG, and ISR tradeoffs
- **Dynamic routes**: Scaling pages with content
- **API routes**: Server logic colocated with UI
- **Shipping**: Metadata, env variables, and deployment

**Key takeaway**: Next.js gives you server and client tools, but you still need a plan.

---

## Real-World Application

This week at work, you might use these concepts to:

- Migrate a static page to the App Router
- Add server-rendered product pages for SEO
- Build a small API route for internal tooling
- Implement route-level loading and error UI
- Set environment variables for staging and production

---

## Further Reading

- [Next.js Documentation](https://nextjs.org/docs)
- [App Router Migration Guide](https://nextjs.org/docs/app/building-your-application/upgrading/app-router-migration)

---

**Navigation**: [← Previous Module](./09-code-quality.md)
