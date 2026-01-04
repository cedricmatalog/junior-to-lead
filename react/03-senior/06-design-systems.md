# Design Systems

Build and maintain component libraries that scale across teams and products.

## Learning Objectives

By the end of this module, you will:
- Structure a component library for reusability
- Use design tokens for consistency
- Document components with Storybook
- Version and distribute packages

## Design System Structure

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

## Design Tokens

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

## Component API Design

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

## Storybook Documentation

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

## Usage

```tsx
import { Button } from '@myorg/components';

<Button variant="primary" onClick={handleClick}>
  Click me
</Button>
```

## Props

<Controls />

## Variants

<Canvas of={ButtonStories.AllVariants} />

## Accessibility

- Use clear, descriptive labels
- Include loading state announcement
- Ensure sufficient color contrast
```

## Versioning and Publishing

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

## Practice Exercises

1. Create a design token system with theme support
2. Build a component with comprehensive Storybook docs
3. Set up versioning with changesets

## Further Reading

- [Storybook Documentation](https://storybook.js.org/docs)
- [Design Tokens W3C](https://design-tokens.github.io/community-group/)
