# Advanced Patterns

> **Last reviewed**: 2026-01-06


## Week One: The Dashboard Refactor

The streaming dashboard finally has traction, but the codebase is showing its age. Sarah asks you to add authentication gates, shared behaviors, and layout variants without copying logic into every component. Marcus points at the newest pull request and sighs: "You're building the same wrapper three times." You need patterns that let components stay flexible while the logic stays centralized. This week is about learning those reusable patterns so new features don't feel like rewrites.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **HOCs** | A power adapter - same device, different outlet |
| **Render props** | A stage manager - gives cues, you decide the scene |
| **Headless UI** | A skeleton - structure without a skin |
| **Polymorphic components** | A Swiss Army knife - one tool, many forms |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Junior Module 02 (Component Patterns) - Comfort with component composition, props, and reusable UI patterns.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Implement Higher-Order Components (HOCs) to reuse cross-cutting logic
- [ ] Use render props to share behavior across different layouts
- [ ] Build headless UI components that separate behavior from presentation
- [ ] Create polymorphic components with safe prop typing
- [ ] Assemble compound components that coordinate internal state
- [ ] Choose the right pattern based on tradeoffs and team constraints

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice these patterns over 3-5 weeks across real features

---

## Chapter 1: Higher-Order Components (HOCs)

Sarah needs an authentication gate for multiple routes, and duplicating the same logic across screens is already getting messy.

"Wrap it once, then reuse it," Sarah says. "Stop copying the same guard."

A higher-order component takes a component and returns an enhanced component.

```jsx
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const { user, isLoading } = useAuth();

    if (isLoading) return <Spinner />;
    if (!user) return <Navigate to="/login" />;

    return <WrappedComponent {...props} user={user} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);
```

**When to use HOCs:**
- Cross-cutting concerns (auth, logging, theming)
- Injecting props from context
- Conditional rendering wrappers

**HOC conventions:**
- Name: `withSomething`
- Pass through props: `{...props}`
- Copy static methods and display name

```jsx
function withLogger(WrappedComponent) {
  function WithLogger(props) {
    useEffect(() => {
      console.log('Mounted:', WrappedComponent.name);
    }, []);
    return <WrappedComponent {...props} />;
  }

  WithLogger.displayName = `WithLogger(${WrappedComponent.name})`;
  return WithLogger;
}
```

## Chapter 2: Render Props

Marcus wants a reusable mouse-tracking behavior, but each screen needs a different UI.

"Keep the behavior, swap the view," he says. "That's a render prop."

You pass a function as children or props to share logic.

```jsx
function MouseTracker({ children }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);

  return children(position);
}

// Usage
<MouseTracker>
  {({ x, y }) => (
    <div>Mouse is at ({x}, {y})</div>
  )}
</MouseTracker>
```

**Modern alternative: Custom hooks are often cleaner:**

```jsx
function useMousePosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  // ... same logic
  return position;
}

function MyComponent() {
  const { x, y } = useMousePosition();
  return <div>Mouse is at ({x}, {y})</div>;
}
```

## Chapter 3: Headless UI Components

Design hands you a new visual system. You need to keep behavior consistent while swapping the presentation entirely.

"Separate the brain from the skin," Sarah says.

Headless components handle logic without rendering UI.

```jsx
function useToggle(initialValue = false) {
  const [isOpen, setIsOpen] = useState(initialValue);

  const toggle = useCallback(() => setIsOpen(v => !v), []);
  const open = useCallback(() => setIsOpen(true), []);
  const close = useCallback(() => setIsOpen(false), []);

  return { isOpen, toggle, open, close };
}

function useDisclosure(initialValue = false) {
  const { isOpen, toggle, open, close } = useToggle(initialValue);

  const getButtonProps = (props = {}) => ({
    ...props,
    onClick: toggle,
    'aria-expanded': isOpen,
  });

  const getContentProps = (props = {}) => ({
    ...props,
    hidden: !isOpen,
  });

  return { isOpen, open, close, toggle, getButtonProps, getContentProps };
}

// Usage - user controls all styling
function Accordion({ title, children }) {
  const { getButtonProps, getContentProps } = useDisclosure();

  return (
    <div className="my-custom-accordion">
      <button {...getButtonProps({ className: 'accordion-trigger' })}>
        {title}
      </button>
      <div {...getContentProps({ className: 'accordion-content' })}>
        {children}
      </div>
    </div>
  );
}
```

## Chapter 4: Polymorphic Components

Your design system needs a `Button` that can be a link, a button, or a router component without losing type safety.

"One component, many faces," Marcus says.

Polymorphic components render as different elements.

```tsx
type PolymorphicProps<E extends React.ElementType> = {
  as?: E;
  children: React.ReactNode;
} & Omit<React.ComponentPropsWithoutRef<E>, 'as' | 'children'>;

function Text<E extends React.ElementType = 'span'>({
  as,
  children,
  ...props
}: PolymorphicProps<E>) {
  const Component = as || 'span';
  return <Component {...props}>{children}</Component>;
}

// Usage
<Text>Default span</Text>
<Text as="p">Paragraph</Text>
<Text as="h1" className="title">Heading</Text>
<Text as="a" href="/about">Link</Text>
```

## Chapter 5: Compound Components Pattern

The select menu is getting complex. You need sub-components that share state without threading props through every layer.

"Stop threading props and let the parts collaborate," Sarah says.

Compound components work together with shared state.

```jsx
const SelectContext = createContext(null);

function Select({ value, onChange, children }) {
  return (
    <SelectContext.Provider value={{ value, onChange }}>
      <div className="select">{children}</div>
    </SelectContext.Provider>
  );
}

function SelectTrigger({ children }) {
  const { value } = useContext(SelectContext);
  return <button className="select-trigger">{children || value}</button>;
}

function SelectOptions({ children }) {
  return <ul className="select-options">{children}</ul>;
}

function SelectOption({ value, children }) {
  const { value: selectedValue, onChange } = useContext(SelectContext);
  const isSelected = value === selectedValue;

  return (
    <li
      className={`select-option ${isSelected ? 'selected' : ''}`}
      onClick={() => onChange(value)}
    >
      {children}
    </li>
  );
}

// Attach sub-components
Select.Trigger = SelectTrigger;
Select.Options = SelectOptions;
Select.Option = SelectOption;

// Usage
<Select value={country} onChange={setCountry}>
  <Select.Trigger>Select a country</Select.Trigger>
  <Select.Options>
    <Select.Option value="us">United States</Select.Option>
    <Select.Option value="uk">United Kingdom</Select.Option>
  </Select.Options>
</Select>
```

---

## Common Mistakes

1. **Stacking too many patterns** - A hook is often simpler than an HOC or render props.
2. **Render-prop nesting** - Deeply nested render props make code hard to read and test.
3. **Leaking presentation into headless logic** - Keep styles out of headless components so they stay reusable.
4. **Skipping prop forwarding** - Polymorphic components must forward props and refs to avoid broken accessibility.


Example:
```jsx
// ❌ Copy-paste auth checks in every screen
if (!user) return <Login />;

// ✅ Share behavior with a HOC
const ProtectedDashboard = withAuth(Dashboard);
```

## Practice Exercises

1. Build a `withDataFetching` HOC that handles loading/error states
2. Create a headless `useCombobox` hook for autocomplete
3. Build a polymorphic `Button` that can render as `button`, `a`, or `Link`

### Solutions

<details>
<summary>Exercise 1: withDataFetching HOC</summary>

```jsx
function withDataFetching(WrappedComponent, fetchData, getDeps = () => []) {
  return function WithDataFetching(props) {
    const [data, setData] = useState(null);
    const [error, setError] = useState(null);
    const [isLoading, setIsLoading] = useState(true);

    useEffect(() => {
      let cancelled = false;
      const controller = new AbortController();

      setIsLoading(true);
      setError(null);

      fetchData({ ...props, signal: controller.signal })
        .then((result) => {
          if (!cancelled) {
            setData(result);
            setIsLoading(false);
          }
        })
        .catch((err) => {
          if (!cancelled && err.name !== 'AbortError') {
            setError(err);
            setIsLoading(false);
          }
        });

      return () => {
        cancelled = true;
        controller.abort();
      };
    }, [fetchData, ...getDeps(props)]);

    return (
      <WrappedComponent
        {...props}
        data={data}
        error={error}
        isLoading={isLoading}
      />
    );
  };
}

// Usage
const withUserData = (Component) =>
  withDataFetching(
    Component,
    ({ userId, signal }) =>
      fetch(`/api/users/${userId}`, { signal }).then((r) => r.json()),
    (props) => [props.userId]
  );
```

**Key points:**
- HOC owns loading/error state and passes it down as props.
- Cleanup prevents state updates after unmount.
- `getDeps` keeps the effect stable and avoids re-fetching on every render.

</details>

<details>
<summary>Exercise 2: Headless useCombobox Hook</summary>

```jsx
function useCombobox({ items, itemToString = (item) => item, onSelect }) {
  const [query, setQuery] = useState('');
  const [isOpen, setIsOpen] = useState(false);
  const [activeIndex, setActiveIndex] = useState(0);
  const listId = 'combobox-list';

  const filteredItems = items.filter((item) =>
    itemToString(item).toLowerCase().includes(query.toLowerCase())
  );

  const getInputProps = () => ({
    value: query,
    onChange: (event) => {
      setQuery(event.target.value);
      setIsOpen(true);
      setActiveIndex(0);
    },
    onFocus: () => setIsOpen(true),
    onBlur: () => setIsOpen(false),
    onKeyDown: (event) => {
      if (!isOpen) return;
      if (event.key === 'ArrowDown') {
        setActiveIndex((index) => Math.min(index + 1, filteredItems.length - 1));
      }
      if (event.key === 'ArrowUp') {
        setActiveIndex((index) => Math.max(index - 1, 0));
      }
      if (event.key === 'Enter') {
        onSelect?.(filteredItems[activeIndex]);
        setIsOpen(false);
      }
    },
    'aria-controls': listId,
    'aria-expanded': isOpen
  });

  const getItemProps = (item, index) => ({
    id: `${listId}-option-${index}`,
    onMouseDown: () => {
      setQuery(itemToString(item));
      setIsOpen(false);
      onSelect?.(item);
    },
    role: 'option',
    'aria-selected': index === activeIndex
  });

  return {
    isOpen,
    query,
    filteredItems,
    activeIndex,
    listId,
    getInputProps,
    getItemProps
  };
}

// Usage
function Combobox({ items }) {
  const { isOpen, filteredItems, activeIndex, listId, getInputProps, getItemProps } = useCombobox({
    items,
    onSelect: (item) => console.log('Selected', item)
  });

  return (
    <div className="combobox">
      <input
        {...getInputProps()}
        role="combobox"
        aria-autocomplete="list"
        aria-activedescendant={`${listId}-option-${activeIndex}`}
      />
      {isOpen && (
        <ul id={listId} role="listbox">
          {filteredItems.map((item, index) => (
            <li key={item.id ?? item} {...getItemProps(item, index)}>
              {item.label ?? item}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

**Key points:**
- The hook owns state and handlers; UI remains customizable.
- `itemToString` keeps the hook flexible across data shapes.
- Keyboard navigation and aria attributes improve accessibility.

</details>

<details>
<summary>Exercise 3: Polymorphic Button</summary>

```tsx
import { forwardRef } from 'react';
import type { ElementType, ComponentPropsWithoutRef } from 'react';

type ButtonProps<C extends ElementType> = {
  as?: C;
  variant?: 'primary' | 'ghost';
} & ComponentPropsWithoutRef<C>;

const Button = forwardRef(function Button<C extends ElementType = 'button'>(
  { as, variant = 'primary', className, ...props }: ButtonProps<C>,
  ref: React.Ref<Element>
) {
  const Component = as ?? 'button';
  const classes = ['btn', `btn-${variant}`, className].filter(Boolean).join(' ');

  const resolvedProps =
    Component === 'button' && !('type' in props)
      ? { type: 'button', ...props }
      : props;

  return <Component ref={ref} className={classes} {...resolvedProps} />;
});
```

**Key points:**
- `as` controls the rendered element without losing prop typing.
- Props are forwarded so accessibility attributes and handlers work.
- Default `type="button"` avoids accidental form submits.

</details>

---

## What You Learned

This module covered:

- **HOCs**: Wrap components to inject shared behavior
- **Render props**: Share logic while letting UI vary
- **Headless UI**: Separate behavior from presentation
- **Polymorphic components**: Render different elements safely
- **Compound components**: Coordinate state across sub-components

**Key takeaway**: Patterns keep complex UI flexible without duplicating logic.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add authentication gates to multiple routes without repetition
- Build a design-system `Select` that supports custom layouts
- Share drag-and-drop logic across different cards
- Create a `Button` that works in both web and router contexts
- Refactor a monolithic component into composable sub-components

---

## Further Reading

- [Patterns.dev - React Patterns](https://www.patterns.dev/react)
- [Headless UI](https://headlessui.com/)

---

**Navigation**: [Next Module →](./02-performance-optimization.md)
