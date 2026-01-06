# Styling Approaches

> **Last reviewed**: 2026-01-06


## Week Six: The Design System

The job application portal is complete. But when you open it in the browser, something feels off.

"It works," Marcus says, squinting at the screen. "But it looks like it was designed by an engineer."

He's not wrong. The buttons don't match. The spacing is inconsistent. Every component has its own shade of blue.

"You need a design system," Sarah says. "And that means choosing a styling approach. Today, you pick your weapon."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **CSS Modules** | Labeled storage bins - each component gets its own, no name collisions |
| **Tailwind CSS** | Pre-made building blocks - combine existing pieces quickly |
| **CSS-in-JS** | Custom furniture - styles designed specifically for each component |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 05 (Forms and Validation) - Understanding component structure and state management for styling dynamic components.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Choose between CSS Modules, Tailwind, and CSS-in-JS based on project needs
- [ ] Build reusable button variants using CSS Modules with scoped styles
- [ ] Create responsive layouts quickly with Tailwind's utility classes
- [ ] Implement dynamic theming with styled-components and ThemeProvider
- [ ] Define and use design tokens consistently across styling approaches
- [ ] Combine multiple styling approaches effectively in the same project

---

## Time Estimate

- **Reading**: 50-65 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Experiment with different approaches over 3-4 weeks to find your preference

---

## Chapter 1: The Options on the Table

Sarah pulls up a comparison chart:

| Approach | Scoping | DX | Performance | Learning Curve |
|----------|---------|-----|-------------|----------------|
| CSS Modules | Automatic | Good | Excellent | Low |
| Tailwind CSS | Utility classes | Excellent | Excellent | Medium |
| styled-components | Automatic | Good | Good | Medium |
| Plain CSS | Manual (BEM) | Basic | Excellent | Low |

"Each has trade-offs," she explains. "Here's what they look like in practice."

---

## Chapter 2: CSS Modules — The Safe Choice

"CSS Modules are the gateway drug," Marcus jokes. "Regular CSS, but with automatic scoping."

```css
/* Button.module.css */
.button {
  padding: 8px 16px;
  border-radius: 4px;
}

.primary {
  background: blue;
  color: white;
}

.secondary {
  background: gray;
  color: black;
}
```

```jsx
import styles from './Button.module.css';

function Button({ variant = 'primary', children }) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  );
}
```

"See how you import `styles`? Those class names get transformed at build time. Your `.button` becomes `.Button_button_x7h3k`. No collisions possible."

**Benefits:**
- No naming collisions
- Regular CSS (easy to learn)
- Zero runtime cost
- Works with all CSS features

---

## Chapter 3: Tailwind CSS — The Speedrun

"Tailwind is different," Sarah says. "Instead of writing CSS, you compose utility classes."

```jsx
function Card({ title, children }) {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
      <h2 className="text-xl font-bold text-gray-800 mb-4">{title}</h2>
      <div className="text-gray-600">{children}</div>
    </div>
  );
}
```

"Wait," you say. "That's a lot of classes in the HTML."

"It is. But watch how fast you can build."

She builds a button component in under a minute:

```jsx
import clsx from 'clsx';

function Button({ variant, size, disabled, children }) {
  return (
    <button
      className={clsx(
        'rounded font-medium transition-colors',
        {
          'bg-blue-500 text-white hover:bg-blue-600': variant === 'primary',
          'bg-gray-200 text-gray-800 hover:bg-gray-300': variant === 'secondary',
          'px-4 py-2 text-base': size === 'medium',
          'px-6 py-3 text-lg': size === 'large',
          'opacity-50 cursor-not-allowed': disabled,
        }
      )}
      disabled={disabled}
    >
      {children}
    </button>
  );
}
```

"The `clsx` library helps organize conditional classes. And you customize the design tokens in the config:"

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        brand: {
          50: '#f0f9ff',
          500: '#0ea5e9',
          900: '#0c4a6e',
        },
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
    },
  },
};
```

---

## Chapter 4: styled-components — The Dynamic Option

"Sometimes you need styles that respond to props at runtime," Marcus says. "That's where CSS-in-JS shines."

```jsx
import styled from 'styled-components';

const Button = styled.button`
  padding: 8px 16px;
  border-radius: 4px;
  font-weight: 500;
  cursor: pointer;

  /* Dynamic styles based on props */
  background: ${props => props.$primary ? '#0066cc' : '#e0e0e0'};
  color: ${props => props.$primary ? 'white' : 'black'};

  &:hover {
    opacity: 0.9;
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
`;

// Usage
<Button $primary>Primary</Button>
<Button>Secondary</Button>
```

"The `$` prefix is a convention for props that shouldn't hit the DOM," Marcus explains. "This prevents React warnings about invalid DOM attributes and is the recommended way to pass transient props in styled-components."

"And you can theme the whole app:"

```jsx
import { ThemeProvider } from 'styled-components';

const theme = {
  colors: {
    primary: '#0066cc',
    secondary: '#e0e0e0',
    text: '#333',
  },
  spacing: {
    sm: '8px',
    md: '16px',
    lg: '24px',
  },
};

function App() {
  return (
    <ThemeProvider theme={theme}>
      <MyComponent />
    </ThemeProvider>
  );
}

const Card = styled.div`
  padding: ${props => props.theme.spacing.md};
  color: ${props => props.theme.colors.text};
`;
```

---

## Chapter 5: Design Tokens — The Shared Language

"Whichever approach you choose," Sarah says, "you need design tokens. They're the shared vocabulary between designers and developers."

```js
// tokens.js
export const tokens = {
  colors: {
    primary: '#0066cc',
    primaryHover: '#0052a3',
    error: '#d32f2f',
    text: '#1a1a1a',
    textMuted: '#666666',
  },
  spacing: {
    xs: '4px',
    sm: '8px',
    md: '16px',
    lg: '24px',
    xl: '32px',
  },
  borderRadius: {
    sm: '4px',
    md: '8px',
    lg: '16px',
    full: '9999px',
  },
};
```

"Now your buttons, cards, and modals all speak the same language. When the designer says 'primary color', everyone knows exactly what that means."

---

## Chapter 6: Building the Travel Platform

A new project lands: a travel booking feature. Hotels, flights, the works. Perfect chance to apply what you've learned.

"Use Tailwind for the main UI," Sarah suggests. "CSS Modules for reusable components. styled-components for the fancy stuff."

You start with the search bar:

```jsx
function SearchBar({ onSearch }) {
  return (
    <form
      onSubmit={onSearch}
      className="bg-white rounded-2xl shadow-lg p-4 md:p-6 flex flex-col md:flex-row gap-4"
    >
      <div className="flex-1">
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Destination
        </label>
        <div className="relative">
          <MapPinIcon className="absolute left-3 top-1/2 -translate-y-1/2 w-5 h-5 text-gray-400" />
          <input
            type="text"
            placeholder="Where are you going?"
            className="w-full pl-10 pr-4 py-3 border border-gray-300 rounded-lg
                       focus:ring-2 focus:ring-blue-500 focus:border-transparent
                       placeholder:text-gray-400"
          />
        </div>
      </div>

      <div className="flex gap-4">
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-1">Check-in</label>
          <input
            type="date"
            className="px-4 py-3 border border-gray-300 rounded-lg
                       focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          />
        </div>
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-1">Check-out</label>
          <input
            type="date"
            className="px-4 py-3 border border-gray-300 rounded-lg
                       focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          />
        </div>
      </div>

      <button
        type="submit"
        className="bg-blue-600 hover:bg-blue-700 text-white font-semibold
                   px-8 py-3 rounded-lg transition-colors self-end
                   focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
      >
        Search
      </button>
    </form>
  );
}
```

Then the property cards:

```jsx
function PropertyCard({ property }) {
  const { name, location, price, rating, reviewCount, images, amenities, isSuperhost } = property;

  return (
    <article className="group bg-white rounded-xl overflow-hidden shadow-md hover:shadow-xl transition-shadow">
      <div className="relative aspect-[4/3] overflow-hidden">
        <img
          src={images[0]}
          alt={name}
          className="w-full h-full object-cover group-hover:scale-105 transition-transform duration-300"
        />
        {isSuperhost && (
          <span className="absolute top-3 left-3 bg-white px-2 py-1 rounded-md text-xs font-medium">
            Superhost
          </span>
        )}
        <button className="absolute top-3 right-3 p-2 rounded-full bg-white/80 hover:bg-white transition-colors">
          <HeartIcon className="w-5 h-5" />
        </button>
      </div>

      <div className="p-4">
        <div className="flex justify-between items-start mb-2">
          <div>
            <h3 className="font-semibold text-gray-900 line-clamp-1">{name}</h3>
            <p className="text-sm text-gray-500">{location}</p>
          </div>
          <div className="flex items-center gap-1">
            <StarIcon className="w-4 h-4 text-yellow-400 fill-current" />
            <span className="text-sm font-medium">{rating}</span>
            <span className="text-sm text-gray-400">({reviewCount})</span>
          </div>
        </div>

        <div className="flex gap-2 mb-3">
          {amenities.slice(0, 3).map(amenity => (
            <span
              key={amenity}
              className="text-xs bg-gray-100 text-gray-600 px-2 py-1 rounded-full"
            >
              {amenity}
            </span>
          ))}
        </div>

        <p className="text-gray-900">
          <span className="font-bold">${price}</span>
          <span className="text-gray-500"> / night</span>
        </p>
      </div>
    </article>
  );
}
```

---

### The Reusable Button with CSS Modules

For the button component that appears everywhere, you use CSS Modules:

```css
/* Button.module.css */
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  border-radius: 0.5rem;
  transition: all 150ms ease;
}

.primary {
  background-color: var(--color-blue-600);
  color: white;
}
.primary:hover { background-color: var(--color-blue-700); }

.outline {
  border: 2px solid var(--color-gray-300);
  color: var(--color-gray-700);
}
.outline:hover { border-color: var(--color-gray-400); }

.sm { padding: 0.5rem 1rem; font-size: 0.875rem; }
.md { padding: 0.75rem 1.5rem; font-size: 1rem; }
.lg { padding: 1rem 2rem; font-size: 1.125rem; }
```

```jsx
import styles from './Button.module.css';
import clsx from 'clsx';

function Button({ variant = 'primary', size = 'md', children, className, ...props }) {
  return (
    <button
      className={clsx(
        styles.button,
        styles[variant],
        styles[size],
        className
      )}
      {...props}
    >
      {children}
    </button>
  );
}
```

---

### The Fancy Confirmation with styled-components

For the booking confirmation—something special that needs animations—styled-components fits perfectly:

```jsx
import styled from 'styled-components';

const ConfirmationCard = styled.div`
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 1rem;
  padding: 2rem;
  color: white;
  text-align: center;

  @media (min-width: 768px) {
    padding: 3rem;
  }
`;

const CheckmarkCircle = styled.div`
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 1.5rem;

  svg {
    width: 40px;
    height: 40px;
    animation: checkmark 0.5s ease-out forwards;
  }

  @keyframes checkmark {
    0% { transform: scale(0); opacity: 0; }
    50% { transform: scale(1.2); }
    100% { transform: scale(1); opacity: 1; }
  }
`;

const BookingDetails = styled.div`
  background: rgba(255, 255, 255, 0.1);
  border-radius: 0.75rem;
  padding: 1.5rem;
  margin-top: 1.5rem;
  backdrop-filter: blur(10px);
`;
```

"Animations, gradients, pseudo-elements—styled-components handles the complex stuff elegantly," Sarah observes.

---

## Chapter 7: The Pattern Emerges

By Friday, the travel platform is styled and polished. You step back and see the pattern:

- **Tailwind** for rapid layout and utility styling
- **CSS Modules** for reusable component variants
- **styled-components** for dynamic, animated elements
- **Design tokens** tying everything together

"You don't have to pick just one," Marcus says. "You pick the right tool for each job."

Sarah nods. "The consistency comes from the tokens, not from the technology."

---

## Styling Decision Tree

| Situation | Best Choice |
|-----------|-------------|
| Rapid prototyping | Tailwind |
| Reusable component library | CSS Modules |
| Heavy dynamic styling | CSS-in-JS |
| Team knows CSS well | CSS Modules |
| Need design system constraints | Tailwind |

## Common Mistakes

1. **Mixing approaches inconsistently** - Pick a primary method, use others sparingly
2. **Not using design tokens** - Hard to maintain consistency
3. **Styles scattered everywhere** - Keep them organized
4. **Overusing !important** - Fix specificity issues properly


Example:
```jsx
// ❌ Magic numbers and inline styles everywhere
<button style={{ padding: 13, background: '#1E90FF' }}>Save</button>

// ✅ Reuse tokens or classes
<button className="btn btn-primary">Save</button>
```

---

## Debug This Code

Before moving to the exercises, test your debugging skills. Each snippet has bugs - can you spot and fix them?

<details>
<summary>Challenge 1: CSS Modules Import</summary>

```jsx
// Button.jsx
import styles from './Button.module.css';

function Button({ variant, children }) {
  return (
    <button className={styles.button styles.variant}>
      {children}
    </button>
  );
}
```

```css
/* Button.module.css */
.button {
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  border: none;
  cursor: pointer;
}

.primary {
  background: blue;
  color: white;
}

.secondary {
  background: gray;
  color: white;
}
```

**How many bugs can you find?** (Answer: 2 bugs)

<details>
<summary>Hint</summary>

Look at how className is constructed. Is the syntax correct? Does `variant` actually map to a CSS class?

</details>

<details>
<summary>Solution</summary>

**Bug 1**: Missing template literal or string concatenation - `className={styles.button styles.variant}` is invalid syntax.

**Bug 2**: `styles.variant` tries to access a CSS class named `.variant`, but you need to dynamically access `.primary` or `.secondary` based on the prop value.

**Fixed code:**

```jsx
// Button.jsx
import styles from './Button.module.css';

function Button({ variant = 'primary', children }) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  );
}

// Usage
<Button variant="primary">Click me</Button>
<Button variant="secondary">Cancel</Button>
```

**Alternative with clsx library:**

```jsx
import styles from './Button.module.css';
import clsx from 'clsx';

function Button({ variant = 'primary', children }) {
  return (
    <button className={clsx(styles.button, styles[variant])}>
      {children}
    </button>
  );
}
```

**Why it matters**: CSS Modules exports an object where keys are class names. You need proper string concatenation or bracket notation to dynamically access classes.

</details>

</details>

<details>
<summary>Challenge 2: Tailwind Conditional Classes</summary>

```jsx
function Alert({ type, message }) {
  const bgColor = type === 'error' ? 'bg-red-500' : 'bg-blue-500';

  return (
    <div className={bgColor text-white p-4 rounded}>
      {message}
    </div>
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

Look at the className value. Is it a valid JavaScript expression?

</details>

<details>
<summary>Solution</summary>

**Bug**: Missing template literal or quotes - `className={bgColor text-white p-4 rounded}` is invalid syntax. Multiple classes need to be in a string.

**Fixed code:**

```jsx
function Alert({ type, message }) {
  const bgColor = type === 'error' ? 'bg-red-500' : 'bg-blue-500';

  return (
    <div className={`${bgColor} text-white p-4 rounded`}>
      {message}
    </div>
  );
}
```

**Alternative with clsx:**

```jsx
import clsx from 'clsx';

function Alert({ type, message }) {
  return (
    <div className={clsx(
      'text-white p-4 rounded',
      type === 'error' ? 'bg-red-500' : 'bg-blue-500'
    )}>
      {message}
    </div>
  );
}
```

**Why it matters**: className expects a string. When combining dynamic and static classes, you need proper string interpolation.

</details>

</details>

<details>
<summary>Challenge 3: Styled Components Theme Access</summary>

```jsx
import styled from 'styled-components';

const theme = {
  colors: {
    primary: '#0070f3',
    secondary: '#7928ca'
  }
};

const Button = styled.button`
  background: ${theme.colors.primary};
  color: white;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 0.25rem;
  cursor: pointer;
`;

function App() {
  return (
    <ThemeProvider theme={theme}>
      <Button>Click me</Button>
    </ThemeProvider>
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

Styled components access theme via props. Is the theme being accessed correctly?

</details>

<details>
<summary>Solution</summary>

**Bug**: Theme is accessed directly instead of through props - styled components receive theme via `props.theme`, not as a closure variable.

**Fixed code:**

```jsx
import styled, { ThemeProvider } from 'styled-components';

const theme = {
  colors: {
    primary: '#0070f3',
    secondary: '#7928ca'
  }
};

const Button = styled.button`
  background: ${props => props.theme.colors.primary};
  color: white;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 0.25rem;
  cursor: pointer;

  &:hover {
    background: ${props => props.theme.colors.secondary};
  }
`;

function App() {
  return (
    <ThemeProvider theme={theme}>
      <Button>Click me</Button>
    </ThemeProvider>
  );
}
```

**Why it matters**: Styled components inject theme values via props, allowing components to reactively update when the theme changes. Direct variable access breaks this reactivity.

</details>

</details>

---

## Practice Exercises

1. Build a button component with 3 variants using CSS Modules
2. Create a responsive card grid with Tailwind
3. Build a theme switcher (light/dark) with styled-components

### Solutions

<details>
<summary>Exercise 1: Button Component with CSS Modules</summary>

```css
/* Button.module.css */
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  border-radius: 0.5rem;
  border: none;
  cursor: pointer;
  transition: all 0.15s ease;
  font-family: inherit;
}

.button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* Variants */
.primary {
  background-color: #3b82f6;
  color: white;
}

.primary:hover:not(:disabled) {
  background-color: #2563eb;
}

.primary:active:not(:disabled) {
  background-color: #1d4ed8;
}

.secondary {
  background-color: #f3f4f6;
  color: #374151;
  border: 1px solid #d1d5db;
}

.secondary:hover:not(:disabled) {
  background-color: #e5e7eb;
}

.danger {
  background-color: #ef4444;
  color: white;
}

.danger:hover:not(:disabled) {
  background-color: #dc2626;
}

/* Sizes */
.small {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
}

.medium {
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
}

.large {
  padding: 1rem 2rem;
  font-size: 1.125rem;
}

/* Loading state */
.loading {
  position: relative;
  color: transparent;
}

.loading::after {
  content: '';
  position: absolute;
  width: 1rem;
  height: 1rem;
  top: 50%;
  left: 50%;
  margin-left: -0.5rem;
  margin-top: -0.5rem;
  border: 2px solid currentColor;
  border-radius: 50%;
  border-top-color: transparent;
  animation: spin 0.6s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
```

```jsx
// Button.jsx
import styles from './Button.module.css';
import clsx from 'clsx';

function Button({
  variant = 'primary',
  size = 'medium',
  loading = false,
  disabled = false,
  children,
  className,
  ...props
}) {
  return (
    <button
      className={clsx(
        styles.button,
        styles[variant],
        styles[size],
        loading && styles.loading,
        className
      )}
      disabled={disabled || loading}
      {...props}
    >
      {children}
    </button>
  );
}

// Usage examples
function Examples() {
  return (
    <div style={{ display: 'flex', gap: '1rem', flexWrap: 'wrap' }}>
      <Button variant="primary" size="small">Small Primary</Button>
      <Button variant="primary" size="medium">Medium Primary</Button>
      <Button variant="primary" size="large">Large Primary</Button>

      <Button variant="secondary">Secondary</Button>
      <Button variant="danger">Danger</Button>

      <Button loading>Loading...</Button>
      <Button disabled>Disabled</Button>
    </div>
  );
}
```

**Key points:**
- CSS Modules automatically scope class names to avoid collisions
- clsx utility combines class names conditionally
- :not(:disabled) prevents hover effects on disabled buttons
- Loading spinner uses CSS animation instead of external icons
- Spreading ...props allows onClick and other props to pass through

</details>

<details>
<summary>Exercise 2: Responsive Card Grid with Tailwind</summary>

```jsx
function ProductGrid({ products }) {
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8 text-gray-900">
        Our Products
      </h1>

      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
        {products.map(product => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
    </div>
  );
}

function ProductCard({ product }) {
  const { id, name, price, image, category, rating, inStock } = product;

  return (
    <article className="group bg-white rounded-lg shadow-md hover:shadow-xl transition-shadow duration-300 overflow-hidden">
      {/* Image */}
      <div className="relative aspect-square overflow-hidden bg-gray-100">
        <img
          src={image}
          alt={name}
          className="w-full h-full object-cover group-hover:scale-110 transition-transform duration-300"
        />

        {/* Stock badge */}
        {!inStock && (
          <div className="absolute top-2 right-2 bg-red-500 text-white text-xs font-bold px-2 py-1 rounded">
            Out of Stock
          </div>
        )}

        {/* Category badge */}
        <div className="absolute bottom-2 left-2 bg-black/70 text-white text-xs px-2 py-1 rounded">
          {category}
        </div>
      </div>

      {/* Content */}
      <div className="p-4">
        {/* Title and Rating */}
        <div className="mb-2">
          <h3 className="text-lg font-semibold text-gray-900 line-clamp-2 mb-1">
            {name}
          </h3>

          <div className="flex items-center gap-1">
            {[...Array(5)].map((_, i) => (
              <svg
                key={i}
                className={`w-4 h-4 ${
                  i < Math.floor(rating)
                    ? 'text-yellow-400 fill-current'
                    : 'text-gray-300'
                }`}
                viewBox="0 0 20 20"
              >
                <path d="M10 15l-5.878 3.09 1.123-6.545L.489 6.91l6.572-.955L10 0l2.939 5.955 6.572.955-4.756 4.635 1.123 6.545z" />
              </svg>
            ))}
            <span className="text-sm text-gray-600 ml-1">
              ({rating.toFixed(1)})
            </span>
          </div>
        </div>

        {/* Price and Button */}
        <div className="flex items-center justify-between mt-4">
          <span className="text-2xl font-bold text-gray-900">
            ${price.toFixed(2)}
          </span>

          <button
            disabled={!inStock}
            className={`
              px-4 py-2 rounded-lg font-medium transition-colors
              ${inStock
                ? 'bg-blue-600 hover:bg-blue-700 text-white'
                : 'bg-gray-300 text-gray-500 cursor-not-allowed'
              }
            `}
          >
            {inStock ? 'Add to Cart' : 'Sold Out'}
          </button>
        </div>
      </div>
    </article>
  );
}

// Example usage
function App() {
  const products = [
    {
      id: 1,
      name: 'Wireless Headphones',
      price: 99.99,
      image: '/images/headphones.jpg',
      category: 'Electronics',
      rating: 4.5,
      inStock: true,
    },
    {
      id: 2,
      name: 'Laptop Stand',
      price: 49.99,
      image: '/images/stand.jpg',
      category: 'Accessories',
      rating: 4.8,
      inStock: false,
    },
    // More products...
  ];

  return <ProductGrid products={products} />;
}
```

**Key points:**
- Responsive grid uses Tailwind's breakpoint prefixes (sm:, lg:, xl:)
- aspect-square maintains 1:1 ratio for images
- group and group-hover enable parent-child hover effects
- line-clamp-2 truncates text to 2 lines with ellipsis
- Conditional classes handle stock status dynamically

</details>

<details>
<summary>Exercise 3: Theme Switcher with styled-components</summary>

```jsx
import { createContext, useContext, useState, useEffect } from 'react';
import styled, { ThemeProvider, createGlobalStyle } from 'styled-components';

// Theme definitions
const lightTheme = {
  name: 'light',
  colors: {
    background: '#ffffff',
    surface: '#f3f4f6',
    text: '#1f2937',
    textMuted: '#6b7280',
    primary: '#3b82f6',
    primaryHover: '#2563eb',
    border: '#e5e7eb',
  },
};

const darkTheme = {
  name: 'dark',
  colors: {
    background: '#111827',
    surface: '#1f2937',
    text: '#f9fafb',
    textMuted: '#9ca3af',
    primary: '#60a5fa',
    primaryHover: '#3b82f6',
    border: '#374151',
  },
};

// Theme context
const ThemeContext = createContext();

export function ThemeContextProvider({ children }) {
  const [theme, setTheme] = useState('light');

  // Load saved theme from localStorage
  useEffect(() => {
    const saved = localStorage.getItem('theme');
    if (saved) {
      setTheme(saved);
    }
  }, []);

  // Save theme to localStorage
  useEffect(() => {
    localStorage.setItem('theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme(prev => (prev === 'light' ? 'dark' : 'light'));
  };

  const currentTheme = theme === 'light' ? lightTheme : darkTheme;

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <ThemeProvider theme={currentTheme}>
        {children}
      </ThemeProvider>
    </ThemeContext.Provider>
  );
}

export const useTheme = () => useContext(ThemeContext);

// Global styles
const GlobalStyle = createGlobalStyle`
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    background-color: ${props => props.theme.colors.background};
    color: ${props => props.theme.colors.text};
    transition: background-color 0.3s ease, color 0.3s ease;
  }
`;

// Styled components
const Container = styled.div`
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
`;

const Card = styled.div`
  background-color: ${props => props.theme.colors.surface};
  border: 1px solid ${props => props.theme.colors.border};
  border-radius: 0.5rem;
  padding: 1.5rem;
  margin-bottom: 1rem;
  transition: all 0.3s ease;

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  }
`;

const Title = styled.h1`
  font-size: 2rem;
  margin-bottom: 1rem;
  color: ${props => props.theme.colors.text};
`;

const Text = styled.p`
  color: ${props => props.theme.colors.textMuted};
  margin-bottom: 1rem;
  line-height: 1.6;
`;

const Button = styled.button`
  background-color: ${props => props.theme.colors.primary};
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 0.5rem;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s ease;

  &:hover {
    background-color: ${props => props.theme.colors.primaryHover};
  }
`;

const ThemeToggle = styled.button`
  position: fixed;
  top: 1rem;
  right: 1rem;
  background-color: ${props => props.theme.colors.surface};
  border: 1px solid ${props => props.theme.colors.border};
  border-radius: 50%;
  width: 48px;
  height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 1.5rem;

  &:hover {
    transform: scale(1.1);
  }
`;

// App component
function App() {
  const { theme, toggleTheme } = useTheme();

  return (
    <>
      <GlobalStyle />
      <ThemeToggle onClick={toggleTheme} aria-label="Toggle theme">
        {theme === 'light' ? '🌙' : '☀️'}
      </ThemeToggle>

      <Container>
        <Title>Theme Switcher Demo</Title>

        <Card>
          <h2 style={{ marginBottom: '0.5rem' }}>Welcome!</h2>
          <Text>
            This is a demo of theme switching with styled-components.
            Click the button in the top-right to toggle between light and dark modes.
          </Text>
          <Button>Get Started</Button>
        </Card>

        <Card>
          <h2 style={{ marginBottom: '0.5rem' }}>Features</h2>
          <Text>
            The theme preference is saved to localStorage, so it persists
            across page reloads. All colors smoothly transition when switching themes.
          </Text>
        </Card>
      </Container>
    </>
  );
}

// Root component with provider
export default function Root() {
  return (
    <ThemeContextProvider>
      <App />
    </ThemeContextProvider>
  );
}
```

**Key points:**
- Theme object contains all design tokens in one place
- ThemeProvider makes theme available to all styled components
- createGlobalStyle applies theme to body and global elements
- localStorage persists user's theme choice
- CSS transitions create smooth theme switching animations
- Fixed toggle button uses semantic emoji for clear affordance

</details>

---

## What You Learned

This module covered:

- **CSS Modules**: Automatic scoping prevents naming collisions with zero runtime cost
- **Tailwind CSS**: Utility-first classes enable rapid prototyping with design constraints
- **styled-components**: Dynamic styling with props and theming via JavaScript
- **Design Tokens**: Shared color, spacing, and typography values create consistency
- **Choosing Approaches**: Different tools excel at different tasks - mix them strategically

**Key takeaway**: No single styling approach is best - choose based on team familiarity, project needs, and whether you prioritize speed, flexibility, or type safety.

---

## Real-World Application

This week at work, you might use these concepts to:

- Build a component library with CSS Modules for predictable, scoped styles
- Rapidly prototype new features with Tailwind's utility classes
- Implement light/dark mode theming using styled-components ThemeProvider
- Create a design system with shared tokens referenced across all styling methods
- Refactor inconsistent styles into a cohesive system using design tokens

---

## Further Reading

- [CSS Modules Documentation](https://github.com/css-modules/css-modules)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [styled-components Documentation](https://styled-components.com/docs)

---

**Navigation**: [← Previous Module](./05-forms-and-validation.md) | [Next Module →](./07-testing-fundamentals.md)
