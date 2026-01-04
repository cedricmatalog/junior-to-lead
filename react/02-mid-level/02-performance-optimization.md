# Performance Optimization

Make your React applications fast and responsive.

## Learning Objectives

By the end of this module, you will:
- Understand React's rendering behavior
- Use memoization effectively (memo, useMemo, useCallback)
- Implement virtualization for large lists
- Analyze and optimize bundle size

## How React Renders

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

## React.memo

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

## useMemo

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

## useCallback

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

## Virtualization

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

## Code Splitting

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

## Bundle Analysis

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

## Profiling

Use React DevTools Profiler:

1. Start recording
2. Interact with your app
3. Stop recording
4. Analyze flame chart

Look for:
- Components that render too often
- Long render times
- Cascading re-renders

## Practice Exercises

1. Optimize a list of 10,000 items with virtualization
2. Use React DevTools Profiler to find and fix unnecessary re-renders
3. Implement code splitting for a multi-page app

## Further Reading

- [React.dev - Optimizing Performance](https://react.dev/learn/render-and-commit)
- [TanStack Virtual](https://tanstack.com/virtual/latest)
