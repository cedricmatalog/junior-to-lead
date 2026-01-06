# Code Quality

> **Last reviewed**: 2026-01-06


## Week 19: The Maintainability Check

The team is moving fast, but the codebase is getting harder to review. Sarah wants consistent tooling and fewer style debates. Marcus points out the last refactor took twice as long because the file structure was unclear. This week is about creating guardrails: linting, formatting, project structure, and review habits that keep quality high as the team grows.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Linting** | Spellcheck - catch mistakes before review |
| **Formatting** | Dress code - consistent appearance |
| **Refactoring** | Renovation - improve structure without changing purpose |
| **Code review** | Gate check - quality before shipping |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 08 (TypeScript Mastery) - Familiarity with project tooling and component structure.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Configure ESLint and Prettier for consistent style
- [ ] Organize project structure for scalability
- [ ] Apply refactoring techniques safely
- [ ] Use code review checklists to reduce defects
- [ ] Add pre-commit hooks for quality enforcement
- [ ] Define team conventions that stick

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice code-quality habits over 4-6 weeks

---

## Chapter 1: ESLint Configuration

You need the tooling to catch issues before they reach review.

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

## Chapter 2: Prettier Configuration

Formatting debates slow teams down. Prettier eliminates them.

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

## Chapter 3: Project Structure

A clear layout makes features easy to find and own.

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

## Chapter 4: Code Organization Patterns

Structure patterns keep components predictable as the app grows.

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

## Chapter 5: Refactoring Techniques

You want improvements without introducing regressions.

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

## Chapter 6: Code Review Checklist

Reviews need consistency, not personal preferences.

- [ ] No commented-out code
- [ ] No console.logs in production code
- [ ] Proper error handling
- [ ] Accessible (proper ARIA, keyboard support)
- [ ] TypeScript types are specific (not `any`)
- [ ] Tests cover important behavior
- [ ] No duplicate code
- [ ] Clear naming
- [ ] Consistent with existing patterns

## Chapter 7: Pre-commit Hooks

Automation keeps quality consistent even when teams are busy.

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

---

## Common Mistakes

1. **Linting without fixing** - Warnings ignored become permanent debt.
2. **Inconsistent structure** - Teams waste time finding the right file.
3. **Refactoring without tests** - Improvements become regressions.
4. **Skipping automation** - Manual checks never scale.


Example:
```jsx
// ❌ Disable the rule instead of fixing the issue
/* eslint-disable-next-line react-hooks/exhaustive-deps */

// ✅ Fix dependencies explicitly
useEffect(() => {
  doWork(userId);
}, [userId]);
```

## Practice Exercises

1. Set up ESLint + Prettier for a new project
2. Refactor a large component into smaller pieces
3. Create a pre-commit hook that runs tests

### Solutions

<details>
<summary>Exercise 1: ESLint + Prettier</summary>

```js
// .eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:jsx-a11y/recommended',
    'prettier'
  ],
  plugins: ['react', 'react-hooks', '@typescript-eslint', 'jsx-a11y'],
  rules: {
    'no-console': ['warn', { allow: ['warn', 'error'] }]
  },
  settings: {
    react: {
      version: 'detect'
    }
  },
  overrides: [
    {
      files: ['**/*.test.*', '**/*.spec.*'],
      rules: {
        '@typescript-eslint/no-explicit-any': 'off'
      }
    }
  }
};
```

```json
// .prettierrc
{
  "singleQuote": true,
  "semi": true,
  "printWidth": 100,
  "trailingComma": "all"
}
```

**Key points:**
- Prettier should be last in ESLint extends.
- Keep rules minimal to reduce noise.
- Enforce consistency across the team.

</details>

<details>
<summary>Exercise 2: Refactor a Component</summary>

```jsx
function ProductPage({ product }) {
  return (
    <section>
      <ProductHeader title={product.name} sku={product.sku} />
      <ProductMeta price={product.price} rating={product.rating} />
      <ProductActions
        productId={product.id}
        inStock={product.inStock}
        onAddToCart={() => addToCart(product.id)}
      />
    </section>
  );
}
```

**Key points:**
- Split by responsibility (header, meta, actions).
- Each child component is easier to test.
- The parent stays readable.

</details>

<details>
<summary>Exercise 3: Pre-commit Hook</summary>

```bash
npx husky-init && npm install
npm install --save-dev lint-staged
```

```json
// package.json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "npm test -- --watch=false"
    ]
  }
}
```

```bash
# .husky/pre-commit
npm run lint-staged
```

**Key points:**
- Only staged files are checked.
- Tests run before code enters the repo.
- Auto-fixers reduce manual churn.

</details>

---

## What You Learned

This module covered:

- **Linting**: Automated checks for code correctness
- **Formatting**: Consistent styling with Prettier
- **Structure**: Organizing code for scalability
- **Refactoring**: Improving code safely
- **Review discipline**: Checklists that reduce defects

**Key takeaway**: Quality is a system, not a one-time cleanup.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add linting and formatting to a greenfield repo
- Standardize file structure across teams
- Refactor a long component into clear sections
- Create a review checklist for PRs
- Add pre-commit hooks to prevent broken code

---

## Further Reading

- [ESLint Rules](https://eslint.org/docs/rules/)
- [Prettier Options](https://prettier.io/docs/en/options.html)

---

**Navigation**: [← Previous Module](./08-typescript-mastery.md) | [Next Module →](./10-nextjs-fundamentals.md)
