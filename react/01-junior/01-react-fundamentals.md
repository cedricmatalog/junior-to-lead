# React Fundamentals

Master the building blocks of every React application.

## Learning Objectives

By the end of this module, you will:
- Understand JSX syntax and how it differs from HTML
- Create functional components with props
- Render lists and handle conditional rendering
- Know when React re-renders and why

## Core Concepts

### JSX

JSX is a syntax extension that looks like HTML but compiles to JavaScript function calls.

```jsx
// JSX
const element = <h1 className="title">Hello, World!</h1>;

// Compiles to
const element = React.createElement('h1', { className: 'title' }, 'Hello, World!');
```

**Key differences from HTML:**
- `className` instead of `class`
- `htmlFor` instead of `for`
- camelCase for attributes (`onClick`, `tabIndex`)
- Self-closing tags required (`<img />`, `<br />`)
- JavaScript expressions in curly braces: `{variable}`

### Components

Components are reusable building blocks. Always use functional components (class components are legacy).

```jsx
// A simple component
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// Using the component
<Greeting name="Alice" />
```

**Component rules:**
- Name must start with uppercase letter
- Must return a single root element (or Fragment)
- Props are read-only - never mutate them

### Props

Props pass data from parent to child components.

```jsx
function UserCard({ name, email, isAdmin = false }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
      {isAdmin && <span>Admin</span>}
    </div>
  );
}

// Passing props
<UserCard name="Alice" email="alice@example.com" isAdmin />
```

**Props patterns:**
- Destructure in parameters for clarity
- Provide default values with `=`
- Use `children` for nested content
- Spread props when forwarding: `{...props}`

### Conditional Rendering

Render different UI based on conditions.

```jsx
// && for simple conditions
{isLoggedIn && <LogoutButton />}

// Ternary for either/or
{isLoggedIn ? <Dashboard /> : <LoginForm />}

// Early return for guards
function UserProfile({ user }) {
  if (!user) return <p>Loading...</p>;
  return <div>{user.name}</div>;
}
```

### Rendering Lists

Use `.map()` to render arrays. Always provide a unique `key`.

```jsx
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

**Key rules:**
- Must be unique among siblings
- Use stable IDs, not array indices
- Don't generate keys during render

## Common Mistakes

1. **Mutating props** - Props are read-only, never modify them
2. **Using index as key** - Causes bugs with reordering
3. **Returning multiple elements** - Wrap in Fragment `<>...</>`
4. **Forgetting curly braces** - `{variable}` not `variable`

## Practice Exercises

1. Create a `ProductCard` component that displays name, price, and availability
2. Build a `FilterableList` that conditionally renders items based on a search term
3. Create a `Navbar` that shows different links for logged in/out users

## Further Reading

- [React.dev - Your First Component](https://react.dev/learn/your-first-component)
- [React.dev - Rendering Lists](https://react.dev/learn/rendering-lists)
- [React.dev - Conditional Rendering](https://react.dev/learn/conditional-rendering)
