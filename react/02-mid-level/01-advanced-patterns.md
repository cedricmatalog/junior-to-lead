# Advanced Patterns

Master patterns that solve complex component design challenges.

## Learning Objectives

By the end of this module, you will:
- Implement Higher-Order Components (HOCs)
- Use render props for flexible composition
- Build headless UI components
- Create polymorphic components with TypeScript

## Higher-Order Components (HOCs)

Functions that take a component and return an enhanced component.

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

## Render Props

Pass a function as children or props to share logic.

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

## Headless UI Components

Components that handle logic without rendering UI.

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

## Polymorphic Components

Components that render as different elements.

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

## Compound Components Pattern

Components that work together with shared state.

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

## Practice Exercises

1. Build a `withDataFetching` HOC that handles loading/error states
2. Create a headless `useCombobox` hook for autocomplete
3. Build a polymorphic `Button` that can render as `button`, `a`, or `Link`

## Further Reading

- [Patterns.dev - React Patterns](https://www.patterns.dev/react)
- [Headless UI](https://headlessui.com/)
