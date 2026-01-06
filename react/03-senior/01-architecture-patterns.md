# Architecture Patterns

> **Last reviewed**: 2026-01-06


## Week One: The Architecture Review

The product is growing fast and the codebase is starting to drift. Sarah asks you to lead an architecture review before the next feature push. Marcus pulls up a diagram with three competing structures and says, "You need a map that scales with the team." This week is about organizing code so features are easy to find, dependencies stay predictable, and decisions are documented instead of tribal knowledge.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Feature slices** | Neighborhoods - group related work together |
| **Module boundaries** | Fences - control what can pass through |
| **Monorepos** | A campus - shared infrastructure, separate buildings |
| **Dependency injection** | Power outlets - swap implementations safely |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 12 (Next.js Fundamentals) - Comfort with project structure and production concerns.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Apply feature-sliced design to organize a growing codebase
- [ ] Define module boundaries with clear public APIs
- [ ] Structure monorepos with shared libraries and tooling
- [ ] Use dependency injection to decouple implementations
- [ ] Document architectural decisions with explicit trade-offs
- [ ] Evaluate architecture options based on team constraints

---

## Time Estimate

- **Reading**: 70-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice architecture decisions over 6-8 weeks

---

## Chapter 1: Feature-Sliced Design

You need a structure that scales without burying features across folders.

Organize code by features, not technical layers.

```
src/
├── app/                    # App initialization, providers
├── pages/                  # Route entry points
├── widgets/                # Composite UI blocks
├── features/               # User interactions
├── entities/               # Business entities
├── shared/                 # Reusable utilities
│   ├── ui/                 # UI kit
│   ├── lib/                # Utilities
│   ├── api/                # API client
│   └── config/             # Config
```

### Layers

```
┌─────────────────────────────────────┐
│            app (routing, providers) │
├─────────────────────────────────────┤
│            pages (route components) │
├─────────────────────────────────────┤
│         widgets (complex UI blocks) │
├─────────────────────────────────────┤
│  features (user actions & flows)    │
├─────────────────────────────────────┤
│      entities (business objects)    │
├─────────────────────────────────────┤
│        shared (reusable code)       │
└─────────────────────────────────────┘

Rule: Higher layers can import from lower layers, never the reverse.
```

### Example Structure

```
features/
├── auth/
│   ├── ui/
│   │   ├── LoginForm.tsx
│   │   └── LogoutButton.tsx
│   ├── model/
│   │   ├── useAuth.ts
│   │   └── authSlice.ts
│   ├── api/
│   │   └── authApi.ts
│   └── index.ts            # Public API

entities/
├── user/
│   ├── ui/
│   │   └── UserCard.tsx
│   ├── model/
│   │   └── types.ts
│   └── index.ts
```

## Chapter 2: Module Boundaries

Clear boundaries prevent accidental coupling between features.

### Public API Pattern

```tsx
// features/cart/index.ts
// Only export what other modules should use
export { AddToCartButton } from './ui/AddToCartButton';
export { useCart } from './model/useCart';
export type { CartItem } from './model/types';

// Internal implementation stays private
// features/cart/model/cartSlice.ts - not exported
```

### Dependency Rules

```tsx
// ✅ Good: Feature uses entity
// features/checkout/ui/OrderSummary.tsx
import { ProductCard } from '@/entities/product';
import { useCart } from '@/features/cart';

// ❌ Bad: Entity uses feature (violates layer order)
// entities/product/ui/ProductCard.tsx
import { AddToCartButton } from '@/features/cart'; // Wrong!

// ✅ Fix: Pass as prop
interface ProductCardProps {
  product: Product;
  actions?: React.ReactNode;
}

// Usage in feature layer
<ProductCard
  product={product}
  actions={<AddToCartButton productId={product.id} />}
/>
```

## Chapter 3: Monorepo Structure

Some teams need shared packages without losing ownership.

```
packages/
├── web/                    # Main web app
│   ├── src/
│   └── package.json
├── mobile/                 # React Native app
│   ├── src/
│   └── package.json
├── shared/
│   ├── ui/                 # Shared components
│   │   ├── src/
│   │   └── package.json
│   ├── utils/              # Shared utilities
│   └── types/              # Shared TypeScript types
└── package.json            # Root workspace
```

### Turborepo Configuration

```json
// turbo.json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "test": {
      "dependsOn": ["build"]
    },
    "lint": {}
  }
}
```

### Shared Package Setup

```json
// packages/shared/ui/package.json
{
  "name": "@myorg/ui",
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "dependencies": {
    "react": "^18.2.0"
  }
}

// packages/web/package.json
{
  "dependencies": {
    "@myorg/ui": "workspace:*"
  }
}
```

## Chapter 4: Dependency Injection

When dependencies change, you should not need to rewrite every consumer.

Decouple components from implementations.

```tsx
// Define interface
interface AnalyticsService {
  track(event: string, data?: Record<string, unknown>): void;
}

// Create context
const AnalyticsContext = createContext<AnalyticsService | null>(null);

// Provider with concrete implementation
function AnalyticsProvider({ children }: { children: React.ReactNode }) {
  const analytics: AnalyticsService = {
    track(event, data) {
      // Production: send to analytics service
      if (process.env.NODE_ENV === 'production') {
        analyticsClient.track(event, data);
      } else {
        console.log('Track:', event, data);
      }
    },
  };

  return (
    <AnalyticsContext.Provider value={analytics}>
      {children}
    </AnalyticsContext.Provider>
  );
}

// Hook for consuming
function useAnalytics() {
  const context = useContext(AnalyticsContext);
  if (!context) throw new Error('Missing AnalyticsProvider');
  return context;
}

// Usage in component - doesn't know implementation
function CheckoutButton() {
  const analytics = useAnalytics();

  const handleClick = () => {
    analytics.track('checkout_started');
  };

  return <button onClick={handleClick}>Checkout</button>;
}
```

## Chapter 5: Trade-off Analysis

Architecture choices need a paper trail so the team can revisit them later.

When making architectural decisions, document trade-offs:

```markdown
### Decision: State Management Approach

### Context
Need to manage complex checkout flow state.

### Options Considered

1. **React Context + useReducer**
   - Pros: Simple, no dependencies
   - Cons: Re-render performance, debugging tools

2. **Redux Toolkit**
   - Pros: DevTools, middleware, proven at scale
   - Cons: Boilerplate, learning curve

3. **XState**
   - Pros: Explicit states, impossible states impossible
   - Cons: Team unfamiliar, overhead for simple cases

### Decision
Redux Toolkit - team familiarity, DevTools for debugging.

### Consequences
- Need to train new team members
- Keep slices focused to manage complexity
```

---

## Common Mistakes

1. **Organizing by file type only** - Features get scattered and hard to reason about.
2. **Leaky module boundaries** - Internal details become public API by accident.
3. **Monorepo without ownership** - Shared code becomes nobody's responsibility.
4. **Undocumented decisions** - Teams repeat the same debates every quarter.

## Practice Exercises

1. Refactor a flat project structure to feature-sliced design
2. Set up a monorepo with shared UI library
3. Write an ADR for a technical decision

### Solutions

<details>
<summary>Exercise 1: Feature-Sliced Refactor</summary>

```text
src/
├── app/
├── pages/
├── widgets/
├── features/
│   ├── checkout/
│   │   ├── ui/
│   │   ├── model/
│   │   ├── api/
│   │   ├── lib/
│   │   └── index.ts
│   └── search/
│       ├── ui/
│       ├── model/
│       └── index.ts
├── entities/
│   └── user/
│       ├── ui/
│       ├── model/
│       ├── api/
│       └── index.ts
└── shared/
    ├── ui/
    ├── lib/
    ├── config/
    └── api/
```

**Key points:**
- Group by feature, not by layer.
- Expose only `index.ts` as the public API.
- Keep shared utilities and config in `shared/`.
- Features depend on entities, not the reverse.

</details>

<details>
<summary>Exercise 2: Monorepo Layout</summary>

```text
apps/
├── web/
└── admin/
packages/
├── ui/
├── design-tokens/
├── api-client/
├── eslint-config/
├── tsconfig/
└── utils/
turbo.json
pnpm-workspace.yaml
```

**Key points:**
- Apps consume packages through stable interfaces.
- Shared packages keep tooling and styling consistent.
- Tooling packages prevent config drift.
- Ownership is explicit per package.

</details>

<details>
<summary>Exercise 3: ADR Template</summary>

```markdown
# Decision: Client State Management

## Context
We need shared state for checkout and user session across multiple apps.

## Decision Drivers
- Debugging support
- Team familiarity
- Performance impact
- Migration cost

## Options Considered
1. Context + useReducer
2. Redux Toolkit
3. XState

## Decision
Redux Toolkit for DevTools support and team familiarity.

## Consequences
- Training required for new hires
- Keep slices small to avoid complexity
```

**Key points:**
- Document options and trade-offs.
- Capture consequences for future teams.
- Keep it short and actionable.

</details>

---

## What You Learned

This module covered:

- **Feature slicing**: Organizing by user-facing capability
- **Boundaries**: Public APIs that prevent coupling
- **Monorepos**: Shared tooling with clear ownership
- **Dependency injection**: Swappable implementations
- **Trade-offs**: Decisions that stay visible over time

**Key takeaway**: Architecture is a map for people, not just code.

---

## Real-World Application

This week at work, you might use these concepts to:

- Propose a new folder structure for a growing app
- Define public APIs for feature modules
- Split shared UI into a package
- Document a tooling decision with an ADR
- Review dependency boundaries during a refactor

---

## Further Reading

- [Feature-Sliced Design](https://feature-sliced.design/)
- [Turborepo Handbook](https://turbo.build/repo/docs)

---

**Navigation**: [Next Module →](./02-architecture-decision-records.md)