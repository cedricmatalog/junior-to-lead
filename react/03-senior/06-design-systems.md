# Design Systems

> **Last reviewed**: 2026-01-06


## Week 26: The Shared Language

Multiple teams are shipping UI, but the product looks inconsistent. Sarah wants a design system that can scale across products without slowing delivery. Marcus points out that a system is only useful if it has clear APIs, reliable tokens, and documentation people actually read. This week is about building a design system that is easy to adopt, hard to misuse, and simple to evolve.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Design system** | A style guide - shared rules and reusable pieces |
| **Tokens** | A palette - consistent values across the system |
| **Component API** | A contract - how consumers should use the component |
| **Versioning** | A release train - predictable updates |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 25 (Accessibility) - Familiarity with inclusive component design.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Structure a component library for reusability
- [ ] Define design tokens for consistent styling
- [ ] Design component APIs that scale across teams
- [ ] Document components with Storybook
- [ ] Provide usage guidelines and accessibility notes
- [ ] Version and distribute design system packages

---

## Time Estimate

- **Reading**: 70-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice design system maintenance over 6-8 weeks

---

## Chapter 1: Design System Structure

You need a repository layout that works for both designers and engineers.

```
design-system/
├── packages/
│   ├── tokens/              # Design tokens
│   │   ├── colors.ts
│   │   ├── spacing.ts
│   │   ├── typography.ts
│   │   └── index.ts
│   ├── components/          # UI components
│   │   ├── Button/
│   │   ├── Input/
│   │   ├── Modal/
│   │   └── index.ts
│   └── icons/               # Icon library
├── apps/
│   └── docs/                # Storybook / documentation
└── package.json
```

## Chapter 2: Design Tokens

Tokens keep styles consistent even when products diverge.

### Token Definition

```ts
// packages/tokens/colors.ts
export const colors = {
  // Primitives (don't use directly in components)
  blue: {
    50: '#eff6ff',
    100: '#dbeafe',
    500: '#3b82f6',
    600: '#2563eb',
    900: '#1e3a8a',
  },

  // Semantic tokens (use these)
  primary: {
    default: '#3b82f6',
    hover: '#2563eb',
    active: '#1d4ed8',
  },
  text: {
    primary: '#1a1a1a',
    secondary: '#6b7280',
    inverse: '#ffffff',
  },
  background: {
    default: '#ffffff',
    subtle: '#f9fafb',
    muted: '#f3f4f6',
  },
  border: {
    default: '#e5e7eb',
    focus: '#3b82f6',
  },
  status: {
    error: '#ef4444',
    success: '#22c55e',
    warning: '#f59e0b',
  },
} as const;

// packages/tokens/spacing.ts
export const spacing = {
  0: '0',
  1: '4px',
  2: '8px',
  3: '12px',
  4: '16px',
  5: '20px',
  6: '24px',
  8: '32px',
  10: '40px',
  12: '48px',
  16: '64px',
} as const;

// packages/tokens/typography.ts
export const typography = {
  fontFamily: {
    sans: 'Inter, system-ui, sans-serif',
    mono: 'JetBrains Mono, monospace',
  },
  fontSize: {
    xs: '12px',
    sm: '14px',
    base: '16px',
    lg: '18px',
    xl: '20px',
    '2xl': '24px',
    '3xl': '30px',
  },
  fontWeight: {
    normal: 400,
    medium: 500,
    semibold: 600,
    bold: 700,
  },
} as const;
```

### Token Usage

```tsx
import { colors, spacing, typography } from '@myorg/tokens';
import styled from 'styled-components';

const Button = styled.button`
  font-family: ${typography.fontFamily.sans};
  font-size: ${typography.fontSize.sm};
  font-weight: ${typography.fontWeight.medium};
  padding: ${spacing[2]} ${spacing[4]};
  background: ${colors.primary.default};
  color: ${colors.text.inverse};
  border-radius: 6px;

  &:hover {
    background: ${colors.primary.hover};
  }
`;
```

## Chapter 3: Component API Design

APIs should make the right usage obvious.

### Consistent Props

```tsx
interface CommonProps {
  className?: string;
  children?: React.ReactNode;
}

interface SizeProps {
  size?: 'sm' | 'md' | 'lg';
}

interface VariantProps {
  variant?: 'primary' | 'secondary' | 'ghost';
}

// Button uses common patterns
interface ButtonProps extends CommonProps, SizeProps, VariantProps {
  disabled?: boolean;
  loading?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
  onClick?: () => void;
}

export function Button({
  variant = 'primary',
  size = 'md',
  disabled,
  loading,
  leftIcon,
  rightIcon,
  children,
  ...props
}: ButtonProps) {
  return (
    <button
      disabled={disabled || loading}
      data-variant={variant}
      data-size={size}
      {...props}
    >
      {loading ? <Spinner /> : leftIcon}
      {children}
      {rightIcon}
    </button>
  );
}
```

### Compound Components

```tsx
// Card.tsx
const CardContext = createContext<{ variant: string }>({ variant: 'default' });

function Card({ variant = 'default', children }: CardProps) {
  return (
    <CardContext.Provider value={{ variant }}>
      <div className="card" data-variant={variant}>
        {children}
      </div>
    </CardContext.Provider>
  );
}

function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className="card-header">{children}</div>;
}

function CardBody({ children }: { children: React.ReactNode }) {
  return <div className="card-body">{children}</div>;
}

function CardFooter({ children }: { children: React.ReactNode }) {
  return <div className="card-footer">{children}</div>;
}

// Attach subcomponents
Card.Header = CardHeader;
Card.Body = CardBody;
Card.Footer = CardFooter;

// Usage
<Card variant="elevated">
  <Card.Header>Title</Card.Header>
  <Card.Body>Content</Card.Body>
  <Card.Footer>Actions</Card.Footer>
</Card>
```

## Chapter 4: Storybook Documentation

Documentation is how the system actually gets adopted.

### Story Structure

```tsx
// Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'ghost'],
    },
    size: {
      control: 'radio',
      options: ['sm', 'md', 'lg'],
    },
  },
};

export default meta;
type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: {
    children: 'Button',
    variant: 'primary',
  },
};

export const Secondary: Story = {
  args: {
    children: 'Button',
    variant: 'secondary',
  },
};

export const AllVariants: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: 16 }}>
      <Button variant="primary">Primary</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="ghost">Ghost</Button>
    </div>
  ),
};
```

### Documentation Pages

```mdx
// Button.mdx
import { Canvas, Meta, Story, Controls } from '@storybook/blocks';
import * as ButtonStories from './Button.stories';

<Meta of={ButtonStories} />

# Button

Buttons trigger actions or navigation.

## Chapter 5: Usage Guidelines

Guidelines keep the system consistent across teams.

```tsx
import { Button } from '@myorg/components';

<Button variant="primary" onClick={handleClick}>
  Click this
</Button>
```

### Props

<Controls />

### Variants

<Canvas of={ButtonStories.AllVariants} />

### Accessibility

- Use clear, descriptive labels
- Include loading state announcement
- Ensure sufficient color contrast
```

## Chapter 6: Versioning and Publishing

Releases need to be predictable, even with many consumers.

### Semantic Versioning

```json
// package.json
{
  "name": "@myorg/components",
  "version": "2.1.0"
}
```

- **Major (2.x.x)**: Breaking changes
- **Minor (x.1.x)**: New features, backwards compatible
- **Patch (x.x.1)**: Bug fixes

### Changesets for Versioning

```bash
# Create changeset
npx changeset

# Version packages
npx changeset version

# Publish
npx changeset publish
```

### Package Distribution

```json
// package.json
{
  "name": "@myorg/components",
  "main": "./dist/index.js",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.js",
      "types": "./dist/index.d.ts"
    },
    "./Button": {
      "import": "./dist/Button/index.mjs",
      "require": "./dist/Button/index.js"
    }
  },
  "sideEffects": false
}
```

---

## Common Mistakes

1. **Shipping components without guidelines** - Adoption stalls without clear usage rules.
2. **Token drift** - Hardcoded values undermine consistency.
3. **Overly complex APIs** - If components are hard to use, teams will reimplement.
4. **Ignoring versioning** - Breaking changes spread silently.


Example:
```tsx
// ❌ Ad-hoc colors per component
const Button = () => <button style={{ color: '#4F46E5' }}>Save</button>;

// ✅ Use design tokens
const Button = () => <button className="text-primary">Save</button>;
```

## Practice Exercises

1. Create a design token system with theme support
2. Build a component with comprehensive Storybook docs
3. Set up versioning with changesets

### Solutions

<details>
<summary>Exercise 1: Token System</summary>

```ts
// tokens/colors.ts
export const colors = {
  brand: {
    500: '#1e40af',
    600: '#1e3a8a'
  },
  neutral: {
    100: '#f5f5f5',
    900: '#111827'
  }
};

// Semantic tokens map to primitives
export const semantic = {
  light: {
    surface: colors.neutral[100],
    text: colors.neutral[900],
    primary: colors.brand[500]
  },
  dark: {
    surface: colors.neutral[900],
    text: colors.neutral[100],
    primary: colors.brand[600]
  }
};
```

**Key points:**
- Tokens are centralized and reused.
- Semantic tokens map design intent to primitives.
- Themes swap token values, not component code.
- Consumers reference tokens, not raw hex values.

</details>

<details>
<summary>Exercise 2: Storybook Docs</summary>

```tsx
// Button.stories.tsx
export default {
  title: 'Components/Button',
  component: Button,
  args: { variant: 'primary', children: 'Save' },
  parameters: {
    a11y: { disable: false }
  }
};

export const Primary = {};
export const Secondary = { args: { variant: 'secondary' } };
```

**Key points:**
- Stories show common variants and states.
- Args allow interactive testing.
- A11y checks run alongside docs.

</details>

<details>
<summary>Exercise 3: Changesets</summary>

```bash
npx changeset
npx changeset version
npx changeset publish
```

**Key points:**
- Each change creates a version note.
- Versioning stays consistent across packages.
- Publishing supports tags (e.g. next) for pre-releases.

</details>

---

## What You Learned

This module covered:

- **Structure**: A scalable repo layout for design systems
- **Tokens**: Single source of truth for styling
- **Component APIs**: Consistent, reusable interfaces
- **Documentation**: Storybook as the adoption engine
- **Versioning**: Predictable releases for consumers

**Key takeaway**: A design system is a product with users, not just a component folder.

---

## Real-World Application

This week at work, you might use these concepts to:

- Move shared styles into a token package
- Document components with Storybook stories
- Define accessibility guidelines per component
- Publish a versioned UI package for internal teams
- Standardize component APIs across products

---

## Further Reading

- [Storybook Documentation](https://storybook.js.org/docs)
- [Design Tokens W3C](https://design-tokens.github.io/community-group/)

---

**Navigation**: [<- Previous Module](./05-accessibility-a11y.md) | [Next Module ->](./07-observability-monitoring.md)
