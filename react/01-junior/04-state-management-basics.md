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

    if (!order.restaurantId) {
      dispatch({ type: 'SET_RESTAURANT', restaurantId });
    }

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
        <div key={index} className="order-item">
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

## Practice Exercises

1. Build a shopping cart with Context (add, remove, update quantity)
2. Create a form with multiple inputs sharing validation state
3. Implement a notification system with Context and reducer

## Further Reading

- [React.dev - Sharing State Between Components](https://react.dev/learn/sharing-state-between-components)
- [React.dev - Scaling Up with Reducer and Context](https://react.dev/learn/scaling-up-with-reducer-and-context)

---

**Navigation**: [← Previous Module](./03-hooks-deep-dive.md) | [Next Module →](./05-forms-and-validation.md)
