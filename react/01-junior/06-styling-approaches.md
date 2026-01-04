# Styling Approaches

Choose and implement styling solutions that scale with your application.

## Learning Objectives

By the end of this module, you will:
- Understand trade-offs between CSS approaches in React
- Use CSS Modules for scoped styling
- Apply Tailwind CSS utility-first patterns
- Implement CSS-in-JS with styled-components

## Styling Options Comparison

| Approach | Scoping | DX | Performance | Learning Curve |
|----------|---------|-----|-------------|----------------|
| CSS Modules | Automatic | Good | Excellent | Low |
| Tailwind CSS | Utility classes | Excellent | Excellent | Medium |
| styled-components | Automatic | Good | Good | Medium |
| Plain CSS | Manual (BEM) | Basic | Excellent | Low |

## CSS Modules

Locally scoped CSS - each class name is unique at build time.

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

**Benefits:**
- No naming collisions
- Regular CSS (easy to learn)
- Zero runtime cost
- Works with CSS features (variables, media queries)

## Tailwind CSS

Utility-first CSS framework with predefined classes.

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

**Organizing complex class strings:**

```jsx
// Using clsx or classnames library
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

**Tailwind config for design tokens:**

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

## styled-components

CSS-in-JS with full CSS power and dynamic styling.

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

**Theming with styled-components:**

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

## Design Tokens

Centralize design decisions regardless of styling approach.

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

## Common Mistakes

1. **Mixing approaches inconsistently** - Pick one primary method
2. **Not using design tokens** - Hard to maintain consistency
3. **Styling in component files** - Keep styles organized
4. **Overusing !important** - Fix specificity issues properly

## Practice Exercises

1. Build a button component with 3 variants using CSS Modules
2. Create a responsive card grid with Tailwind
3. Build a theme switcher (light/dark) with styled-components

## Further Reading

- [CSS Modules Documentation](https://github.com/css-modules/css-modules)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [styled-components Documentation](https://styled-components.com/docs)
