# System Design

Design frontend systems that handle real-world scale and complexity.

## Learning Objectives

By the end of this module, you will:
- Design frontend architectures for scale
- Evaluate micro-frontend approaches
- Plan build and deployment strategies
- Make informed technology choices

## Frontend Architecture Layers

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

## Micro-Frontends

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

## Build System Design

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

## Deployment Strategies

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
3. Switch traffic to Green
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

## CDN and Caching Strategy

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

## Performance Budget

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

## Error Tracking Architecture

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

## Practice Exercises

1. Design a frontend architecture for an e-commerce platform
2. Set up a CI/CD pipeline with staging and production
3. Implement feature flags with gradual rollout

## Further Reading

- [Micro Frontends](https://micro-frontends.org/)
- [Web.dev Performance](https://web.dev/performance/)
