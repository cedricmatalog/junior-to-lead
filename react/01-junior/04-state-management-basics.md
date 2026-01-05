# State Management Basics

## Week Four: The Cart Problem

Three weeks in, and you've built some impressive features. But Friday afternoon, Sarah drops a new project on your desk.

"We're pivoting," she says. "The streaming service is adding food delivery. Users can order snacks while watching."

Marcus looks up. "Uber Eats meets Netflix?"

"Exactly. And the cart," Sarah continues, "is going to be your biggest state management challenge yet."

She's right. The cart needs to persist across pages. The menu items need to read from it. The checkout needs to update it. The header needs to show the item count. Data flowing everywhere.

"Where does all this state *live*?" you ask.

Sarah smiles. "That's the right question."

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Local state** | A personal notebook - only you can see it, belongs to one component |
| **Global state** | An office whiteboard - everyone can see and reference it |
| **Lifting state** | Moving a note to the bulletin board - shared info goes to a common location |

Keep these in mind. They'll click as we build.

---

## Prerequisites

Module 03 (Hooks Deep Dive) - Understanding useState, useEffect, useContext, and custom hooks.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Determine where state should live (local, lifted, or global)
- [ ] Lift state up to common ancestors when siblings need shared data
- [ ] Use Context API to share global state without prop drilling
- [ ] Implement useReducer for complex state logic with multiple update patterns
- [ ] Build custom hooks that combine Context and useReducer for domain-specific state
- [ ] Decide when to use built-in React state vs external state management libraries

---

## Time Estimate

- **Reading**: 55-70 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice state management decisions over 4-5 weeks building features

---

## Chapter 1: State Has Addresses

"Think of state like mail," Marcus explains. "It has to live somewhere. The question is: which mailbox?"

**Local state** - belongs to one component:

```jsx
function SearchInput() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

"The search query only matters to this input. Local state."

**Shared state** - needed by siblings, lives in parent:

```jsx
function MenuPage() {
  const [filter, setFilter] = useState('all');

  return (
    <>
      <FilterButtons filter={filter} onChange={setFilter} />
      <MenuItems filter={filter} />
    </>
  );
}
```

"Both components need the filter. So it lives in their common parent."

**Global state** - needed everywhere:

```jsx
const UserContext = createContext(null);

function App() {
  const [user, setUser] = useState(null);
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Router />
    </UserContext.Provider>
  );
}
```

"The logged-in user matters to the whole app. Global state."

---

## Chapter 2: Lifting State Up

You start building the menu page. Temperature-controlled drinks need Celsius and Fahrenheit displays. Your first attempt:

```jsx
function TemperatureInputs() {
  return (
    <>
      <CelsiusInput />    {/* has own state */}
      <FahrenheitInput /> {/* has own state */}
    </>
  );
}
```

"They're out of sync," Marcus points out. "Change one, the other doesn't know."

The fix: move the state up to their parent.

```jsx
function TemperatureInputs() {
  const [celsius, setCelsius] = useState(0);
  const fahrenheit = (celsius * 9/5) + 32;

  return (
    <>
      <CelsiusInput
        value={celsius}
        onChange={setCelsius}
      />
      <FahrenheitInput
        value={fahrenheit}
        onChange={f => setCelsius((f - 32) * 5/9)}
      />
    </>
  );
}
```

"Now there's one source of truth. Both inputs reflect the same data."

**The rule**: When siblings need the same state, lift it to their nearest common ancestor.

Here's how state lifting works:

```
┌──────────────────────────────────────────────────────────┐
│              State Lifting Pattern                       │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  BEFORE (siblings have separate state):                  │
│                                                          │
│         TemperatureInputs                                │
│              │                                           │
│       ┌──────┴──────┐                                    │
│       │             │                                    │
│  CelsiusInput   FahrenheitInput                          │
│    state: 0°C     state: 32°F                            │
│                                                          │
│  Problem: Change one, other doesn't update!              │
│  They don't communicate                                  │
│                                                          │
│  ─────────────────────────────────────────────────────   │
│                                                          │
│  AFTER (state lifted to parent):                         │
│                                                          │
│         TemperatureInputs                                │
│         state: celsius = 0                               │
│              │                                           │
│         ┌────┴─────┐                                     │
│         ↓          ↓                                     │
│    CelsiusInput   FahrenheitInput                        │
│    props:         props:                                 │
│    • value: 0     • value: 32                            │
│    • onChange     • onChange                             │
│                                                          │
│  Parent calculates: fahrenheit = (celsius * 9/5) + 32    │
│  Both inputs stay in sync!                               │
│                                                          │
│  Data flows DOWN (via props)                             │
│  Events flow UP (via callbacks)                          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Chapter 3: Context for the Cart

"The cart is different," Sarah says. "It's needed by the header, the menu items, the checkout page, the order summary. They're not siblings—they're scattered across the whole app."

Lifting state all the way to `App` and passing it through every component would be chaos. This is where Context shines.

```jsx
const ThemeContext = createContext('light');

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const toggle = () => setTheme(t => t === 'light' ? 'dark' : 'light');

  return (
    <ThemeContext.Provider value={{ theme, toggle }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

Any component can now grab the theme:

```jsx
function ThemeToggle() {
  const { theme, toggle } = useContext(ThemeContext);
  return (
    <button onClick={toggle}>
      Current: {theme}
    </button>
  );
}
```

"No props. No drilling. The component just reaches up and grabs what it needs."

Here's how Context works as a provider tree:

```
┌──────────────────────────────────────────────────────────┐
│           Context Provider Tree                          │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  <App>                                                   │
│    └─ <ThemeProvider value={{ theme, toggle }}>          │
│         │                                                │
│         ├─ <Header>                                      │
│         │    └─ <ThemeToggle>                            │
│         │         └─ useContext(ThemeContext) ✓         │
│         │            Gets: { theme, toggle }             │
│         │                                                │
│         ├─ <Sidebar>                                     │
│         │    └─ <Navigation>                             │
│         │         └─ (doesn't use theme)                 │
│         │                                                │
│         └─ <MainContent>                                 │
│              └─ <ArticleCard>                            │
│                   └─ useContext(ThemeContext) ✓         │
│                      Gets: { theme, toggle }             │
│                                                          │
│  Key points:                                             │
│  • Provider wraps tree at top level                      │
│  • Any descendant can useContext                         │
│  • No prop drilling through intermediate components      │
│  • Components not using context don't re-render due to    │
│    context changes (but can re-render when parents do)   │
│  • Multiple contexts can be nested                       │
│                                                          │
│  Without Context (prop drilling):                        │
│  App → Header → ThemeToggle                              │
│   │     ↑                                                │
│   └─────┘ (pass theme through every level)               │
│                                                          │
│  With Context (direct access):                           │
│  App → Header → ThemeToggle                              │
│   │              ↑                                       │
│   └──────────────┘ (skip intermediate levels)            │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Chapter 4: The Cart Takes Shape

Time to build the real thing. The cart state is complex—items, quantities, customizations, restaurant info—so you use `useReducer` with Context.

```jsx
const OrderContext = createContext(null);

function OrderProvider({ children }) {
  const [order, dispatch] = useReducer(orderReducer, {
    restaurantId: null,
    items: [],
    subtotal: 0,
    deliveryAddress: null,
    specialInstructions: '',
  });

  // Memoize to prevent unnecessary re-renders
  const value = useMemo(() => ({ order, dispatch }), [order]);

  return (
    <OrderContext.Provider value={value}>
      {children}
    </OrderContext.Provider>
  );
}
```

The reducer handles all the cart logic:

```jsx
function orderReducer(state, action) {
  switch (action.type) {
    case 'SET_RESTAURANT':
      // Clear cart when switching restaurants
      return { ...state, restaurantId: action.restaurantId, items: [], subtotal: 0 };

    case 'ADD_ITEM':
      const existingIndex = state.items.findIndex(
        item => item.menuItemId === action.item.menuItemId &&
                JSON.stringify(item.customizations) === JSON.stringify(action.item.customizations)
      );

      if (existingIndex >= 0) {
        const updatedItems = [...state.items];
        updatedItems[existingIndex].quantity += action.item.quantity;
        return {
          ...state,
          items: updatedItems,
          subtotal: calculateSubtotal(updatedItems),
        };
      }

      return {
        ...state,
        items: [...state.items, action.item],
        subtotal: calculateSubtotal([...state.items, action.item]),
      };

    case 'REMOVE_ITEM':
      const filtered = state.items.filter((_, i) => i !== action.index);
      return { ...state, items: filtered, subtotal: calculateSubtotal(filtered) };

    case 'SET_ADDRESS':
      return { ...state, deliveryAddress: action.address };

    case 'CLEAR_ORDER':
      return { restaurantId: null, items: [], subtotal: 0, deliveryAddress: null, specialInstructions: '' };

    default:
      return state;
  }
}
```

"The reducer is the brain," Marcus explains. "It knows all the rules: can't mix restaurants, quantities stack, clearing resets everything."

Here's how useReducer works as a state machine:

```
┌──────────────────────────────────────────────────────────┐
│          useReducer State Machine                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Component dispatches action:                            │
│  dispatch({ type: 'ADD_ITEM', item: {...} })             │
│       │                                                  │
│       ↓                                                  │
│  ┌─────────────────────────────────────┐                 │
│  │       Reducer Function              │                 │
│  │  (current state, action) => new state│                │
│  └─────────────────────────────────────┘                 │
│       │                                                  │
│       ├─ switch (action.type)                            │
│       │                                                  │
│       ├──→ 'ADD_ITEM'                                    │
│       │      • Check if item exists                      │
│       │      • Update quantity or add new                │
│       │      • Recalculate subtotal                      │
│       │      • Return new state                          │
│       │                                                  │
│       ├──→ 'REMOVE_ITEM'                                 │
│       │      • Filter out item                           │
│       │      • Recalculate subtotal                      │
│       │      • Return new state                          │
│       │                                                  │
│       ├──→ 'SET_RESTAURANT'                              │
│       │      • Clear existing items                      │
│       │      • Set new restaurant                        │
│       │      • Return new state                          │
│       │                                                  │
│       └──→ default                                       │
│              • Return unchanged state                    │
│       │                                                  │
│       ↓                                                  │
│  React updates component with new state                  │
│       │                                                  │
│       ↓                                                  │
│  Component re-renders                                    │
│                                                          │
│  Benefits of useReducer:                                 │
│  • Complex state logic in one place                      │
│  • Predictable state transitions                         │
│  • Easy to test (pure function)                          │
│  • Action history for debugging                          │ 
│                                                          │
│  vs useState:                                            │
│  • useState: Simple state (toggle, counter)              │
│  • useReducer: Complex state (cart, form, wizard)        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Chapter 5: A Custom Hook for Cleaner Code

Sarah reviews your work. "Add a safety net."

```jsx
function useOrder() {
  const context = useContext(OrderContext);
  if (!context) {
    throw new Error('useOrder must be used within OrderProvider');
  }
  return context;
}
```

"Now if someone forgets to wrap their component in the provider, they get a helpful error instead of undefined crashes."

**Context best practices:**
1. Split contexts by domain - Don't put everything in one
2. Memoize provider values - Prevent unnecessary re-renders
3. Create custom hooks - Better DX and error handling

---

## Chapter 6: Mixing Local and Global

The menu item component shows the pattern in action:

```jsx
function MenuItem({ item, restaurantId }) {
  // Local state for this component's UI
  const [isCustomizing, setIsCustomizing] = useState(false);
  const [quantity, setQuantity] = useState(1);
  const [customizations, setCustomizations] = useState({});

  // Global state from context
  const { order, dispatch } = useOrder();

  const isDifferentRestaurant = order.restaurantId && order.restaurantId !== restaurantId;

  const handleAddToOrder = () => {
    if (isDifferentRestaurant) {
      // Show confirmation modal to clear existing order
      return;
    }

    if (!order.restaurantId) {
      dispatch({ type: 'SET_RESTAURANT', restaurantId });
    }

    dispatch({
      type: 'ADD_ITEM',
      item: {
        menuItemId: item.id,
        name: item.name,
        price: item.price,
        quantity,
        customizations,
      },
    });

    // Reset local state
    setIsCustomizing(false);
    setQuantity(1);
    setCustomizations({});
  };

  return (
    <div className="menu-item">
      <img src={item.image} alt={item.name} />
      <div>
        <h3>{item.name}</h3>
        <p>{item.description}</p>
        <span className="price">${item.price.toFixed(2)}</span>
      </div>
      <button onClick={() => setIsCustomizing(true)}>Add</button>

      {isCustomizing && (
        <CustomizationModal
          item={item}
          quantity={quantity}
          onQuantityChange={setQuantity}
          customizations={customizations}
          onCustomizationChange={setCustomizations}
          onAdd={handleAddToOrder}
          onClose={() => setIsCustomizing(false)}
        />
      )}
    </div>
  );
}
```

Notice the split:
- **Local state**: Modal open/closed, quantity selector, customizations in progress
- **Global state**: The actual cart contents

"Local state is ephemeral," Sarah explains. "The customization modal's state disappears when you close it. The cart persists across the whole session."

---

## Chapter 7: The Order Summary

The cart sidebar shows how easy it is to consume global state:

```jsx
function OrderSummary() {
  const { order, dispatch } = useOrder();

  if (order.items.length === 0) {
    return <p className="empty-cart">Your cart is empty</p>;
  }

  return (
    <div className="order-summary">
      <h2>Your Order</h2>
      {order.items.map((item, index) => (
        <div
          key={`${item.menuItemId}-${JSON.stringify(item.customizations)}`}
          className="order-item"
        >
          <span>{item.quantity}x {item.name}</span>
          <span>${(item.price * item.quantity).toFixed(2)}</span>
          <button onClick={() => dispatch({ type: 'REMOVE_ITEM', index })}>
            Remove
          </button>
        </div>
      ))}
      <div className="subtotal">
        <span>Subtotal</span>
        <span>${order.subtotal.toFixed(2)}</span>
      </div>
    </div>
  );
}
```

This component doesn't care where it's rendered—header, sidebar, checkout page. It grabs what it needs from context and displays it.

---

## Chapter 8: When to Reach for Libraries

"When do we need Redux?" you ask.

"Rarely," Sarah admits. "But there are signs."

**Stick with useState + Context when:**
- App is small to medium
- State updates are straightforward
- Team is newer to React

**Consider Redux/Zustand when:**
- Complex state with many interconnected updates
- Need middleware (logging, persistence)
- Large team needing strict patterns
- DevTools and time-travel debugging needed

```jsx
// Zustand - simpler alternative to Redux
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set(state => ({ count: state.count + 1 })),
}));

function Counter() {
  const { count, increment } = useStore();
  return <button onClick={increment}>{count}</button>;
}
```

"For this food delivery app? Context + useReducer is perfect. We'll know when we outgrow it."

---

## State Decision Tree

| Question | If Yes... |
|----------|-----------|
| Does only one component need it? | Local state (useState) |
| Do sibling components need it? | Lift to parent |
| Do distant components need it? | Context API |
| Is it server data with caching needs? | TanStack Query / SWR |
| Is it complex with many update patterns? | useReducer or external library |

## Common Mistakes

1. **Using state for derived values** - If you can calculate it, don't store it
2. **Too much global state** - Keep state as local as possible
3. **Premature Context** - A few levels of props is fine

---

## Debug This Code

Before moving to the exercises, test your debugging skills. Each snippet has bugs - can you spot and fix them?

<details>
<summary>Challenge 1: Lifting State Gone Wrong</summary>

```jsx
function TemperatureConverter() {
  return (
    <>
      <CelsiusInput />
      <FahrenheitInput />
    </>
  );
}

function CelsiusInput() {
  const [celsius, setCelsius] = useState(0);

  return (
    <div>
      <label>Celsius:</label>
      <input
        type="number"
        value={celsius}
        onChange={(e) => setCelsius(Number(e.target.value))}
      />
    </div>
  );
}

function FahrenheitInput() {
  const [fahrenheit, setFahrenheit] = useState(32);

  return (
    <div>
      <label>Fahrenheit:</label>
      <input
        type="number"
        value={fahrenheit}
        onChange={(e) => setFahrenheit(Number(e.target.value))}
      />
    </div>
  );
}
```

**How many bugs can you find?** (Answer: 1 architectural issue)

<details>
<summary>Hint</summary>

The components render, but do they stay in sync when you change one temperature?

</details>

<details>
<summary>Solution</summary>

**Bug**: State is not lifted - each input has independent state, so they don't stay synchronized. Changing Celsius doesn't update Fahrenheit and vice versa.

**Fixed code:**

```jsx
function TemperatureConverter() {
  const [celsius, setCelsius] = useState(0);
  const fahrenheit = (celsius * 9/5) + 32;

  return (
    <>
      <CelsiusInput value={celsius} onChange={setCelsius} />
      <FahrenheitInput
        value={fahrenheit}
        onChange={(f) => setCelsius((f - 32) * 5/9)}
      />
    </>
  );
}

function CelsiusInput({ value, onChange }) {
  return (
    <div>
      <label>Celsius:</label>
      <input
        type="number"
        value={value}
        onChange={(e) => onChange(Number(e.target.value))}
      />
    </div>
  );
}

function FahrenheitInput({ value, onChange }) {
  return (
    <div>
      <label>Fahrenheit:</label>
      <input
        type="number"
        value={value}
        onChange={(e) => onChange(Number(e.target.value))}
      />
    </div>
  );
}
```

**Why it matters**: Sibling components that need synchronized state must share a single source of truth from their parent.

</details>

</details>

<details>
<summary>Challenge 2: Context Provider Missing</summary>

```jsx
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState('light');

  return (
    <div>
      <Header />
      <MainContent />
    </div>
  );
}

function Header() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <header>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme (Current: {theme})
      </button>
    </header>
  );
}

function MainContent() {
  const { theme } = useContext(ThemeContext);

  return (
    <main className={`theme-${theme}`}>
      <p>Content goes here</p>
    </main>
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

Look at what wraps the component tree. Is the Context value being provided?

</details>

<details>
<summary>Solution</summary>

**Bug**: `ThemeContext.Provider` is never used - the state exists in App but isn't provided to descendants via Context.

**Fixed code:**

```jsx
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <div>
        <Header />
        <MainContent />
      </div>
    </ThemeContext.Provider>
  );
}

function Header() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <header>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme (Current: {theme})
      </button>
    </header>
  );
}

function MainContent() {
  const { theme } = useContext(ThemeContext);

  return (
    <main className={`theme-${theme}`}>
      <p>Content goes here</p>
    </main>
  );
}
```

**Why it matters**: Without a Provider, `useContext` returns `undefined`, causing runtime errors when trying to destructure properties.

</details>

</details>

<details>
<summary>Challenge 3: Reducer Mutation Bug</summary>

```jsx
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      // Add item to cart
      state.items.push(action.item);
      return state;

    case 'UPDATE_QUANTITY':
      // Update item quantity
      const item = state.items.find(i => i.id === action.id);
      if (item) {
        item.quantity = action.quantity;
      }
      return state;

    case 'REMOVE_ITEM':
      // Remove item
      state.items = state.items.filter(i => i.id !== action.id);
      return state;

    default:
      return state;
  }
}

function CartProvider({ children }) {
  const [cart, dispatch] = useReducer(cartReducer, { items: [] });

  return (
    <CartContext.Provider value={{ cart, dispatch }}>
      {children}
    </CartContext.Provider>
  );
}
```

**How many bugs can you find?** (Answer: 3 bugs - same pattern)

<details>
<summary>Hint</summary>

Reducers must be pure functions. Are we modifying the existing state object?

</details>

<details>
<summary>Solution</summary>

**Bug**: All three cases directly mutate the state object instead of returning a new state. Reducers must never mutate - they should return new objects.

**Fixed code:**

```jsx
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      // Create new state with new items array
      return {
        ...state,
        items: [...state.items, action.item]
      };

    case 'UPDATE_QUANTITY':
      // Create new state with mapped items array
      return {
        ...state,
        items: state.items.map(item =>
          item.id === action.id
            ? { ...item, quantity: action.quantity }
            : item
        )
      };

    case 'REMOVE_ITEM':
      // Create new state with filtered items array
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.id)
      };

    default:
      return state;
  }
}

function CartProvider({ children }) {
  const [cart, dispatch] = useReducer(cartReducer, { items: [] });

  return (
    <CartContext.Provider value={{ cart, dispatch }}>
      {children}
    </CartContext.Provider>
  );
}
```

**Why it matters**: Mutating state breaks React's change detection - React compares references, so if you return the same object reference, it won't trigger re-renders even though the contents changed.

</details>

</details>

---

## Practice Exercises

1. Build a shopping cart with Context (add, remove, update quantity)
2. Create a form with multiple inputs sharing validation state
3. Implement a notification system with Context and reducer

### Solutions

<details>
<summary>Exercise 1: Shopping Cart with Context</summary>

```jsx
import { createContext, useContext, useReducer, useMemo } from 'react';

// Context
const CartContext = createContext(null);

// Reducer
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      const existingIndex = state.items.findIndex(
        item => item.id === action.item.id
      );

      if (existingIndex >= 0) {
        const updatedItems = [...state.items];
        updatedItems[existingIndex].quantity += action.item.quantity || 1;
        return { ...state, items: updatedItems };
      }

      return {
        ...state,
        items: [...state.items, { ...action.item, quantity: action.item.quantity || 1 }]
      };

    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.id)
      };

    case 'UPDATE_QUANTITY':
      return {
        ...state,
        items: state.items.map(item =>
          item.id === action.id
            ? { ...item, quantity: action.quantity }
            : item
        )
      };

    case 'CLEAR_CART':
      return { items: [] };

    default:
      return state;
  }
}

// Provider
function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });

  const total = useMemo(() => {
    return state.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );
  }, [state.items]);

  const itemCount = useMemo(() => {
    return state.items.reduce((sum, item) => sum + item.quantity, 0);
  }, [state.items]);

  const value = useMemo(
    () => ({ cart: state, dispatch, total, itemCount }),
    [state, total, itemCount]
  );

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}

// Custom hook
function useCart() {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error('useCart must be used within CartProvider');
  }
  return context;
}

// Usage components
function ProductCard({ product }) {
  const { dispatch } = useCart();

  const addToCart = () => {
    dispatch({ type: 'ADD_ITEM', item: product });
  };

  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price.toFixed(2)}</p>
      <button onClick={addToCart}>Add to Cart</button>
    </div>
  );
}

function CartSummary() {
  const { cart, dispatch, total, itemCount } = useCart();

  if (itemCount === 0) {
    return <p>Your cart is empty</p>;
  }

  return (
    <div className="cart-summary">
      <h2>Cart ({itemCount} items)</h2>
      {cart.items.map(item => (
        <div key={item.id}>
          <span>{item.name}</span>
          <input
            type="number"
            value={item.quantity}
            min="1"
            onChange={(e) =>
              dispatch({
                type: 'UPDATE_QUANTITY',
                id: item.id,
                quantity: parseInt(e.target.value) || 1
              })
            }
          />
          <span>${(item.price * item.quantity).toFixed(2)}</span>
          <button onClick={() => dispatch({ type: 'REMOVE_ITEM', id: item.id })}>
            Remove
          </button>
        </div>
      ))}
      <div className="total">
        <strong>Total: ${total.toFixed(2)}</strong>
      </div>
    </div>
  );
}
```

**Key points:**
- Reducer handles all cart logic in one place for consistency
- useMemo prevents recalculating totals on every render
- Custom hook provides clean API and error checking
- Memoizing the context value prevents unnecessary re-renders
- Quantity updates merge items instead of duplicating

</details>

<details>
<summary>Exercise 2: Form with Shared Validation State</summary>

```jsx
import { useState } from 'react';

function useFormValidation(initialValues, validationRules) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const validate = (fieldName, value) => {
    const rules = validationRules[fieldName];
    if (!rules) return '';

    for (const rule of rules) {
      const error = rule(value, values);
      if (error) return error;
    }
    return '';
  };

  const handleChange = (name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));

    if (touched[name]) {
      const error = validate(name, value);
      setErrors(prev => ({ ...prev, [name]: error }));
    }
  };

  const handleBlur = (name) => {
    setTouched(prev => ({ ...prev, [name]: true }));
    const error = validate(name, values[name]);
    setErrors(prev => ({ ...prev, [name]: error }));
  };

  const validateAll = () => {
    const newErrors = {};
    let isValid = true;

    Object.keys(validationRules).forEach(fieldName => {
      const error = validate(fieldName, values[fieldName]);
      if (error) {
        newErrors[fieldName] = error;
        isValid = false;
      }
    });

    setErrors(newErrors);
    setTouched(
      Object.keys(validationRules).reduce(
        (acc, key) => ({ ...acc, [key]: true }),
        {}
      )
    );

    return isValid;
  };

  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    validateAll
  };
}

// Validation rules
const required = (value) => (!value ? 'This field is required' : '');
const email = (value) =>
  !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) ? 'Invalid email address' : '';
const minLength = (min) => (value) =>
  value.length < min ? `Must be at least ${min} characters` : '';
const matchField = (fieldName) => (value, allValues) =>
  value !== allValues[fieldName] ? `Must match ${fieldName}` : '';

// Usage
function SignupForm() {
  const {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    validateAll
  } = useFormValidation(
    {
      email: '',
      password: '',
      confirmPassword: '',
      username: ''
    },
    {
      email: [required, email],
      password: [required, minLength(8)],
      confirmPassword: [required, matchField('password')],
      username: [required, minLength(3)]
    }
  );

  const handleSubmit = (e) => {
    e.preventDefault();
    if (validateAll()) {
      console.log('Form is valid!', values);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Username</label>
        <input
          value={values.username}
          onChange={(e) => handleChange('username', e.target.value)}
          onBlur={() => handleBlur('username')}
        />
        {touched.username && errors.username && (
          <span className="error">{errors.username}</span>
        )}
      </div>

      <div>
        <label>Email</label>
        <input
          type="email"
          value={values.email}
          onChange={(e) => handleChange('email', e.target.value)}
          onBlur={() => handleBlur('email')}
        />
        {touched.email && errors.email && (
          <span className="error">{errors.email}</span>
        )}
      </div>

      <div>
        <label>Password</label>
        <input
          type="password"
          value={values.password}
          onChange={(e) => handleChange('password', e.target.value)}
          onBlur={() => handleBlur('password')}
        />
        {touched.password && errors.password && (
          <span className="error">{errors.password}</span>
        )}
      </div>

      <div>
        <label>Confirm Password</label>
        <input
          type="password"
          value={values.confirmPassword}
          onChange={(e) => handleChange('confirmPassword', e.target.value)}
          onBlur={() => handleBlur('confirmPassword')}
        />
        {touched.confirmPassword && errors.confirmPassword && (
          <span className="error">{errors.confirmPassword}</span>
        )}
      </div>

      <button type="submit">Sign Up</button>
    </form>
  );
}
```

**Key points:**
- Custom hook centralizes validation logic across multiple fields
- Touched state prevents showing errors before user interacts with field
- Validation rules are composable and reusable functions
- Cross-field validation (like password matching) has access to all values
- Only validates on blur to avoid annoying users while typing

</details>

<details>
<summary>Exercise 3: Notification System with Context and Reducer</summary>

```jsx
import { createContext, useContext, useReducer, useMemo, useCallback } from 'react';

const NotificationContext = createContext(null);

// Reducer
function notificationReducer(state, action) {
  switch (action.type) {
    case 'ADD_NOTIFICATION':
      return [...state, action.notification];

    case 'REMOVE_NOTIFICATION':
      return state.filter(notification => notification.id !== action.id);

    case 'CLEAR_ALL':
      return [];

    default:
      return state;
  }
}

// Provider
function NotificationProvider({ children }) {
  const [notifications, dispatch] = useReducer(notificationReducer, []);

  const addNotification = useCallback((message, type = 'info', duration = 3000) => {
    const notification = {
      id: `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`,
      message,
      type
    };
    dispatch({ type: 'ADD_NOTIFICATION', notification });

    if (duration > 0) {
      setTimeout(() => {
        dispatch({ type: 'REMOVE_NOTIFICATION', id: notification.id });
      }, duration);
    }
  }, []);

  const removeNotification = useCallback((id) => {
    dispatch({ type: 'REMOVE_NOTIFICATION', id });
  }, []);

  const clearAll = useCallback(() => {
    dispatch({ type: 'CLEAR_ALL' });
  }, []);

  const value = useMemo(
    () => ({
      notifications,
      addNotification,
      removeNotification,
      clearAll
    }),
    [notifications, addNotification, removeNotification, clearAll]
  );

  return (
    <NotificationContext.Provider value={value}>
      {children}
    </NotificationContext.Provider>
  );
}

// Custom hook
function useNotifications() {
  const context = useContext(NotificationContext);
  if (!context) {
    throw new Error('useNotifications must be used within NotificationProvider');
  }
  return context;
}

// Notification display component
function NotificationList() {
  const { notifications, removeNotification } = useNotifications();

  if (notifications.length === 0) return null;

  return (
    <div className="notification-container">
      {notifications.map(notification => (
        <div
          key={notification.id}
          className={`notification notification-${notification.type}`}
          role="alert"
        >
          <span>{notification.message}</span>
          <button onClick={() => removeNotification(notification.id)}>
            ×
          </button>
        </div>
      ))}
    </div>
  );
}

// Usage
function App() {
  return (
    <NotificationProvider>
      <NotificationList />
      <MyApp />
    </NotificationProvider>
  );
}

function MyApp() {
  const { addNotification } = useNotifications();

  const handleSuccess = () => {
    addNotification('Changes saved successfully!', 'success');
  };

  const handleError = () => {
    addNotification('Something went wrong', 'error', 5000);
  };

  return (
    <div>
      <button onClick={handleSuccess}>Save</button>
      <button onClick={handleError}>Trigger Error</button>
    </div>
  );
}
```

**Key points:**
- Auto-dismiss notifications after duration using setTimeout
- useCallback prevents recreating functions on every render
- Type parameter allows different notification styles (success, error, warning)
- Unique IDs generated with Date.now() for simple key management
- ARIA role="alert" announces notifications to screen readers

</details>

---

## What You Learned

This module covered:

- **State Location**: Local state stays in one component, shared state lifts to common ancestor, global state uses Context
- **Lifting State Up**: Moving state to parent components when siblings need to share data
- **Context API**: Provides global state without passing props through every level
- **useReducer**: Manages complex state with multiple update types using reducer pattern
- **Custom State Hooks**: Combines Context, useReducer, and custom logic for reusable domain-specific state
- **When to Use Libraries**: useState + Context works for most apps; consider Redux/Zustand for complex needs

**Key takeaway**: Keep state as local as possible, lift when needed, and use Context for truly global concerns like auth or themes.

---

## Real-World Application

This week at work, you might use these concepts to:

- Build a shopping cart that persists across pages using Context and useReducer
- Create an authentication provider that shares user state across the entire app
- Implement a multi-step form where each step shares validation state with the parent
- Design a notification system where any component can trigger toasts
- Refactor prop drilling by moving shared state into Context providers

---

## Further Reading

- [React.dev - Sharing State Between Components](https://react.dev/learn/sharing-state-between-components)
- [React.dev - Scaling Up with Reducer and Context](https://react.dev/learn/scaling-up-with-reducer-and-context)

---

**Navigation**: [← Previous Module](./03-hooks-deep-dive.md) | [Next Module →](./05-forms-and-validation.md)
