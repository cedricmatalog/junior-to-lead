# Framework Comparison

> **Last reviewed**: 2026-01-06


## Week 31: The Platform Decision

Leadership wants to launch a new product and asks for a framework recommendation. Sarah expects a clear comparison, not opinions. Marcus reminds you that the best choice depends on constraints: team skill, hosting, data needs, and long-term maintenance. This week is about evaluating frameworks systematically, planning migrations, and staying current without chasing every trend.

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Framework choice** | A vehicle - built for different terrains |
| **Decision matrix** | A scorecard - compare with criteria |
| **Migration** | A move - plan steps and packing |
| **Ecosystem health** | A forecast - watch for long-term risk |

Keep these in mind. They'll click as we build.

---

## Prerequisites

Module 30 (Next.js Advanced) - Familiarity with modern React frameworks.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Compare major React frameworks objectively
- [ ] Match frameworks to project requirements
- [ ] Use decision matrices to justify choices
- [ ] Plan migrations with reduced risk
- [ ] Evaluate ecosystem health signals
- [ ] Document framework decisions clearly

---

## Time Estimate

- **Reading**: 70-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice framework evaluation over 6-8 weeks

---

## Chapter 1: Framework Overview

You need a baseline comparison before ranking options.

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

## Chapter 2: Decision Matrix

A matrix keeps decisions grounded in evidence, not preference.

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

## Chapter 3: When to Choose What

Different products require different trade-offs.

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

## Chapter 4: Migration Strategies

Migrations succeed when they are staged and measurable.

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

## Chapter 5: Staying Current

Staying current is about patterns, not chasing every release.

### Following Development

- GitHub release notes
- Framework Discord/community
- Conference talks
- Core team Twitter/blogs

### Evaluating New Features

```markdown
## Feature Evaluation Template

Templates make decisions repeatable across teams.

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

---

## Common Mistakes

1. **Choosing by hype** - Popular does not mean right for your constraints.
2. **Ignoring ops impact** - Hosting and deployment complexity matter.
3. **Underestimating migrations** - Move in phases, not big bangs.
4. **Skipping decision records** - Teams need to know why choices were made.

## Practice Exercises

1. Build the same feature in Next.js and Remix, compare
2. Create a migration plan for a legacy CRA app
3. Evaluate a new framework feature for your team

### Solutions

<details>
<summary>Exercise 1: Feature Comparison</summary>

```text
Next.js: SSR/ISR, App Router, strong Vercel integration
Remix: Nested routes, loader/action model, great form handling
Astro: Island architecture, content-heavy sites, multi-framework
```

**Key points:**
- Compare developer experience and runtime trade-offs.
- Measure bundle size and load time.
- Document operational impact.

</details>

<details>
<summary>Exercise 2: Migration Plan</summary>

```markdown
Phase 1: Extract shared UI into @myorg/ui
Phase 2: Route new features to Next.js behind feature flags
Phase 3: Migrate high-traffic routes with dual-run monitoring
Phase 4: Cut over remaining routes and decommission CRA
```

**Key points:**
- Phase migrations to reduce risk.
- Use feature flags to control rollouts.
- Track metrics during each phase.

</details>

<details>
<summary>Exercise 3: Feature Evaluation</summary>

```markdown
Feature: React Server Components
Benefits: Smaller bundles, server data access
Costs: New mental model, tooling changes, testing updates
Decision: Pilot on one route with success criteria
Success: 20% bundle reduction, no regression in LCP
```

**Key points:**
- Match benefits to real problems.
- Limit scope for early trials.
- Document the decision.

</details>

---

## What You Learned

This module covered:

- **Framework comparison**: Strengths and trade-offs
- **Decision matrices**: Objective selection criteria
- **Migration planning**: Phased, low-risk moves
- **Ecosystem health**: Signals that matter long term
- **Evaluation templates**: Repeatable decision making

**Key takeaway**: Framework choices should be justified with evidence, not preference.

---

## Real-World Application

This week at work, you might use these concepts to:

- Build a decision matrix for a new project
- Compare Next.js and Remix for a product launch
- Draft a phased migration plan
- Present a framework recommendation to leadership
- Document framework choices in an ADR

---

## Further Reading

- [Next.js Documentation](https://nextjs.org/docs)
- [Remix Documentation](https://remix.run/docs)
- [Astro Documentation](https://docs.astro.build)

---

**Navigation**: [<- Previous Module](./10-nextjs-advanced.md)
