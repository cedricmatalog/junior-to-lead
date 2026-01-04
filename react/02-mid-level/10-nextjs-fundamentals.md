# Next.js Fundamentals

Build production-ready React applications with Next.js.

## Learning Objectives

By the end of this module, you will:
- Understand Pages Router vs App Router
- Implement SSR, SSG, and ISR
- Create API routes
- Configure for production deployment

## Pages Router vs App Router

| Feature | Pages Router | App Router |
|---------|-------------|------------|
| Directory | `pages/` | `app/` |
| Routing | File-based | Folder-based |
| Data fetching | getServerSideProps, getStaticProps | Server Components, fetch |
| Layouts | Manual `_app.js` | Nested layouts |
| Loading states | Manual | `loading.js` |
| Status | Stable, established | Newer, more features |

## App Router Basics

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

## Data Fetching

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

## Dynamic Routes

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

## API Routes

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

## Loading and Error States

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

## Metadata

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

## Environment Variables

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

## Deployment

```bash
# Build for production
npm run build

# Start production server
npm start

# Or deploy to Vercel
npx vercel
```

## Practice Exercises

1. Create a blog with dynamic routes and static generation
2. Build an API route with CRUD operations
3. Implement loading states and error boundaries

## Further Reading

- [Next.js Documentation](https://nextjs.org/docs)
- [App Router Migration Guide](https://nextjs.org/docs/app/building-your-application/upgrading/app-router-migration)
