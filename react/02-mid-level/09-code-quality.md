# Code Quality

Maintain high standards through tooling, patterns, and practices.

## Learning Objectives

By the end of this module, you will:
- Configure ESLint and Prettier for consistency
- Organize code for maintainability
- Apply refactoring techniques
- Establish team conventions

## ESLint Configuration

```js
// .eslintrc.js
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:@typescript-eslint/recommended',
    'prettier', // Must be last
  ],
  plugins: ['react', 'react-hooks', '@typescript-eslint'],
  rules: {
    // React specific
    'react/prop-types': 'off', // Using TypeScript
    'react/react-in-jsx-scope': 'off', // React 17+
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',

    // TypeScript
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-unused-vars': ['warn', { argsIgnorePattern: '^_' }],

    // Code quality
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'prefer-const': 'error',
    'no-nested-ternary': 'warn',
  },
  settings: {
    react: {
      version: 'detect',
    },
  },
};
```

## Prettier Configuration

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "arrowParens": "avoid"
}
```

```json
// package.json scripts
{
  "scripts": {
    "lint": "eslint src --ext .ts,.tsx",
    "lint:fix": "eslint src --ext .ts,.tsx --fix",
    "format": "prettier --write src/**/*.{ts,tsx,css}",
    "format:check": "prettier --check src/**/*.{ts,tsx,css}"
  }
}
```

## Project Structure

```
src/
├── components/           # Shared/reusable components
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   ├── Button.module.css
│   │   └── index.ts
│   └── index.ts         # Barrel export
├── features/            # Feature-based modules
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── api.ts
│   │   ├── types.ts
│   │   └── index.ts
│   └── users/
├── hooks/               # Shared hooks
├── utils/               # Utility functions
├── types/               # Global types
├── api/                 # API client
└── App.tsx
```

**Principles:**
- Co-locate related code (tests, styles with components)
- Feature folders contain everything for that feature
- Shared code in dedicated directories
- Flat is better than nested (until it isn't)

## Code Organization Patterns

### Component File Structure

```tsx
// Button.tsx
import { useState, useCallback } from 'react';
import styles from './Button.module.css';
import type { ButtonProps } from './types';

// 1. Types (or import from types.ts)
interface InternalState {
  isPressed: boolean;
}

// 2. Constants
const ANIMATION_DURATION = 200;

// 3. Helper functions (outside component)
function getButtonClassName(variant: string, isPressed: boolean) {
  return `${styles.button} ${styles[variant]} ${isPressed ? styles.pressed : ''}`;
}

// 4. Component
export function Button({ variant = 'primary', children, onClick }: ButtonProps) {
  // State
  const [isPressed, setIsPressed] = useState(false);

  // Derived values
  const className = getButtonClassName(variant, isPressed);

  // Callbacks
  const handleClick = useCallback((e: React.MouseEvent) => {
    setIsPressed(true);
    onClick?.(e);
    setTimeout(() => setIsPressed(false), ANIMATION_DURATION);
  }, [onClick]);

  // Render
  return (
    <button className={className} onClick={handleClick}>
      {children}
    </button>
  );
}
```

### Barrel Exports

```tsx
// components/index.ts
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';

// Usage
import { Button, Input, Modal } from '@/components';
```

## Refactoring Techniques

### Extract Component

```tsx
// Before: Large component
function UserDashboard({ user }) {
  return (
    <div>
      <header>
        <img src={user.avatar} />
        <h1>{user.name}</h1>
        <p>{user.bio}</p>
      </header>
      {/* ... lots more JSX */}
    </div>
  );
}

// After: Extracted component
function UserHeader({ user }) {
  return (
    <header>
      <img src={user.avatar} />
      <h1>{user.name}</h1>
      <p>{user.bio}</p>
    </header>
  );
}

function UserDashboard({ user }) {
  return (
    <div>
      <UserHeader user={user} />
      {/* ... */}
    </div>
  );
}
```

### Extract Hook

```tsx
// Before: Logic mixed with UI
function ProductPage({ id }) {
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/products/${id}`)
      .then(r => r.json())
      .then(setProduct)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [id]);

  // ... render
}

// After: Logic in custom hook
function useProduct(id) {
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // ... same logic
  }, [id]);

  return { product, loading, error };
}

function ProductPage({ id }) {
  const { product, loading, error } = useProduct(id);
  // ... render
}
```

## Code Review Checklist

- [ ] No commented-out code
- [ ] No console.logs in production code
- [ ] Proper error handling
- [ ] Accessible (proper ARIA, keyboard support)
- [ ] TypeScript types are specific (not `any`)
- [ ] Tests cover important behavior
- [ ] No duplicate code
- [ ] Clear naming
- [ ] Consistent with existing patterns

## Pre-commit Hooks

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{css,json,md}": ["prettier --write"]
  }
}
```

```bash
# Install husky
npx husky-init && npm install

# .husky/pre-commit
npm run lint-staged
```

## Practice Exercises

1. Set up ESLint + Prettier for a new project
2. Refactor a large component into smaller pieces
3. Create a pre-commit hook that runs tests

## Further Reading

- [ESLint Rules](https://eslint.org/docs/rules/)
- [Prettier Options](https://prettier.io/docs/en/options.html)
