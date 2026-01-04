# Advanced Performance

Optimize for real-world performance metrics that matter to users.

## Learning Objectives

By the end of this module, you will:
- Optimize for Core Web Vitals
- Implement server-side rendering strategies
- Design effective caching architectures
- Profile and debug performance issues

## Core Web Vitals

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

## Rendering Strategies

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

## Caching Strategies

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

## Performance Profiling

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

## Practice Exercises

1. Optimize a page to pass Core Web Vitals
2. Implement streaming SSR with Suspense boundaries
3. Set up performance monitoring with alerts

## Further Reading

- [Web.dev Core Web Vitals](https://web.dev/vitals/)
- [React Performance](https://react.dev/learn/render-and-commit)
