# Component Patterns

Learn patterns that make your components flexible, reusable, and maintainable.

## Learning Objectives

By the end of this module, you will:
- Use composition to build flexible UIs
- Understand the children prop pattern
- Build compound components for complex UI
- Know when to use controlled vs uncontrolled components

## Core Concepts

### Composition Over Inheritance

React favors composition - building complex UIs from simple pieces.

```jsx
// Bad: Trying to use inheritance-style thinking
function FancyButton extends Button { ... } // Don't do this

// Good: Composition
function FancyButton({ children, ...props }) {
  return (
    <Button className="fancy" {...props}>
      <Sparkle />
      {children}
    </Button>
  );
}
```

### The Children Pattern

`children` is a special prop for nested content.

```jsx
function Card({ children, title }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-content">{children}</div>
    </div>
  );
}

// Usage
<Card title="Welcome">
  <p>This is the card content.</p>
  <button>Click me</button>
</Card>
```

### Slots Pattern

Pass multiple content sections as different props.

```jsx
function Layout({ header, sidebar, children, footer }) {
  return (
    <div className="layout">
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{children}</main>
      <footer>{footer}</footer>
    </div>
  );
}

// Usage
<Layout
  header={<NavBar />}
  sidebar={<Menu />}
  footer={<Copyright />}
>
  <PageContent />
</Layout>
```

### Compound Components

Components that work together, sharing implicit state.

```jsx
function Tabs({ children, defaultTab }) {
  const [activeTab, setActiveTab] = useState(defaultTab);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children }) {
  return <div className="tab-list">{children}</div>;
}

function Tab({ id, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  return (
    <button
      className={activeTab === id ? 'active' : ''}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
}

function TabPanel({ id, children }) {
  const { activeTab } = useContext(TabsContext);
  return activeTab === id ? <div>{children}</div> : null;
}

// Usage - components work together naturally
<Tabs defaultTab="home">
  <TabList>
    <Tab id="home">Home</Tab>
    <Tab id="profile">Profile</Tab>
  </TabList>
  <TabPanel id="home">Home content</TabPanel>
  <TabPanel id="profile">Profile content</TabPanel>
</Tabs>
```

### Controlled vs Uncontrolled Components

**Controlled**: Parent manages state via props.

```jsx
function ControlledInput({ value, onChange }) {
  return <input value={value} onChange={onChange} />;
}

// Parent controls the value
const [text, setText] = useState('');
<ControlledInput value={text} onChange={e => setText(e.target.value)} />
```

**Uncontrolled**: Component manages its own state.

```jsx
function UncontrolledInput({ defaultValue }) {
  const inputRef = useRef();
  return <input ref={inputRef} defaultValue={defaultValue} />;
}

// Read value imperatively when needed
inputRef.current.value
```

**When to use which:**
- **Controlled**: Forms, validation, syncing with other state
- **Uncontrolled**: File inputs, integrating non-React libraries, simple forms

## Common Mistakes

1. **Over-abstracting too early** - Start simple, extract patterns when you see repetition
2. **Prop drilling through many layers** - Use Context for deep trees
3. **Mixing controlled and uncontrolled** - Pick one and stick with it

## Practice Exercises

1. Build a `Modal` component using the children pattern
2. Create a compound `Accordion` component with `AccordionItem` children
3. Build a controlled `SearchInput` with debounced onChange

## Further Reading

- [React.dev - Passing Props](https://react.dev/learn/passing-props-to-a-component)
- [Patterns.dev - Compound Pattern](https://www.patterns.dev/react/compound-pattern)
