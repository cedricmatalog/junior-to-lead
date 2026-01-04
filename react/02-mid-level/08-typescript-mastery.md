# TypeScript Mastery

Leverage TypeScript's full power for safer, more maintainable React code.

## Learning Objectives

By the end of this module, you will:
- Use generics for reusable typed components
- Apply utility types effectively
- Handle complex type patterns
- Configure TypeScript for maximum safety

## Generics in Components

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

## Utility Types

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

## Discriminated Unions

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

## Type Guards

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

## Component Props Patterns

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

## Strict Configuration

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

## Advanced Patterns

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

## Practice Exercises

1. Create a generic `useLocalStorage` hook with proper types
2. Build a type-safe form with discriminated unions for field types
3. Implement a polymorphic `as` prop component

## Further Reading

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
