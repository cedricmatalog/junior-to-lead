# System Design

> **Last reviewed**: 2026-01-06


## Week Three: The Scale Blueprint

Traffic is growing and the product roadmap is expanding. Sarah asks you to propose a system design that supports multiple teams, faster deployments, and better reliability. Marcus warns that the current build pipeline will not survive another year of growth. This week is about thinking beyond components and designing the full frontend system: layers, deployments, caching, and observability.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Architecture layers** | A city map - each layer has a job |
| **Micro-frontends** | City districts - teams own their blocks |
| **Build systems** | Assembly lines - predictable outputs at scale |
| **CDN strategy** | Highway ramps - deliver fast and local |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 01 (Architecture Patterns) - Familiarity with structure, boundaries, and trade-offs.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Design frontend architecture layers for scale
- [ ] Evaluate when micro-frontends make sense
- [ ] Plan build and deployment strategies
- [ ] Define caching and CDN approaches
- [ ] Set performance budgets and error tracking
- [ ] Justify system-level technology choices

---

## Time Estimate

- **Reading**: 70-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice system design decisions over 6-8 weeks

---

## Chapter 1: Frontend Architecture Layers

You need a shared model of the system before teams can move independently.

```
┌─────────────────────────────────────────────┐
│              Presentation Layer              │
│    (Components, Pages, Styling, Animations) │
├─────────────────────────────────────────────┤
│               State Layer                    │
│  (Local State, Global State, Server Cache)  │
├─────────────────────────────────────────────┤
│              Service Layer                   │
│    (API Clients, Auth, Analytics, i18n)     │
├─────────────────────────────────────────────┤
│            Infrastructure Layer              │
│   (Build, Bundling, CDN, Error Tracking)    │
└─────────────────────────────────────────────┘
```

## Chapter 2: Micro-Frontends

Multiple teams want ownership without stepping on each other.

### When to Consider

- Multiple teams working independently
- Different tech stacks needed
- Independent deployment required
- Large, complex applications

### Approaches

**Module Federation (Webpack 5)**

```js
// host/webpack.config.js
new ModuleFederationPlugin({
  name: 'host',
  remotes: {
    checkout: 'checkout@https://checkout.example.com/remoteEntry.js',
    profile: 'profile@https://profile.example.com/remoteEntry.js',
  },
});

// Usage in host
const Checkout = React.lazy(() => import('checkout/CheckoutPage'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/checkout" element={<Checkout />} />
      </Routes>
    </Suspense>
  );
}
```

**Route-based Splitting**

```
https://example.com/          → Main app
https://example.com/checkout  → Checkout micro-frontend
https://example.com/admin     → Admin micro-frontend
```

### Trade-offs

| Pros | Cons |
|------|------|
| Independent deployments | Increased complexity |
| Team autonomy | Shared state challenges |
| Technology flexibility | Bundle size duplication |
| Fault isolation | Consistent UX harder |

## Chapter 3: Build System Design

Builds are getting slow and fragile. You need a scalable pipeline.

### Modern Build Pipeline

```
Source Code
    │
    ▼
TypeScript Compilation
    │
    ▼
Bundling (Vite/webpack)
    │
    ├── Code Splitting
    ├── Tree Shaking
    ├── Minification
    └── Asset Optimization
    │
    ▼
Static Assets
    │
    ▼
CDN Distribution
```

### Vite Configuration

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom', 'react-router-dom'],
          ui: ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
        },
      },
    },
    sourcemap: true,
  },
  server: {
    proxy: {
      '/api': 'http://localhost:3001',
    },
  },
});
```

## Chapter 4: Deployment Strategies

Releases need to be safer and more frequent.

### Blue-Green Deployment

```
Production Traffic
       │
       ▼
   Load Balancer
    /         \
   ▼           ▼
┌──────┐    ┌──────┐
│ Blue │    │Green │
│(live)│    │(new) │
└──────┘    └──────┘

1. Deploy new version to Green
2. Test Green environment
3. Shift traffic to Green
4. Blue becomes standby
```

### Canary Releases

```
100% → Version A (current)

Deploy Version B:
 95% → Version A
  5% → Version B (canary)

Monitor metrics, gradually increase:
 80% → Version A
 20% → Version B

If healthy:
100% → Version B
```

### Feature Flags

```tsx
import { useFeatureFlag } from './featureFlags';

function Header() {
  const showNewNav = useFeatureFlag('new-navigation');

  return showNewNav ? <NewNavigation /> : <LegacyNavigation />;
}

// Flag configuration
const flags = {
  'new-navigation': {
    enabled: true,
    rolloutPercentage: 25,
    targetUsers: ['beta-testers'],
  },
};
```

## Chapter 5: CDN and Caching Strategy

Speed depends on where content lives and how it is cached.

```
┌─────────────┐     ┌─────────┐     ┌────────┐
│   Browser   │ ──▶ │   CDN   │ ──▶ │ Origin │
└─────────────┘     └─────────┘     └────────┘

Cache-Control Headers:
- index.html:     no-cache (always validate)
- JS/CSS (hashed): max-age=31536000, immutable
- API responses:  varies by endpoint
```

```nginx
# nginx configuration example
location /assets/ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

location /index.html {
    expires -1;
    add_header Cache-Control "no-store, must-revalidate";
}
```

## Chapter 6: Performance Budget

Budgets keep performance from drifting over time.

```json
// performance-budget.json
{
  "bundles": {
    "main": { "maxSize": "200kb" },
    "vendor": { "maxSize": "150kb" }
  },
  "metrics": {
    "firstContentfulPaint": { "max": 1500 },
    "largestContentfulPaint": { "max": 2500 },
    "timeToInteractive": { "max": 3500 },
    "cumulativeLayoutShift": { "max": 0.1 }
  }
}
```

## Chapter 7: Error Tracking Architecture

You need visibility across the entire frontend system.

```
┌─────────────────────────────────────────────┐
│                  Browser                     │
│  ┌─────────────────────────────────────┐    │
│  │         Error Boundary              │    │
│  │  ┌─────────────────────────────┐   │    │
│  │  │    Global Error Handler     │   │    │
│  │  │  (window.onerror)           │   │    │
│  │  └─────────────────────────────┘   │    │
│  └─────────────────────────────────────┘    │
└───────────────────┬─────────────────────────┘
                    │
                    ▼
            ┌──────────────┐
            │    Sentry    │
            │  (Aggregate) │
            └──────────────┘
                    │
                    ▼
          ┌────────────────┐
          │   Alerting     │
          │ (Slack/PD)     │
          └────────────────┘
```

---

## Common Mistakes

1. **Designing in isolation** - System design needs input from infrastructure and backend teams.
2. **Ignoring deployment constraints** - Architecture must match release processes.
3. **Skipping budgets** - Without limits, performance regresses over time.
4. **Unowned observability** - Alerts without owners become noise.


Example:
```ts
// ❌ UI reaches into the data layer directly
import { db } from '../db';

// ✅ UI depends on an interface boundary
import { ordersRepository } from '../repositories/orders';
```

## Practice Exercises

1. Design a frontend architecture for an e-commerce platform
2. Set up a CI/CD pipeline with staging and production
3. Implement feature flags with gradual rollout

### Solutions

<details>
<summary>Exercise 1: Architecture Outline</summary>

```text
Layers:
- Presentation: pages, components, design system
- State: local UI state + server cache (TanStack Query)
- Services: API client, auth, analytics, i18n, feature flags
- Edge: CDN, image optimization, request routing
- Infrastructure: build, deploy, observability
```

**Key points:**
- Each layer has a clear responsibility and interface.
- Shared services reduce duplication.
- Edge and infra decisions align with performance goals.

</details>

<details>
<summary>Exercise 2: CI/CD Pipeline</summary>

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm test -- --ci
      - run: npm run typecheck
      - run: npm run build
      - run: npm run lighthouse:ci

  deploy-staging:
    needs: build-test
    runs-on: ubuntu-latest
    steps:
      - run: ./scripts/deploy staging
      - run: ./scripts/smoke-test staging

  deploy-prod:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - run: ./scripts/deploy production --canary=10
      - run: ./scripts/verify canary
      - run: ./scripts/promote production
```

**Key points:**
- Tests gate deployments.
- Staging deploy happens before production.
- Canary reduces risk and supports rollback.

</details>

<details>
<summary>Exercise 3: Feature Flags</summary>

```jsx
function FeatureFlag({ flag, children }) {
  const flags = useFeatureFlags();
  return flags[flag] ? children : null;
}

// Usage
<FeatureFlag flag="new_checkout">
  <NewCheckout />
</FeatureFlag>
```

**Key points:**
- Flags allow gradual rollout with kill switches.
- Safe fallback exists if disabled.
- Centralized flag provider enables analytics and audits.

</details>

---

## What You Learned

This module covered:

- **Architecture layers**: Separating concerns across the system
- **Micro-frontends**: Team boundaries at scale
- **Build and deploy**: Pipelines that support rapid releases
- **CDN and caching**: Delivering assets fast and reliably
- **Budgets and observability**: Keeping performance and errors under control

**Key takeaway**: System design turns frontend work into a predictable, scalable delivery machine.

---

## Real-World Application

This week at work, you might use these concepts to:

- Propose a layered architecture for a new product
- Evaluate if micro-frontends fit your team structure
- Add performance budgets to CI
- Define CDN caching rules for static assets
- Improve alerting for frontend failures

---

## Further Reading

- [Micro Frontends](https://micro-frontends.org/)
- [Web.dev Performance](https://web.dev/performance/)

---

**Navigation**: [← Previous Module](./02-architecture-decision-records.md) | [Next Module →](./04-advanced-performance.md)
