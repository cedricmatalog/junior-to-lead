# Routing and Navigation

> **Last reviewed**: 2026-01-06


## Week Five: The Multi-Page Illusion

The product catalog is growing fast. Marketing wants separate pages for the home feed, product details, and the checkout flow. The catch: they still want it to feel instant, like a single app.

"Welcome to routing," Sarah says. "You need real URLs, browser back/forward, and direct links. But you still want React to control the UI."

Marcus adds, "Good routing makes the app feel solid. Bad routing makes it feel broken."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Route** | A road sign - it matches a path and points to a view |
| **Router** | A traffic controller - it decides which screen to show |
| **Nested routes** | A building with floors - shared layout, different rooms |
| **Navigation** | A train switch - change tracks without reloading |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Junior Module 03 (Hooks Deep Dive) - Comfort with components, props, and basic hooks.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Set up React Router and define core routes
- [ ] Build navigation links that preserve SPA behavior
- [ ] Create nested layouts with shared UI using `Outlet`
- [ ] Read URL parameters and query strings for dynamic pages
- [ ] Implement protected routes for authenticated pages
- [ ] Handle not-found routes and safe redirects

---

## Time Estimate

- **Reading**: 60-75 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice routing patterns over 2-3 weeks in real apps

---

## Chapter 1: The First Route

Sarah hands you a short list: Home, Catalog, Checkout.

"Start with the routes," she says.

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/catalog" element={<Catalog />} />
        <Route path="/checkout" element={<Checkout />} />
      </Routes>
    </BrowserRouter>
  );
}
```

"Routes are just a switchboard," Marcus says. "Path in, screen out."

---

## Chapter 2: Navigation That Feels Instant

You add a header with links. A normal `<a>` tag reloads the page. That breaks the SPA feel.

"Use `Link`," Sarah says.

```jsx
import { Link } from 'react-router-dom';

function Header() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/catalog">Catalog</Link>
      <Link to="/checkout">Checkout</Link>
    </nav>
  );
}
```

"Now the URL changes, the page doesn't reload, and React stays in control."

---

## Chapter 3: Shared Layouts with Nested Routes

The catalog pages all need the same sidebar and header.

"Don't repeat layout code," Marcus says. "Use a parent route with an `Outlet`."

```jsx
import { Routes, Route, Outlet } from 'react-router-dom';

function CatalogLayout() {
  return (
    <div className="catalog-layout">
      <CatalogSidebar />
      <main>
        <Outlet />
      </main>
    </div>
  );
}

function AppRoutes() {
  return (
    <Routes>
      <Route path="/catalog" element={<CatalogLayout />}>
        <Route index element={<CatalogList />} />
        <Route path=":productId" element={<ProductDetails />} />
      </Route>
    </Routes>
  );
}
```

"One layout, many pages. That's how you scale routing without duplication."

---

## Chapter 4: Dynamic Routes and Params

Marketing wants URLs like `/catalog/sku-123`.

"That's a route param," Sarah says.

```jsx
import { useParams } from 'react-router-dom';

function ProductDetails() {
  const { productId } = useParams();

  return (
    <section>
      <h1>Product: {productId}</h1>
      <ProductView id={productId} />
    </section>
  );
}
```

"Now one component can handle every product page."

---

## Chapter 4.1: Query Strings for UI State

The filters team wants URLs like `/catalog?sort=price&view=grid`.

"That's a query string," Marcus says. "Use it for UI state you want to share or bookmark."

```jsx
import { useSearchParams } from 'react-router-dom';

function CatalogToolbar() {
  const [searchParams, setSearchParams] = useSearchParams();
  const sort = searchParams.get('sort') ?? 'featured';

  function updateSort(nextSort) {
    setSearchParams(prev => {
      prev.set('sort', nextSort);
      return prev;
    });
  }

  return (
    <select value={sort} onChange={e => updateSort(e.target.value)}>
      <option value="featured">Featured</option>
      <option value="price">Price</option>
      <option value="rating">Rating</option>
    </select>
  );
}
```

"Params identify *what* you're looking at. Query strings describe *how* you're viewing it."

---

## Chapter 5: Protected Routes

Checkout should be locked behind login.

"Wrap the route in a guard," Marcus says.

```jsx
import { Navigate } from 'react-router-dom';

function RequireAuth({ user, children }) {
  if (!user) return <Navigate to="/login" replace />;
  return children;
}

<Route
  path="/checkout"
  element={
    <RequireAuth user={user}>
      <Checkout />
    </RequireAuth>
  }
/>
```

"The guard decides, the router obeys."

---

## Chapter 6: Not Found and Safe Redirects

Users will mistype URLs. You need a 404 screen.

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/catalog" element={<Catalog />} />
  <Route path="/checkout" element={<Checkout />} />
  <Route path="*" element={<NotFound />} />
</Routes>
```

"Always end with a catch-all route," Sarah says. "It keeps users from falling into the void."

---

## Common Mistakes

1. **Using `<a>` for internal links** - It reloads the app and loses state.
2. **Forgetting `Outlet` in nested routes** - Child pages never render.
3. **Hardcoding params** - Use `useParams` so routes stay dynamic.
4. **Missing a 404** - Users land on blank screens when paths don't match.


Example:
```jsx
// ❌ Reloads the page
<a href="/catalog">Catalog</a>

// ✅ Client-side navigation
<Link to="/catalog">Catalog</Link>
```

---

## Debug This Code

Before moving to the exercises, test your debugging skills. Each snippet has bugs - can you spot and fix them?

<details>
<summary>Challenge 1: Nested Routes That Don't Render</summary>

```jsx
function CatalogLayout() {
  return (
    <div>
      <Sidebar />
      <main>
        {/* Children never show up */}
      </main>
    </div>
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

Nested routes render through a special component.

</details>

<details>
<summary>Solution</summary>

**Bug**: Missing `Outlet` for nested routes.

**Fixed code:**

```jsx
import { Outlet } from 'react-router-dom';

function CatalogLayout() {
  return (
    <div>
      <Sidebar />
      <main>
        <Outlet />
      </main>
    </div>
  );
}
```

**Why it matters**: Without `Outlet`, child routes never render.

</details>
</details>

<details>
<summary>Challenge 2: Link That Refreshes</summary>

```jsx
function Nav() {
  return (
    <nav>
      <a href="/checkout">Checkout</a>
    </nav>
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

SPA navigation should not reload the page.

</details>

<details>
<summary>Solution</summary>

**Bug**: Using `<a>` instead of `Link`.

**Fixed code:**

```jsx
import { Link } from 'react-router-dom';

function Nav() {
  return (
    <nav>
      <Link to="/checkout">Checkout</Link>
    </nav>
  );
}
```

**Why it matters**: Reloads lose state and feel slow.

</details>
</details>

<details>
<summary>Challenge 3: Guard That Never Renders</summary>

```jsx
function RequireAuth({ user, children }) {
  if (!user) return <Navigate to="/login" />;
  return;
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

What should render when the user is logged in?

</details>

<details>
<summary>Solution</summary>

**Bug**: Returns nothing when authenticated.

**Fixed code:**

```jsx
function RequireAuth({ user, children }) {
  if (!user) return <Navigate to="/login" replace />;
  return children;
}
```

**Why it matters**: The protected route should render its children when allowed.

</details>
</details>

---

## Practice Exercises

1. Build a navigation header with active link styling
2. Create a product details page using a route param
3. Add a protected settings route with a login redirect

### Solutions

<details>
<summary>Exercise 1: Active Link Styling</summary>

```jsx
import { NavLink } from 'react-router-dom';

function Header() {
  return (
    <nav>
      <NavLink to="/" className={({ isActive }) => isActive ? 'active' : ''}>
        Home
      </NavLink>
      <NavLink to="/catalog" className={({ isActive }) => isActive ? 'active' : ''}>
        Catalog
      </NavLink>
    </nav>
  );
}
```

**Key points:**
- `NavLink` gives you `isActive` to style current routes.
- Avoid manual URL checks.
- Keep active styles consistent across the app.

</details>

<details>
<summary>Exercise 2: Product Details Page</summary>

```jsx
import { useParams } from 'react-router-dom';

function ProductDetails() {
  const { productId } = useParams();

  return (
    <section>
      <h1>Product {productId}</h1>
      <ProductView id={productId} />
    </section>
  );
}
```

**Key points:**
- URL params make one component handle many products.
- Keep the URL as the source of truth.
- Use params to load data.

</details>

<details>
<summary>Exercise 3: Protected Settings Route</summary>

```jsx
function RequireAuth({ user, children }) {
  if (!user) return <Navigate to="/login" replace />;
  return children;
}

<Route
  path="/settings"
  element={
    <RequireAuth user={user}>
      <Settings />
    </RequireAuth>
  }
/>
```

**Key points:**
- Guards live at the route boundary.
- Use `replace` to avoid back-button loops.
- Keep redirect destinations consistent.

</details>

---

## What You Learned

This module covered:

- **Routes**: Map paths to screens with `Routes` and `Route`
- **Navigation**: Use `Link` and `NavLink` for SPA navigation
- **Nested layouts**: Share UI with `Outlet`
- **Dynamic paths**: Use `useParams` for product-style URLs
- **Guards**: Protect routes with `Navigate`
- **Fallbacks**: Catch bad URLs with a 404 route

**Key takeaway**: Routing gives users real URLs without sacrificing the SPA experience.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add a product details route with a dynamic `:id`
- Create a shared dashboard layout with nested routes
- Protect billing or admin routes behind authentication
- Add a 404 page for mistyped links
- Build a marketing page that deep-links to app screens

---

## Further Reading

- [React Router Docs](https://reactrouter.com/en/main)
- [Nested Routes](https://reactrouter.com/en/main/start/overview#nested-routes)
- [Protected Routes Patterns](https://reactrouter.com/en/main/start/tutorial)

---

**Navigation**: [← Previous Module](./04-state-management-basics.md) | [Next Module →](./06-forms-and-validation.md)
