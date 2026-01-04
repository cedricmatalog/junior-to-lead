# Architecture Patterns

Design scalable frontend architectures that grow with your team and product.

## Learning Objectives

By the end of this module, you will:
- Apply feature-sliced design principles
- Define clear module boundaries
- Structure monorepos effectively
- Make architectural decisions with trade-offs in mind

## Feature-Sliced Design

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

## Module Boundaries

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

## Monorepo Structure

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

## Dependency Injection

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

## Trade-off Analysis

When making architectural decisions, document trade-offs:

```markdown
## Decision: State Management Approach

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

## Practice Exercises

1. Refactor a flat project structure to feature-sliced design
2. Set up a monorepo with shared UI library
3. Write an ADR for a technical decision

## Further Reading

- [Feature-Sliced Design](https://feature-sliced.design/)
- [Turborepo Handbook](https://turbo.build/repo/docs)
