# Performance Optimization

> **Last reviewed**: 2026-01-06


## Week 12: The Laggy Release

The dashboard launched with a splash, but now users are complaining about lag. Sarah pulls up a video: scrolling stutters, filters hang, and mobile devices choke. "We need this to feel instant," she says. Marcus reminds you that optimization isn't about magic tricks -- it's about understanding what React renders, why it renders, and which work can be skipped. Your job this week is to identify waste and make performance predictable.

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Rendering** | A camera refresh - every frame has a cost |
| **Memoization** | A sticky note - reuse results instead of recalculating |
| **Virtualization** | A window shade - only show what's visible |
| **Code splitting** | Packing cubes - load only what you need right now |

Keep these in mind. They'll click as we build.

---

## Prerequisites

Module 01 (Advanced Patterns) - Comfort with components, hooks, and composition patterns.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Explain how React decides when to re-render
- [ ] Apply React.memo, useMemo, and useCallback effectively
- [ ] Implement list virtualization to keep large lists fast
- [ ] Split bundles to improve initial load time
- [ ] Use bundle analysis to identify heavy dependencies
- [ ] Profile and prioritize the biggest performance wins

---

## Time Estimate

- **Reading**: 60-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice performance tuning over 4-6 weeks

---

## Chapter 1: How React Renders

The first step is understanding why that filter dropdown re-renders half the page.

React re-renders a component when:
1. Its state changes
2. Its props change
3. Its parent re-renders (even if props haven't changed)

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      <Child /> {/* Re-renders even though it has no props! */}
    </div>
  );
}
```

## Chapter 2: React.memo

Marcus points out a component that renders on every keystroke even though its props never change.

Prevents re-renders when props haven't changed.

```jsx
const ExpensiveList = memo(function ExpensiveList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
});

// Now only re-renders when `items` actually changes
```

**Custom comparison:**

```jsx
const UserCard = memo(
  function UserCard({ user }) {
    return <div>{user.name}</div>;
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    return prevProps.user.id === nextProps.user.id;
  }
);
```

## Chapter 3: useMemo

Your filtering logic is expensive. You need to cache the result until inputs actually change.

Memoize expensive calculations.

```jsx
function ProductList({ products, filter }) {
  // Recalculates only when products or filter change
  const filteredProducts = useMemo(() => {
    return products.filter(p => p.category === filter);
  }, [products, filter]);

  return <List items={filteredProducts} />;
}
```

**When to use useMemo:**
- Expensive calculations (sorting, filtering large arrays)
- Referential equality for objects/arrays passed to memoized children
- Values used in dependency arrays

**Don't overuse:**

```jsx
// Don't memoize simple operations
const fullName = useMemo(() => `${first} ${last}`, [first, last]); // Unnecessary
const fullName = `${first} ${last}`; // Just do this
```

## Chapter 4: useCallback

Passing new functions on every render keeps re-triggering child renders.

Memoize functions to prevent unnecessary re-renders.

```jsx
function SearchPage() {
  const [query, setQuery] = useState('');

  // Without useCallback, new function created every render
  // Causes SearchResults to re-render unnecessarily
  const handleSearch = useCallback((searchQuery) => {
    fetch(`/api/search?q=${searchQuery}`);
  }, []);

  return (
    <>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <SearchResults onSearch={handleSearch} />
    </>
  );
}

const SearchResults = memo(function SearchResults({ onSearch }) {
  // Only re-renders when onSearch reference changes
});
```

## Chapter 5: Virtualization

The activity feed has thousands of rows. Rendering all of them is killing scroll performance.

Only render visible items in large lists.

```jsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50, // estimated row height
  });

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          position: 'relative',
        }}
      >
        {virtualizer.getVirtualItems().map((virtualRow) => (
          <div
            key={virtualRow.index}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`,
            }}
          >
            {items[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Chapter 6: Code Splitting

The app loads a huge bundle on first paint. You need to load screens only when users navigate.

Load code only when needed.

```jsx
import { lazy, Suspense } from 'react';

// Split by route
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}

// Split by feature
const HeavyChart = lazy(() => import('./components/HeavyChart'));

function Analytics() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && (
        <Suspense fallback={<ChartSkeleton />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

## Chapter 7: Bundle Analysis

Sarah wants to know which dependencies are inflating the bundle. Time to measure, not guess.

```bash
# Using webpack-bundle-analyzer
npm install --save-dev webpack-bundle-analyzer

# Add to webpack config or use
npx source-map-explorer build/static/js/*.js
```

**Common optimizations:**
- Replace moment.js with date-fns or dayjs
- Import only what you need from lodash
- Use dynamic imports for heavy libraries
- Check for duplicate dependencies

```jsx
// Bad: imports entire library
import _ from 'lodash';
_.debounce(fn, 300);

// Good: import only what's needed
import debounce from 'lodash/debounce';
debounce(fn, 300);
```

## Chapter 8: Profiling

You need hard evidence before you optimize. The profiler will show where the time goes.

Use React DevTools Profiler:

1. Start recording
2. Interact with your app
3. Stop recording
4. Analyze flame chart

Look for:
- Components that render too often
- Long render times
- Cascading re-renders

---

## Common Mistakes

1. **Memoizing everything** - `useMemo` and `useCallback` add overhead when overused.
2. **Ignoring dependency arrays** - Stale values lead to bugs and misleading performance gains.
3. **Optimizing before measuring** - Always profile first to confirm the bottleneck.
4. **Rendering huge lists** - Without virtualization, large lists will always feel slow.

## Practice Exercises

1. Optimize a list of 10,000 items with virtualization
2. Use React DevTools Profiler to find and fix unnecessary re-renders
3. Implement code splitting for a multi-page app

### Solutions

<details>
<summary>Exercise 1: Virtualized List</summary>

```jsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualizedList({ items }) {
  const parentRef = useRef(null);

  const rowVirtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 40,
    overscan: 6
  });

  return (
    <div ref={parentRef} style={{ height: 400, overflow: 'auto' }}>
      <div
        style={{
          height: rowVirtualizer.getTotalSize(),
          position: 'relative'
        }}
      >
        {rowVirtualizer.getVirtualItems().map((virtualRow) => {
          const item = items[virtualRow.index];
          return (
            <div
              key={item.id}
              ref={rowVirtualizer.measureElement}
              style={{
                position: 'absolute',
                top: 0,
                left: 0,
                width: '100%',
                height: virtualRow.size,
                transform: `translateY(${virtualRow.start}px)`
              }}
            >
              {item.name}
            </div>
          );
        })}
      </div>
    </div>
  );
}
```

**Key points:**
- Only visible rows render; the rest are virtual.
- Overscan keeps scrolling smooth.
- `measureElement` supports variable row heights.

</details>

<details>
<summary>Exercise 2: Profiler Fix</summary>

```jsx
function Filters({ onChange }) {
  return (
    <div>
      <button onClick={() => onChange('popular')}>Popular</button>
      <button onClick={() => onChange('new')}>New</button>
    </div>
  );
}

const ResultsList = React.memo(function ResultsList({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.title}</li>
      ))}
    </ul>
  );
});

function SearchPage({ items }) {
  const [filter, setFilter] = useState('popular');
  const deferredFilter = useDeferredValue(filter);
  const filtered = useMemo(
    () => items.filter((item) => item.tag === deferredFilter),
    [items, deferredFilter]
  );

  const handleChange = useCallback((nextFilter) => {
    setFilter(nextFilter);
  }, []);

  return (
    <>
      <Filters onChange={handleChange} />
      <ResultsList items={filtered} />
    </>
  );
}
```

**Key points:**
- Memoized list prevents unnecessary renders.
- `useDeferredValue` keeps typing responsive under load.
- `useMemo` avoids repeated filtering work.
- `useCallback` stabilizes handler props.

</details>

<details>
<summary>Exercise 3: Code Splitting</summary>

```jsx
import { lazy, Suspense } from 'react';

const SettingsPage = lazy(() => import('./SettingsPage'));
const prefetchSettings = () => import('./SettingsPage');

function App() {
  return (
    <Router>
      <Routes>
        <Route
          path="/settings"
          element={
            <Suspense fallback={<div>Loading settings...</div>}>
              <SettingsPage />
            </Suspense>
          }
        />
      </Routes>
    </Router>
  );
}
```

**Key points:**
- `lazy` splits a route into a separate bundle.
- `Suspense` provides a loading state.
- Prefetching improves perceived navigation speed.

</details>

---

## What You Learned

This module covered:

- **Rendering behavior**: Why components re-render and when they don't need to
- **Memoization**: Using memo, useMemo, and useCallback responsibly
- **Virtualization**: Rendering only visible list rows
- **Bundle strategy**: Splitting and analyzing bundles
- **Profiling**: Measuring before optimizing

**Key takeaway**: Performance work is about removing wasted renders and unnecessary work, not guessing.

---

## Real-World Application

This week at work, you might use these concepts to:

- Speed up a dashboard with heavy charts or tables
- Fix filter UIs that re-render too much
- Split a large admin panel into smaller route bundles
- Reduce time-to-interactive on slow devices
- Build infinite-scroll feeds that stay smooth

---

## Further Reading

- [React.dev - Optimizing Performance](https://react.dev/learn/render-and-commit)
- [TanStack Virtual](https://tanstack.com/virtual/latest)

---

**Navigation**: [← Previous Module](./01-advanced-patterns.md) | [Next Module →](./03-state-at-scale.md)
      <Link to="/settings" onMouseEnter={prefetchSettings}>
        Settings
      </Link>
