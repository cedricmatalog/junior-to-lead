# Advanced Performance

> **Last reviewed**: 2026-01-06


## Week 23: The Performance Deadline

A major launch is coming, and leadership wants performance guarantees. Sarah shows you the Core Web Vitals report and points to a regression on mobile. Marcus adds that optimization cannot be guesswork -- it needs measurable targets and repeatable strategies. This week is about advanced performance work: rendering strategies, caching layers, profiling, and the guardrails that keep speed from degrading after release.

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Core Web Vitals** | A health check - measured signals of user experience |
| **Rendering strategy** | A staging plan - decide what shows up when |
| **Caching** | A pantry - reuse what is already prepared |
| **Profiling** | A heat map - find where the time goes |

Keep these in mind. They'll click as we build.

---

## Prerequisites

Module 22 (System Design) - Familiarity with delivery, caching, and system-level trade-offs.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Optimize Core Web Vitals with concrete tactics
- [ ] Choose rendering strategies based on product needs
- [ ] Design caching layers for predictable speed
- [ ] Profile and isolate performance bottlenecks
- [ ] Set guardrails to prevent regressions
- [ ] Communicate performance trade-offs to stakeholders

---

## Time Estimate

- **Reading**: 70-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice performance work over 6-8 weeks

---

## Chapter 1: Core Web Vitals

You need concrete targets that the whole team can measure.

### LCP (Largest Contentful Paint)

Target: < 2.5 seconds

```tsx
// Preload critical images
<link rel="preload" as="image" href="/hero.webp" />

// Priority hints
<img src="/hero.webp" fetchpriority="high" />

// Optimize image loading
<Image
  src="/hero.webp"
  priority // Next.js
  loading="eager"
  width={1200}
  height={600}
/>
```

### INP (Interaction to Next Paint)

Target: < 200ms

```tsx
// Break up long tasks
function processLargeList(items: Item[]) {
  const CHUNK_SIZE = 100;
  let index = 0;

  function processChunk() {
    const chunk = items.slice(index, index + CHUNK_SIZE);
    chunk.forEach(processItem);
    index += CHUNK_SIZE;

    if (index < items.length) {
      // Yield to browser between chunks
      requestIdleCallback(processChunk);
    }
  }

  processChunk();
}

// Use transitions for non-urgent updates
import { useTransition } from 'react';

function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    setQuery(e.target.value); // Urgent: update input

    startTransition(() => {
      setResults(filterResults(e.target.value)); // Non-urgent
    });
  };

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <Results items={results} />
    </>
  );
}
```

### CLS (Cumulative Layout Shift)

Target: < 0.1

```tsx
// Reserve space for images
<img
  src="/photo.jpg"
  width={800}
  height={600}
  style={{ aspectRatio: '4/3' }}
/>

// Skeleton loaders
function CardSkeleton() {
  return (
    <div className="card" style={{ height: 200 }}>
      <div className="skeleton" />
    </div>
  );
}

// Avoid inserting content above existing content
// Bad: Banner appears and pushes content down
// Good: Reserve space or use overlay
```

## Chapter 2: Rendering Strategies

Different pages need different trade-offs between speed and freshness.

### SSR (Server-Side Rendering)

```tsx
// Next.js App Router - Server Component
async function ProductPage({ params }) {
  const product = await getProduct(params.id);

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <AddToCartButton productId={product.id} />
    </div>
  );
}
```

### SSG (Static Site Generation)

```tsx
// Pre-render at build time
export async function generateStaticParams() {
  const products = await getProducts();
  return products.map(p => ({ id: p.id }));
}

// Page uses params at build time
async function ProductPage({ params }) {
  const product = await getProduct(params.id);
  // ...
}
```

### ISR (Incremental Static Regeneration)

```tsx
// Revalidate every 60 seconds
async function getProduct(id: string) {
  const res = await fetch(`${API}/products/${id}`, {
    next: { revalidate: 60 },
  });
  return res.json();
}
```

### Streaming SSR

```tsx
// Stream UI progressively
import { Suspense } from 'react';

function ProductPage() {
  return (
    <div>
      <ProductDetails /> {/* Renders immediately */}

      <Suspense fallback={<ReviewsSkeleton />}>
        <Reviews /> {/* Streams when ready */}
      </Suspense>

      <Suspense fallback={<RecommendationsSkeleton />}>
        <Recommendations /> {/* Streams when ready */}
      </Suspense>
    </div>
  );
}
```

## Chapter 3: Caching Strategies

Caching is the fastest performance win when applied thoughtfully.

### Client-Side Caching

```tsx
// TanStack Query stale-while-revalidate
const { data } = useQuery({
  queryKey: ['products'],
  queryFn: fetchProducts,
  staleTime: 5 * 60 * 1000,      // Fresh for 5 minutes
  gcTime: 30 * 60 * 1000,         // Cache for 30 minutes
  refetchOnWindowFocus: false,
});
```

### Service Worker Caching

```js
// sw.js
const CACHE_NAME = 'v1';
const STATIC_ASSETS = ['/app.js', '/styles.css', '/offline.html'];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => cache.addAll(STATIC_ASSETS))
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then(response => {
      // Cache first, then network
      return response || fetch(event.request);
    })
  );
});
```

### Edge Caching

```tsx
// Next.js edge caching
export const runtime = 'edge';

export async function GET(request: Request) {
  const data = await fetchData();

  return new Response(JSON.stringify(data), {
    headers: {
      'Cache-Control': 's-maxage=60, stale-while-revalidate=300',
      'CDN-Cache-Control': 'max-age=3600',
    },
  });
}
```

## Chapter 4: Performance Profiling

Profiling tells you where time is actually spent.

### React DevTools Profiler

1. Open DevTools Profiler tab
2. Click Record
3. Interact with app
4. Stop and analyze

Look for:
- Components that render too often
- Long render times
- Cascading updates

### Browser Performance API

```tsx
// Measure component render time
useEffect(() => {
  performance.mark('component-mount-start');

  return () => {
    performance.mark('component-mount-end');
    performance.measure(
      'component-mount',
      'component-mount-start',
      'component-mount-end'
    );
  };
}, []);

// Report Web Vitals
import { onCLS, onINP, onLCP } from 'web-vitals';

onCLS(console.log);
onINP(console.log);
onLCP(console.log);
```

### Lighthouse CI

```yaml
# .github/workflows/lighthouse.yml
- name: Lighthouse CI
  uses: treosh/lighthouse-ci-action@v10
  with:
    urls: |
      https://example.com/
      https://example.com/products
    budgetPath: ./lighthouse-budget.json
```

## Chapter 5: Guardrails and Regression Prevention

Performance wins fade unless you lock them in. Set budgets, automate checks, and make regressions visible.

```json
// lighthouse-budget.json
[
  {
    "path": "/*",
    "resourceSizes": [
      { "resourceType": "script", "budget": 250 },
      { "resourceType": "image", "budget": 400 }
    ],
    "timings": [
      { "metric": "first-contentful-paint", "budget": 2000 },
      { "metric": "largest-contentful-paint", "budget": 2500 }
    ]
  }
]
```

**Guardrails to consider:**
- Lighthouse budgets in CI
- Bundle size checks per route
- Alerts on Core Web Vital regressions

---

## Common Mistakes

1. **Optimizing without profiling** - Measure before you change anything.
2. **Ignoring cache invalidation** - Stale data can be worse than slow data.
3. **Overusing memoization** - It can add overhead without real gains.
4. **Skipping budgets** - Regressions go unnoticed until users complain.

## Practice Exercises

1. Optimize a page to pass Core Web Vitals
2. Implement streaming SSR with Suspense boundaries
3. Set up performance monitoring with alerts

### Solutions

<details>
<summary>Exercise 1: Core Web Vitals Fix</summary>

```tsx
// Preload hero and reduce layout shift
<link rel="preload" as="image" href="/hero.webp" />
<link rel="preconnect" href="https://cdn.example.com" />

<Image
  src="/hero.webp"
  width={1200}
  height={600}
  sizes="(max-width: 768px) 100vw, 1200px"
  priority
  alt="Hero"
/>

// Avoid CLS for fonts
// @font-face { font-display: swap; }
```

**Key points:**
- Preload critical assets.
- Provide explicit dimensions to reduce CLS.
- Use responsive sizes to avoid over-downloading.
- Prevent font swaps from shifting layout.

</details>

<details>
<summary>Exercise 2: Streaming SSR</summary>

```tsx
// app/products/page.tsx
import { Suspense } from 'react';

export default function ProductsPage() {
  return (
    <>
      <Hero />
      <Suspense fallback={<ProductSkeleton />}>
        <ProductGrid />
      </Suspense>
      <Suspense fallback={<FiltersSkeleton />}>
        <Filters />
      </Suspense>
    </>
  );
}
```

**Key points:**
- Suspense allows streaming of partial UI.
- The page becomes interactive sooner.
- Slow sections load independently.
- Skeletons set user expectations.

</details>

<details>
<summary>Exercise 3: Monitoring Alerts</summary>

```js
import { onCLS, onINP, onLCP } from 'web-vitals';

function report(metric) {
  const payload = {
    ...metric,
    path: window.location.pathname,
    buildId: window.__BUILD_ID__
  };

  navigator.sendBeacon('/api/metrics', JSON.stringify(payload));
}

onCLS(report);
onINP(report);
onLCP(report);
```

**Key points:**
- Send vitals to a backend for aggregation.
- Include route/build context for regressions.
- Alert when thresholds are exceeded.
- Track regressions per release.

</details>

---

## What You Learned

This module covered:

- **Core Web Vitals**: Concrete performance targets
- **Rendering strategies**: SSR, streaming, and client trade-offs
- **Caching**: Faster responses with predictable rules
- **Profiling**: Data-driven optimization
- **Guardrails**: Budgets and alerts that prevent regressions

**Key takeaway**: Sustainable performance comes from measurement plus guardrails, not one-off fixes.

---

## Real-World Application

This week at work, you might use these concepts to:

- Set Core Web Vital targets for a new release
- Split a heavy page into streamed sections
- Add edge caching to slow endpoints
- Introduce Lighthouse budgets in CI
- Report performance metrics to observability tooling

---

## Further Reading

- [Web.dev Core Web Vitals](https://web.dev/vitals/)
- [React Performance](https://react.dev/learn/render-and-commit)

---

**Navigation**: [<- Previous Module](./02-system-design.md) | [Next Module ->](./04-security-practices.md)
