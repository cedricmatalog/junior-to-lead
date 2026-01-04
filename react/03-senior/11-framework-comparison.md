# Framework Comparison

Choose the right tool for each project.

## Learning Objectives

By the end of this module, you will:
- Understand key differences between major frameworks
- Match frameworks to project requirements
- Plan migrations between frameworks
- Stay informed on ecosystem evolution

## Framework Overview

### Next.js

**Best for**: Full-stack apps, complex data requirements, SEO-critical sites

```tsx
// Hybrid rendering, built-in API routes
export default async function Page() {
  const data = await fetch('https://api.example.com/data');
  return <div>{data}</div>;
}
```

**Strengths**:
- Flexible rendering (SSR, SSG, ISR)
- Built-in API routes
- Excellent TypeScript support
- Vercel integration
- Large ecosystem

**Weaknesses**:
- Can be complex to configure
- Vendor-influenced roadmap
- Bundle size without care

### Remix

**Best for**: Web standards, progressive enhancement, nested routing

```tsx
// Loader runs on server
export async function loader({ params }) {
  return json(await getUser(params.id));
}

// Action handles mutations
export async function action({ request }) {
  const formData = await request.formData();
  return createUser(Object.fromEntries(formData));
}

export default function User() {
  const user = useLoaderData();
  return <Form method="post">...</Form>;
}
```

**Strengths**:
- Web standards focus
- Excellent error handling
- Progressive enhancement
- Nested routes with parallel loading
- Works without JavaScript

**Weaknesses**:
- Smaller ecosystem
- Different mental model
- Fewer deployment options

### Astro

**Best for**: Content sites, documentation, blogs, marketing sites

```astro
---
// Component script (server-side)
const posts = await getCollection('blog');
---

<!-- Template (generates HTML) -->
<html>
  <body>
    {posts.map(post => <PostCard post={post} />)}

    <!-- Island: Only this hydrates -->
    <Newsletter client:visible />
  </body>
</html>
```

**Strengths**:
- Near-zero JavaScript by default
- "Islands" architecture
- Use any UI framework
- Excellent for content
- Built-in content collections

**Weaknesses**:
- Not for highly interactive apps
- Smaller community
- Different paradigm

### Vite + React

**Best for**: SPAs, internal tools, prototypes

```tsx
// Pure client-side React
import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('root')).render(<App />);
```

**Strengths**:
- Simple setup
- Fast development
- Full control
- No framework constraints

**Weaknesses**:
- No SSR built-in
- SEO challenges
- Build your own solutions

## Decision Matrix

| Requirement | Next.js | Remix | Astro | Vite+React |
|------------|---------|-------|-------|------------|
| SEO critical | ✅ | ✅ | ✅ | ⚠️ |
| Highly interactive | ✅ | ✅ | ⚠️ | ✅ |
| Content heavy | ✅ | ✅ | ✅✅ | ⚠️ |
| API routes needed | ✅ | ✅ | ⚠️ | ❌ |
| Progressive enhancement | ⚠️ | ✅✅ | ✅ | ❌ |
| Minimal JavaScript | ⚠️ | ⚠️ | ✅✅ | ❌ |
| Team familiarity | ✅✅ | ✅ | ⚠️ | ✅✅ |
| Deployment flexibility | ✅ | ✅ | ✅✅ | ✅✅ |

## When to Choose What

### Choose Next.js When

- Building full-stack applications
- Need flexible rendering strategies
- Team knows React well
- Using Vercel for deployment
- Complex data requirements

### Choose Remix When

- Progressive enhancement is important
- Complex nested routing needed
- Want web-standard patterns
- Forms-heavy applications
- Need great error boundaries

### Choose Astro When

- Building content-focused sites
- Want minimal JavaScript
- Documentation or blogs
- Marketing/landing pages
- Mix of static and dynamic

### Choose Vite + React When

- Building internal tools
- Full SPA is acceptable
- Prototype or MVP
- Want maximum flexibility
- Don't need SSR

## Migration Strategies

### Incremental Migration

```tsx
// Next.js app wrapping legacy React
// pages/legacy/[...path].tsx
export default function LegacyWrapper() {
  return (
    <div id="legacy-root">
      {/* Mount legacy app here */}
    </div>
  );
}

// Migrate route by route
// pages/new-feature.tsx (fully Next.js)
```

### Strangler Pattern

```
Phase 1: New features in new framework
Phase 2: Migrate high-traffic pages
Phase 3: Migrate remaining pages
Phase 4: Decommission old app

Timeline: Feature flags control which version serves each route
```

### Shared Component Library

```
┌─────────────┐     ┌─────────────┐
│  Old App    │     │  New App    │
│  (CRA)      │     │  (Next.js)  │
└──────┬──────┘     └──────┬──────┘
       │                   │
       └───────┬───────────┘
               │
       ┌───────┴───────┐
       │   @myorg/ui   │
       │  (Shared)     │
       └───────────────┘

Extract components to shared package first,
then migrate apps independently.
```

## Staying Current

### Following Development

- GitHub release notes
- Framework Discord/community
- Conference talks
- Core team Twitter/blogs

### Evaluating New Features

```markdown
## Feature Evaluation Template

**Feature**: React Server Components

**Understand**:
- What problem does it solve?
- How does it work?

**Evaluate**:
- Does it solve a problem we have?
- What's the adoption cost?
- Is it stable enough for production?

**Plan**:
- When would we adopt?
- What's the migration path?
- Who needs to learn what?
```

## Practice Exercises

1. Build the same feature in Next.js and Remix, compare
2. Create a migration plan for a legacy CRA app
3. Evaluate a new framework feature for your team

## Further Reading

- [Next.js Documentation](https://nextjs.org/docs)
- [Remix Documentation](https://remix.run/docs)
- [Astro Documentation](https://docs.astro.build)
