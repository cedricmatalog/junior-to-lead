# React Fundamentals

## Your First Day on the Job

It's Monday morning. You've just joined a startup as a frontend developer, and your tech lead Sarah drops by your desk.

"Welcome aboard! We're building a product catalog for our e-commerce platform. I need you to create a page that displays products, handles out-of-stock items, and shows sale badges. Sound good?"

You nod, open your editor, and stare at the blank file.

Where do you even start?

---

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **JSX** | A recipe card - familiar format (HTML-like), but JavaScript underneath |
| **Components** | LEGO bricks - self-contained pieces you snap together and reuse |
| **Props** | An ingredient list - tells the component what values to use |

Keep these in mind. They'll click as we build.

---

## Prerequisites

None - this is the starting point.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Write JSX syntax confidently and understand how it transforms to JavaScript
- [ ] Build reusable React components using function syntax
- [ ] Pass data between components using props and understand the unidirectional data flow
- [ ] Implement conditional rendering patterns (ternary operators, && operator, early returns)
- [ ] Render dynamic lists of data using .map() with proper key management
- [ ] Debug common JSX errors and component issues

---

## Time Estimate

- **Reading**: 45-60 minutes
- **Exercises**: 2-3 hours
- **Mastery**: Practice these concepts over 1-2 weeks with real projects

---

## What You'll Learn

By the end of this module, you'll be able to:
- Write JSX without thinking twice about the syntax
- Build components that you (and your team) can reuse anywhere
- Pass data between components using props
- Show different UI based on conditions
- Render lists of items (like products) dynamically

Let's build that product catalog.

---

## Chapter 1: The Language React Speaks

You start typing HTML out of habit:

```html
<div class="product">
  <h1>Wireless Headphones</h1>
</div>
```

But your editor screams red. Sarah walks by and glances at your screen.

"Ah, you're writing HTML. React uses JSX - it *looks* like HTML, but it's actually JavaScript wearing a disguise."

She pulls up a chair.

"See that `class`? In JavaScript, `class` is a reserved keyword. So in JSX, we use `className` instead."

```jsx
// This is JSX - looks like HTML, thinks like JavaScript
const product = (
  <div className="product">
    <h1>Wireless Headphones</h1>
  </div>
);
```

"And here's the trick," she continues. "That JSX gets transformed into this:"

```jsx
const product = React.createElement(
  'div',
  { className: 'product' },
  React.createElement('h1', null, 'Wireless Headphones')
);
```

Here's how that transformation works:

```
┌─────────────────────────────────────────────────────────────┐
│                    JSX Transformation                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  JSX (What you write):                                      │
│  ┌──────────────────────────────────────────────────┐      │
│  │ <div className="product">                        │      │
│  │   <h1>Wireless Headphones</h1>                   │      │
│  │ </div>                                           │      │
│  └──────────────────────────────────────────────────┘      │
│                          │                                  │
│                          ↓ Babel transforms                 │
│                          │                                  │
│  JavaScript (What runs):                                    │
│  ┌──────────────────────────────────────────────────┐      │
│  │ React.createElement(                             │      │
│  │   'div',                    ← Element type       │      │
│  │   { className: 'product' }, ← Props object       │      │
│  │   React.createElement(      ← Children          │      │
│  │     'h1',                                        │      │
│  │     null,                                        │      │
│  │     'Wireless Headphones'                        │      │
│  │   )                                              │      │
│  │ )                                                │      │
│  └──────────────────────────────────────────────────┘      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

You wince. "That's... verbose."

"Exactly why we have JSX. It's syntactic sugar that makes React bearable to write."

### The Translation Guide

You jot down the differences between HTML and JSX:

| HTML | JSX | Why? |
|------|-----|------|
| `class` | `className` | `class` is reserved in JS |
| `for` | `htmlFor` | `for` is reserved in JS |
| `onclick` | `onClick` | JSX uses camelCase |
| `<img>` | `<img />` | All tags must close |
| `tabindex` | `tabIndex` | camelCase, always |

### Embedding JavaScript

"One more thing," Sarah adds. "When you need JavaScript inside JSX, use curly braces. Think of them as escape hatches back into JavaScript-land."

```jsx
const productName = "Wireless Headphones";
const price = 79.99;
const isOnSale = true;

const product = (
  <div className="product">
    <h1>{productName}</h1>
    <p>Price: ${price}</p>
    <p>Status: {isOnSale ? "On Sale!" : "Regular Price"}</p>
  </div>
);
```

You try it. It works. The curly braces feel natural now - a window from JSX into JavaScript.

---

## Chapter 2: Building Blocks

By lunch, you've got a single product displaying. But Sarah's brief mentioned a *catalog*. Dozens of products.

You start copy-pasting your code and changing the product name each time.

```jsx
<div className="product">
  <h1>Wireless Headphones</h1>
  <p>$79.99</p>
</div>

<div className="product">
  <h1>Bluetooth Speaker</h1>
  <p>$49.99</p>
</div>

<div className="product">
  <h1>USB-C Cable</h1>
  <p>$12.99</p>
</div>

// ... 47 more times? No way.
```

Your colleague Marcus notices your pain.

"You're thinking like it's 2010. In React, we build *components* - reusable templates that accept different data."

He shows you:

```jsx
function ProductCard({ name, price }) {
  return (
    <div className="product">
      <h1>{name}</h1>
      <p>${price}</p>
    </div>
  );
}
```

"Now you can use it like this:"

```jsx
<ProductCard name="Wireless Headphones" price={79.99} />
<ProductCard name="Bluetooth Speaker" price={49.99} />
<ProductCard name="USB-C Cable" price={12.99} />
```

Your eyes light up. Same structure, different data. You write once, use everywhere.

### The Rules of Components

Marcus shares the unwritten rules:

1. **Always capitalize component names.** `ProductCard`, not `productCard`. React uses this to distinguish your components from HTML tags.

2. **Return one root element.** This fails:
   ```jsx
   // ❌ Two siblings, no parent
   function Product() {
     return (
       <h1>Title</h1>
       <p>Price</p>
     );
   }
   ```

   Fix it with a wrapper or Fragment:
   ```jsx
   // ✅ Wrapped in Fragment (invisible container)
   function Product() {
     return (
       <>
         <h1>Title</h1>
         <p>Price</p>
       </>
     );
   }
   ```

3. **Components are functions.** They take input (props), return output (JSX). Pure and predictable.

---

## Chapter 3: The Flow of Data

After lunch, you've got ProductCard working. But now Sarah asks: "Can you add an image, rating, and 'Add to Cart' button?"

You update the component:

```jsx
function ProductCard({ name, price, image, rating }) {
  return (
    <article className="product-card">
      <img src={image} alt={name} />
      <h2>{name}</h2>
      <StarRating value={rating} />
      <p className="price">${price}</p>
      <button>Add to Cart</button>
    </article>
  );
}
```

Those values in the curly braces - `name`, `price`, `image`, `rating` - are called **props** (short for properties). They flow *downward* from parent to child, like water flowing downstream.

```jsx
// Parent passes data down via props
<ProductCard
  name="Wireless Headphones"
  price={79.99}
  image="/headphones.jpg"
  rating={4.5}
/>
```

### Props Are Read-Only

You try something clever - modifying a prop inside the component:

```jsx
function ProductCard({ price }) {
  price = price * 0.9; // Apply 10% discount? ❌
  return <p>${price}</p>;
}
```

Marcus catches you. "Props are read-only. The parent owns that data. If you need to transform it, create a new variable."

```jsx
function ProductCard({ price, discount = 0 }) {
  const finalPrice = price * (1 - discount / 100); // ✅ New variable
  return <p>${finalPrice.toFixed(2)}</p>;
}
```

### Default Props

Speaking of that `discount = 0` - that's a default value. If no discount is passed, it defaults to zero. No more undefined errors.

```jsx
function ProductCard({
  name,
  price,
  image = "/placeholder.jpg",  // Default if missing
  rating = 0,
  inStock = true
}) {
  // ...
}
```

---

## Chapter 4: Decisions, Decisions

The afternoon brings a new requirement. Sarah messages you:

> "Products that are out of stock should show 'Out of Stock' instead of 'Add to Cart'. And disable the button."

Time to make your UI *conditional*.

### Pattern 1: The && Operator

For simple "show this if true" cases:

```jsx
function ProductCard({ name, isNew }) {
  return (
    <article>
      <h2>{name}</h2>
      {isNew && <span className="badge">New!</span>}
    </article>
  );
}
```

If `isNew` is true, the badge appears. If false, nothing renders. Clean and simple.

### Pattern 2: The Ternary Operator

For "either this or that" cases:

```jsx
function ProductCard({ name, inStock }) {
  return (
    <article>
      <h2>{name}</h2>
      {inStock ? (
        <button className="add-to-cart">Add to Cart</button>
      ) : (
        <button disabled className="sold-out">Out of Stock</button>
      )}
    </article>
  );
}
```

### Pattern 3: Early Return

When you want to bail out entirely:

```jsx
function ProductCard({ product }) {
  if (!product) {
    return <p>Loading...</p>;
  }

  return (
    <article>
      <h2>{product.name}</h2>
      {/* ... rest of the card */}
    </article>
  );
}
```

You combine all three for the product card:

```jsx
function ProductCard({ product }) {
  const { name, price, image, inStock, isNew, discount } = product;

  return (
    <article className="product-card">
      <img src={image} alt={name} />

      {/* Pattern 1: Show badge if new */}
      {isNew && <span className="badge new">New!</span>}

      {/* Pattern 1: Show badge if on sale */}
      {discount > 0 && <span className="badge sale">-{discount}%</span>}

      <h2>{name}</h2>

      {/* Pattern 2: Show discounted or regular price */}
      <p className="price">
        {discount > 0 ? (
          <>
            <span className="original">${price}</span>
            <span className="discounted">
              ${(price * (1 - discount / 100)).toFixed(2)}
            </span>
          </>
        ) : (
          <span>${price}</span>
        )}
      </p>

      {/* Pattern 2: Show add to cart or out of stock */}
      {inStock ? (
        <button>Add to Cart</button>
      ) : (
        <button disabled>Out of Stock</button>
      )}
    </article>
  );
}
```

---

## Chapter 5: The List That Builds Itself

End of day. You have one beautiful ProductCard. But the catalog has 50 products. Time to render a list.

In JavaScript, when you want to transform an array, you use `.map()`. Same idea here:

```jsx
function ProductGrid({ products }) {
  return (
    <div className="product-grid">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

Wait - what's that `key` thing?

### The Key to Efficient Updates

React needs to track which items changed, were added, or were removed. The `key` prop is how it keeps track.

```jsx
// ✅ Good: Unique, stable identifier
{products.map(product => (
  <ProductCard key={product.id} product={product} />
))}

// ❌ Bad: Index changes when items reorder
{products.map((product, index) => (
  <ProductCard key={index} product={product} />
))}

// ❌ Bad: Random values regenerate every render
{products.map(product => (
  <ProductCard key={Math.random()} product={product} />
))}
```

Sarah reviews your code. "Using the product ID as the key - smart. That way React knows exactly which card to update when data changes."

### Handling Empty Lists

What if there are no products? Don't render an empty grid:

```jsx
function ProductGrid({ products }) {
  if (products.length === 0) {
    return <p>No products found.</p>;
  }

  return (
    <div className="product-grid">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

---

## Chapter 6: Putting It All Together

It's 5:30 PM. You step back and look at what you've built:

```jsx
// ProductCard.jsx
function ProductCard({ product }) {
  const { name, price, image, inStock, isNew, discount, rating } = product;

  return (
    <article className="product-card">
      <img src={image} alt={name} />

      {isNew && <span className="badge new">New</span>}
      {discount > 0 && <span className="badge sale">-{discount}%</span>}

      <h2>{name}</h2>
      <StarRating value={rating} />

      <p className="price">
        {discount > 0 ? (
          <>
            <span className="original">${price}</span>
            <span className="discounted">
              ${(price * (1 - discount / 100)).toFixed(2)}
            </span>
          </>
        ) : (
          <span>${price}</span>
        )}
      </p>

      {inStock ? (
        <button>Add to Cart</button>
      ) : (
        <button disabled>Out of Stock</button>
      )}
    </article>
  );
}

// ProductGrid.jsx
function ProductGrid({ products, category }) {
  const filteredProducts = category === 'all'
    ? products
    : products.filter(p => p.category === category);

  if (filteredProducts.length === 0) {
    return <p>No products found in this category.</p>;
  }

  return (
    <div className="product-grid">
      {filteredProducts.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// StorePage.jsx
function StorePage() {
  // Hardcoded data for now - we'll learn how to fetch from APIs later
  const products = [
    { id: 1, name: 'Wireless Headphones', price: 79.99, image: '/headphones.jpg', inStock: true, rating: 4.5 },
    { id: 2, name: 'Bluetooth Speaker', price: 49.99, image: '/speaker.jpg', inStock: true, rating: 4.2 },
    { id: 3, name: 'USB-C Cable', price: 12.99, image: '/cable.jpg', inStock: false, rating: 4.8 },
  ];

  return (
    <main>
      <h1>Our Products</h1>
      <ProductGrid products={products} category="all" />
    </main>
  );
}
```

Sarah walks by on her way out. She glances at your screen.

"Ship it."

The component hierarchy you just built looks like this:

```
┌─────────────────────────────────────────────────────────┐
│              Component Hierarchy                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│                    StorePage                            │
│                        │                                │
│                        ├─ products data                 │
│                        │                                │
│                        ↓                                │
│                  ProductGrid                            │
│                  (filters products)                     │
│                        │                                │
│              ┌─────────┼─────────┐                      │
│              ↓         ↓         ↓                      │
│        ProductCard ProductCard ProductCard              │
│        (id: 1)     (id: 2)     (id: 3)                  │
│           │            │            │                   │
│           ├─ name      ├─ name      ├─ name            │
│           ├─ price     ├─ price     ├─ price           │
│           ├─ image     ├─ image     ├─ image           │
│           ├─ rating    ├─ rating    ├─ rating          │
│           └─ inStock   └─ inStock   └─ inStock         │
│                                                         │
│  Data flows DOWN (props) →                             │
│  Components are REUSABLE →                             │
│  Each ProductCard gets different data but same logic   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## The Traps to Avoid

Before you go, Marcus shares the mistakes he made when he started:

| Mistake | What Happens | The Fix |
|---------|--------------|---------|
| Using `class` instead of `className` | Doesn't work, confusing error | Always use `className` in JSX |
| Mutating props | Unpredictable bugs, React won't re-render | Create new variables instead |
| Using array index as key | Bugs when items reorder or delete | Use unique, stable IDs |
| Forgetting curly braces | Variable name shows literally | `{variable}` not `variable` |
| Multiple root elements | Syntax error | Wrap in `<>...</>` Fragment |

---

## Your Turn

You've got the fundamentals. Now practice:

1. **Extend the ProductCard** - Add a "Wishlist" button that only appears when the user hovers
2. **Build a CategoryFilter** - A row of buttons that filters products by category
3. **Create a Navbar** - Shows "Login" for guests, "Welcome, {name}" for logged-in users

### Solutions

<details>
<summary>Exercise 1: Wishlist Button with Hover</summary>

```jsx
function ProductCard({ product }) {
  const { name, price, image, inStock } = product;
  const [isHovered, setIsHovered] = useState(false);

  return (
    <article
      className="product-card"
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      <img src={image} alt={name} />
      <h2>{name}</h2>
      <p className="price">${price}</p>

      {inStock ? (
        <button>Add to Cart</button>
      ) : (
        <button disabled>Out of Stock</button>
      )}

      {/* Wishlist button only shows on hover */}
      {isHovered && (
        <button className="wishlist-btn">
          ♥ Add to Wishlist
        </button>
      )}
    </article>
  );
}
```

**Key points:**
- Use `onMouseEnter` and `onMouseLeave` to track hover state
- Store hover state with `useState` (we'll learn this in Module 3)
- Use conditional rendering (`&&`) to show/hide the button
- Consider accessibility: keyboard users won't see the button! Better approach: always show, but style differently on hover

</details>

<details>
<summary>Exercise 2: Category Filter</summary>

```jsx
function CategoryFilter({ categories, selectedCategory, onCategoryChange }) {
  return (
    <div className="category-filter">
      <button
        className={selectedCategory === 'all' ? 'active' : ''}
        onClick={() => onCategoryChange('all')}
      >
        All
      </button>
      {categories.map(category => (
        <button
          key={category}
          className={selectedCategory === category ? 'active' : ''}
          onClick={() => onCategoryChange(category)}
        >
          {category}
        </button>
      ))}
    </div>
  );
}

// Usage in parent component
function StorePage() {
  const [selectedCategory, setSelectedCategory] = useState('all');
  const categories = ['Electronics', 'Clothing', 'Books'];

  const products = [/* ... */];
  const filteredProducts = selectedCategory === 'all'
    ? products
    : products.filter(p => p.category === selectedCategory);

  return (
    <main>
      <h1>Our Products</h1>
      <CategoryFilter
        categories={categories}
        selectedCategory={selectedCategory}
        onCategoryChange={setSelectedCategory}
      />
      <ProductGrid products={filteredProducts} />
    </main>
  );
}
```

**Key points:**
- Parent component owns the selected category state
- Pass `onCategoryChange` callback to update state
- Use `className` conditionally to highlight active category
- Filter products based on selected category

</details>

<details>
<summary>Exercise 3: Navbar with Conditional Rendering</summary>

```jsx
function Navbar({ user }) {
  return (
    <nav className="navbar">
      <h1 className="logo">My Store</h1>

      <ul className="nav-links">
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li><a href="/about">About</a></li>
      </ul>

      <div className="nav-actions">
        {user ? (
          <>
            <span>Welcome, {user.name}</span>
            <button onClick={user.onLogout}>Logout</button>
          </>
        ) : (
          <button>Login</button>
        )}
      </div>
    </nav>
  );
}

// Usage
<Navbar user={null} /> {/* Guest: shows Login */}
<Navbar user={{ name: 'Alice', onLogout: handleLogout }} /> {/* Logged in */}
```

**Key points:**
- Use ternary operator for either/or rendering
- Fragment (`<>`) groups multiple elements without extra DOM node
- Props can be objects with multiple properties
- Parent component manages authentication state

</details>

---

## Debug This Code

Before moving to wrap-up, test your debugging skills. Each snippet has bugs - can you spot and fix them?

<details>
<summary>Challenge 1: Broken Product List</summary>

```jsx
function ProductList() {
  const products = [
    { name: 'Laptop', price: 999 },
    { name: 'Mouse', price: 29 },
    { name: 'Keyboard', price: 79 }
  ];

  return (
    <div>
      <h2>Products</h2>
      {products.map(product => (
        <div>
          <h3>{product.name}</h3>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

Think about what React needs to efficiently update lists. Check the console for warnings.

</details>

<details>
<summary>Solution</summary>

**Bug**: Missing `key` prop on list items. React needs keys to track which items changed.

**Fixed code:**

```jsx
function ProductList() {
  const products = [
    { name: 'Laptop', price: 999 },
    { name: 'Mouse', price: 29 },
    { name: 'Keyboard', price: 79 }
  ];

  return (
    <div>
      <h2>Products</h2>
      {products.map(product => (
        <div key={product.name}>
          <h3>{product.name}</h3>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  );
}
```

**Why it matters**: Without keys, React can't optimize updates and may re-render items incorrectly when the list changes.

</details>

</details>

<details>
<summary>Challenge 2: Conditional Rendering Gone Wrong</summary>

```jsx
function UserGreeting({ user }) {
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      {user.isPremium === true && (
        <span class="badge">Premium Member</span>
      )}
      {user.notifications > 0 && (
        <p>You have {user.notifications} new notifications</p>
      )}
    </div>
  );
}
```

**How many bugs can you find?** (Answer: 2 bugs)

<details>
<summary>Hint 1</summary>

Look at the HTML attributes. Are they using React syntax?

</details>

<details>
<summary>Hint 2</summary>

What happens when `user.notifications` is 0? What will render?

</details>

<details>
<summary>Solution</summary>

**Bug 1**: Using `class` instead of `className` - JSX uses JavaScript property names.

**Bug 2**: When `user.notifications` is 0, React will render `0` on the page (because `0 && anything` returns `0`).

**Fixed code:**

```jsx
function UserGreeting({ user }) {
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      {user.isPremium === true && (
        <span className="badge">Premium Member</span>
      )}
      {user.notifications > 0 && (
        <p>You have {user.notifications} new notifications</p>
      )}
    </div>
  );
}
```

**Alternative fix for notifications** (more defensive):

```jsx
{user.notifications > 0 ? (
  <p>You have {user.notifications} new notifications</p>
) : null}
```

**Why it matters**: `class` will silently fail or cause warnings. The `0` rendering issue creates unwanted visual bugs.

</details>

</details>

<details>
<summary>Challenge 3: Props and Destructuring Confusion</summary>

```jsx
function ProductCard(name, price, inStock) {
  return (
    <div className="card">
      <h3>{name}</h3>
      <p>${price}</p>
      {inStock ? (
        <button>Add to Cart</button>
      ) : (
        <p style="color: red;">Out of Stock</p>
      )}
    </div>
  );
}

// Usage
<ProductCard name="Laptop" price={999} inStock={true} />
```

**How many bugs can you find?** (Answer: 2 bugs)

<details>
<summary>Hint 1</summary>

How does React pass data to components? Check the function signature.

</details>

<details>
<summary>Hint 2</summary>

Look at the inline styles. Is that the correct JSX syntax?

</details>

<details>
<summary>Solution</summary>

**Bug 1**: Props are passed as an object, not individual parameters. Should destructure from `props` or accept `props` object.

**Bug 2**: Inline `style` attribute should be an object, not a string.

**Fixed code:**

```jsx
function ProductCard({ name, price, inStock }) {
  return (
    <div className="card">
      <h3>{name}</h3>
      <p>${price}</p>
      {inStock ? (
        <button>Add to Cart</button>
      ) : (
        <p style={{ color: 'red' }}>Out of Stock</p>
      )}
    </div>
  );
}

// Usage
<ProductCard name="Laptop" price={999} inStock={true} />
```

**Alternative without destructuring:**

```jsx
function ProductCard(props) {
  return (
    <div className="card">
      <h3>{props.name}</h3>
      <p>${props.price}</p>
      {props.inStock ? (
        <button>Add to Cart</button>
      ) : (
        <p style={{ color: 'red' }}>Out of Stock</p>
      )}
    </div>
  );
}
```

**Why it matters**: Incorrect prop handling means the component receives `undefined` values. Incorrect style syntax causes rendering errors.

</details>

</details>

---

## What You Learned

This module covered:

- **JSX Syntax**: HTML-like syntax that transforms to JavaScript, with key differences like className and self-closing tags
- **Components**: Reusable building blocks that accept props and return JSX
- **Props**: Read-only data flow from parent to child components
- **Conditional Rendering**: Using ternary operators, && operator, and early returns to show/hide UI elements
- **List Rendering**: Using .map() to transform arrays into components with unique keys

**Key takeaway**: React components are just JavaScript functions that return JSX, making UI predictable and reusable.

---

## Real-World Application

This week at work, you might use these concepts to:

- Build a product catalog with cards showing name, image, price, and stock status
- Create a user dashboard that displays different content based on authentication state
- Implement a notification list that dynamically renders items from an API response
- Build reusable UI components like buttons, cards, and badges that work across multiple pages

---

## The Journey Continues

Tomorrow, you'll learn about **component patterns** - how to build flexible, reusable components that work together like a well-oiled machine.

But for now? You just survived your first day.

---

## Further Reading

- [React.dev - Your First Component](https://react.dev/learn/your-first-component)
- [React.dev - Rendering Lists](https://react.dev/learn/rendering-lists)
- [React.dev - Conditional Rendering](https://react.dev/learn/conditional-rendering)

---

**Navigation**: [Next Module →](./02-component-patterns.md)
