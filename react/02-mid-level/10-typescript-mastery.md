# TypeScript Mastery

> **Last reviewed**: 2026-01-06


## Week Ten: The Type Safety Push

The codebase is getting bigger, and bugs are harder to spot in review. Sarah wants more confidence before refactors, and Marcus points out that TypeScript can catch most of the mistakes if you model types well. This week is about using TypeScript as a design tool -- not just for annotations -- so your components are safer, easier to reuse, and harder to misuse.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Generics** | A template - one shape, many versions |
| **Utility types** | Power tools - reshape types quickly |
| **Discriminated unions** | Labeled boxes - pick the right shape |
| **Type guards** | Bouncers - check who gets in |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Mid-Level Module 01 (Advanced Patterns) - Familiarity with component composition and reusable patterns.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Build generic components and hooks with strong typing
- [ ] Use utility types to model real data shapes
- [ ] Apply discriminated unions for complex UI states
- [ ] Write type guards to narrow unsafe data
- [ ] Design prop types that prevent misuse
- [ ] Configure strict TypeScript settings for safety

---

## Time Estimate

- **Reading**: 60-90 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice TypeScript patterns over 4-6 weeks

---

## Chapter 1: Generics in Components

You need reusable components that still keep strict types.

"Reuse without losing safety," Sarah says.

```tsx
// Generic list component
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map(item => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

// Usage - type is inferred
<List
  items={users}
  renderItem={user => <span>{user.name}</span>}
  keyExtractor={user => user.id}
/>

// Generic select component
interface SelectProps<T> {
  options: T[];
  value: T | null;
  onChange: (value: T) => void;
  getLabel: (option: T) => string;
  getValue: (option: T) => string;
}

function Select<T>({
  options,
  value,
  onChange,
  getLabel,
  getValue,
}: SelectProps<T>) {
  return (
    <select
      value={value ? getValue(value) : ''}
      onChange={e => {
        const selected = options.find(o => getValue(o) === e.target.value);
        if (selected) onChange(selected);
      }}
    >
      {options.map(option => (
        <option key={getValue(option)} value={getValue(option)}>
          {getLabel(option)}
        </option>
      ))}
    </select>
  );
}
```

## Chapter 2: Utility Types

You can reshape data types without rewriting them from scratch.

"Utility types keep you honest and fast," Marcus says.

```tsx
// Partial - make all properties optional
interface User {
  id: string;
  name: string;
  email: string;
}

function updateUser(id: string, updates: Partial<User>) {
  // updates can have any subset of User properties
}

// Required - make all properties required
type RequiredUser = Required<User>;

// Pick - select specific properties
type UserPreview = Pick<User, 'id' | 'name'>;

// Omit - exclude specific properties
type CreateUserInput = Omit<User, 'id'>;

// Record - create object type
type UserRoles = Record<string, 'admin' | 'user' | 'guest'>;

// ReturnType - extract function return type
function fetchUser() {
  return { id: '1', name: 'Alice' };
}
type FetchUserResult = ReturnType<typeof fetchUser>;

// Parameters - extract function parameters
type FetchUserParams = Parameters<typeof fetchUser>;

// Awaited - unwrap Promise type
type UserData = Awaited<ReturnType<typeof fetchUserAsync>>;
```

## Chapter 3: Discriminated Unions

Complex UI states need explicit, typed labels.

"Make the states explicit so bugs have nowhere to hide," Sarah says.

```tsx
// State machine with discriminated unions
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

function UserProfile() {
  const [state, setState] = useState<RequestState<User>>({ status: 'idle' });

  // TypeScript narrows the type based on status
  switch (state.status) {
    case 'idle':
      return <button onClick={fetchUser}>Load</button>;
    case 'loading':
      return <Spinner />;
    case 'success':
      return <div>{state.data.name}</div>; // data is available
    case 'error':
      return <div>{state.error.message}</div>; // error is available
  }
}

// Action types
type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'set'; payload: number };

function reducer(state: number, action: Action): number {
  switch (action.type) {
    case 'increment':
      return state + 1;
    case 'decrement':
      return state - 1;
    case 'set':
      return action.payload; // payload only available for 'set'
  }
}
```

## Chapter 4: Type Guards

External data is messy. Guards help you validate it safely.

"Narrow before you use," Marcus says.

```tsx
// Type predicate
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value
  );
}

function handleResponse(data: unknown) {
  if (isUser(data)) {
    console.log(data.name); // TypeScript knows it's a User
  }
}

// Assertion function
function assertIsUser(value: unknown): asserts value is User {
  if (!isUser(value)) {
    throw new Error('Not a user');
  }
}

function processUser(data: unknown) {
  assertIsUser(data);
  console.log(data.name); // data is User after assertion
}

// in operator narrowing
type Admin = { id: string; permissions: string[] };
type Guest = { id: string; expiresAt: Date };

function handleUser(user: Admin | Guest) {
  if ('permissions' in user) {
    console.log(user.permissions); // Admin
  } else {
    console.log(user.expiresAt); // Guest
  }
}
```

## Chapter 5: Component Props Patterns

You need prop types that guide developers toward correct usage.

"Patterns keep teams aligned," Sarah says.

```tsx
// Props with children
interface CardProps {
  title: string;
  children: React.ReactNode;
}

// Optional children
interface PanelProps {
  children?: React.ReactNode;
}

// Specific children type
interface ListProps {
  children: React.ReactElement<ListItemProps>[];
}

// Props that extend HTML elements
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant: 'primary' | 'secondary';
  isLoading?: boolean;
}

function Button({ variant, isLoading, children, ...props }: ButtonProps) {
  return (
    <button {...props}>
      {isLoading ? <Spinner /> : children}
    </button>
  );
}

// Component props extraction
type ExtractedButtonProps = React.ComponentProps<typeof Button>;
type InputProps = React.ComponentProps<'input'>;
```

## Chapter 6: Strict Configuration

Strict settings catch bugs earlier, even if they feel noisy at first.

"Strict is faster than loose," Marcus says.

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true,
    "noPropertyAccessFromIndexSignature": true
  }
}
```

**Key strict options:**
- `strictNullChecks` - Catch null/undefined errors
- `noUncheckedIndexedAccess` - Array access returns T | undefined
- `exactOptionalPropertyTypes` - Distinguish undefined from missing

## Chapter 7: Advanced Patterns

These patterns help when your types get truly complex.

"Use advanced types as tools, not trophies," Sarah says.

```tsx
// Template literal types
type EventName = `on${Capitalize<string>}`;
type OnClick = `onClick`; // valid
type click = `click`; // not valid for EventName

// Conditional types
type ArrayElement<T> = T extends (infer E)[] ? E : never;
type Item = ArrayElement<string[]>; // string

// Mapped types
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Optional<T> = {
  [P in keyof T]?: T[P];
};

// Make specific fields optional
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type UserWithOptionalEmail = PartialBy<User, 'email'>;
```

---

## Common Mistakes

1. **Using `any` as a shortcut** - It breaks the safety net.
2. **Overly complex types** - Prefer readable types over clever ones.
3. **Ignoring `strict` errors** - They signal real risks.
4. **Missing runtime checks** - Types don't validate data at runtime.


Example:
```ts
// ❌ any hides real types
function identity(value: any) {
  return value;
}

// ✅ Generic preserves types
function identity<T>(value: T): T {
  return value;
}
```

## Practice Exercises

1. Create a generic `useLocalStorage` hook with proper types
2. Build a type-safe form with discriminated unions for field types
3. Implement a polymorphic `as` prop component

### Solutions

<details>
<summary>Exercise 1: useLocalStorage</summary>

```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    try {
      const stored = window.localStorage.getItem(key);
      return stored ? (JSON.parse(stored) as T) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setStoredValue = (nextValue: T | ((prev: T) => T)) => {
    setValue((prev) => {
      const resolved = typeof nextValue === 'function'
        ? (nextValue as (prev: T) => T)(prev)
        : nextValue;
      window.localStorage.setItem(key, JSON.stringify(resolved));
      return resolved;
    });
  };

  return [value, setStoredValue] as const;
}
```

**Key points:**
- Generic type is inferred from `initialValue`.
- State and storage stay in sync.
- Returned tuple is typed with `as const`.

</details>

<details>
<summary>Exercise 2: Discriminated Form Fields</summary>

```tsx
type TextField = { type: 'text'; name: string; label: string; placeholder?: string };
type NumberField = { type: 'number'; name: string; label: string; min?: number; max?: number };
type SelectField = { type: 'select'; name: string; label: string; options: string[] };
type Field = TextField | NumberField | SelectField;

function FieldRenderer({ field }: { field: Field }) {
  if (field.type === 'text') {
    return (
      <input
        name={field.name}
        placeholder={field.placeholder}
        aria-label={field.label}
      />
    );
  }

  if (field.type === 'number') {
    return (
      <input
        name={field.name}
        type="number"
        min={field.min}
        max={field.max}
        aria-label={field.label}
      />
    );
  }

  return (
    <select name={field.name} aria-label={field.label}>
      {field.options.map((option) => (
        <option key={option} value={option}>
          {option}
        </option>
      ))}
    </select>
  );
}
```

**Key points:**
- The `type` field narrows the union safely.
- Each variant enforces required props.
- Rendering logic stays type-safe.

</details>

<details>
<summary>Exercise 3: Polymorphic Component</summary>

```tsx
type AsProp<C extends React.ElementType> = { as?: C };
type Props<C extends React.ElementType> = AsProp<C> &
  Omit<React.ComponentPropsWithRef<C>, 'as'> & {
    variant?: 'primary' | 'secondary';
  };

const Text = React.forwardRef(function Text<C extends React.ElementType = 'span'>(
  { as, variant = 'primary', ...props }: Props<C>,
  ref: React.Ref<Element>
) {
  const Component = as ?? 'span';
  const resolvedProps =
    Component === 'button' && !('type' in props)
      ? { type: 'button', ...props }
      : props;
  return <Component ref={ref} data-variant={variant} {...resolvedProps} />;
});
```

**Key points:**
- `as` changes the rendered element safely.
- Props are inferred from the chosen element type.
- Variants stay consistent across elements.

</details>

---

## What You Learned

This module covered:

- **Generics**: Reusable components with strong typing
- **Utility types**: Shaping and composing types
- **Unions**: Explicit, safe UI state modeling
- **Type guards**: Safe narrowing for runtime data
- **Prop patterns**: Preventing misuse with types

**Key takeaway**: Types aren't just annotations -- they're design constraints that prevent bugs.

---

## Real-World Application

This week at work, you might use these concepts to:

- Build a reusable list component with generic typing
- Model API response states with discriminated unions
- Add safe polymorphic components to a design system
- Enforce strict typing for shared utilities
- Catch invalid props before they reach production

---

## Further Reading

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)

---

**Navigation**: [← Previous Module](./09-accessibility-testing-tooling.md) | [Next Module →](./11-code-quality.md)
