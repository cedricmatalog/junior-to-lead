# Styling Approaches

## Week Six: The Design System

The job application portal is complete. But when you open it in the browser, something feels off.

"It works," Marcus says, squinting at the screen. "But it looks like it was designed by an engineer."

He's not wrong. The buttons don't match. The spacing is inconsistent. Every component has its own shade of blue.

"We need a design system," Sarah says. "And that means choosing a styling approach. Today, you pick your weapon."

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **CSS Modules** | Labeled storage bins - each component gets its own, no name collisions |
| **Tailwind CSS** | Pre-made building blocks - combine existing pieces quickly |
| **CSS-in-JS** | Custom furniture - styles designed specifically for each component |

Keep these in mind. They'll click as we build.

---

## Chapter 1: The Options on the Table

Sarah pulls up a comparison chart:

| Approach | Scoping | DX | Performance | Learning Curve |
|----------|---------|-----|-------------|----------------|
| CSS Modules | Automatic | Good | Excellent | Low |
| Tailwind CSS | Utility classes | Excellent | Excellent | Medium |
| styled-components | Automatic | Good | Good | Medium |
| Plain CSS | Manual (BEM) | Basic | Excellent | Low |

"Each has trade-offs," she explains. "Let me show you what they look like in practice."

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

"The `$` prefix is a convention for props that shouldn't hit the DOM. And you can theme the whole app:"

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

## Chapter 7: The Reusable Button with CSS Modules

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

## Chapter 8: The Fancy Confirmation with styled-components

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

## Chapter 9: The Pattern Emerges

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

## Practice Exercises

1. Build a button component with 3 variants using CSS Modules
2. Create a responsive card grid with Tailwind
3. Build a theme switcher (light/dark) with styled-components

## Further Reading

- [CSS Modules Documentation](https://github.com/css-modules/css-modules)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [styled-components Documentation](https://styled-components.com/docs)

---

**Navigation**: [← Previous Module](./05-forms-and-validation.md) | [Next Module →](./07-testing-fundamentals.md)
