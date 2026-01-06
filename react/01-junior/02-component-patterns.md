# Component Patterns

> **Last reviewed**: 2026-01-06


## Week Two: The Dashboard Project

You survived your first week. The product catalog is live, customers are browsing, and Sarah gave you a thumbs up in standup. But now there's a new challenge on the board.

"You're building a streaming dashboard," Sarah announces, pulling up mockups. "Think Netflix meets Spotify. Content rows, tabbed sections, video players. The design team went all out."

Marcus leans over. "The trick is making components that don't fight each other. You'll see why."

You look at the mockups. Dozens of cards, multiple layouts, tabs everywhere. If you hardcode each one, you'll be writing components forever.

"There's a better way," Sarah says, noticing your expression. "Here are some patterns."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Composition** | Nesting gift boxes - smaller pieces inside larger ones, each complete on its own |
| **Children prop** | Gift wrapping - the wrapper doesn't care what's inside, just provides the presentation |
| **Controlled components** | A puppet on strings - parent controls every movement, full control but more work |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Junior Module 01 (React Fundamentals) - Understanding JSX, props, and basic components.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Explain composition over inheritance and why React favors it
- [ ] Build flexible components using the children prop pattern
- [ ] Create multi-slot layouts using named props for different content areas
- [ ] Implement compound components that share implicit state through Context
- [ ] Design controlled vs uncontrolled components and choose appropriately
- [ ] Refactor rigid component hierarchies into composable patterns

---

## Time Estimate

- **Reading**: 50-60 minutes
- **Exercises**: 2-3 hours
- **Mastery**: Practice these patterns over 2-3 weeks as you build features

---

## Chapter 1: The Inheritance Trap

You start with the media cards. Movies need one layout, TV series need another, podcasts need a third. Your first instinct: create a base card and extend it.

"That look says it all," Marcus says. "You're thinking inheritance. BaseCard, MovieCard extends BaseCard..."

You nod.

"React doesn't work that way. Watch what happens."

```jsx
// Your first instinct (don't do this)
// Note: Class-based inheritance is shown here as an anti-pattern
// We use function components throughout this curriculum
class FancyButton extends Button { ... } // ❌ Inheritance - fights against React

// What React wants instead
function FancyButton({ children, ...props }) {
  return (
    <Button className="fancy" {...props}>
      <Sparkle />
      {children}
    </Button>
  );
}
```

"Composition over inheritance," Marcus explains. "Instead of *being* a Button, FancyButton *uses* a Button. It wraps it, adds sparkle, passes everything else through."

The difference is subtle but powerful. Inheritance creates rigid hierarchies. Composition lets you mix and match.

```
┌──────────────────────────────────────────────────────────┐
│        Composition vs Inheritance                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  INHERITANCE (avoid in React):                           │
│  ┌─────────────┐                                         │
│  │   Button    │                                         │
│  └──────┬──────┘                                         │
│         │                                                │
│    ┌────┴────┐                                           │
│    ↓         ↓                                           │
│  Primary  Secondary                                      │
│    ↓                                                     │
│  Large                                                   │
│                                                          │
│  Problems:                                               │
│  • Rigid hierarchy - can't easily combine features       │
│  • Changes at top affect all children                    │
│  • Hard to reuse specific behaviors                      │
│                                                          │
│  ─────────────────────────────────────────────────────   │
│                                                          │
│  COMPOSITION (React way):                                │
│  ┌──────────────────────────────────────┐                │
│  │         <Button>                     │                │
│  │           <Icon />                   │                │
│  │           Click me                   │                │
│  │         </Button>                    │                │
│  └──────────────────────────────────────┘                │
│                                                          │
│  Mix features freely:                                    │
│  <Primary><Large>Text</Large></Primary>                  │
│  <Button variant="primary" size="large">Text</Button>    │
│                                                          │
│  Benefits:                                               │
│  • Flexible - combine any features                       │
│  • Testable - each piece independent                     │
│  • Reusable - pieces work anywhere                       │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Chapter 2: The Magic of Children

Sarah pulls up the first pattern for your media cards.

"Here's the trick. Don't make your card care about what's inside it."

```jsx
function Card({ children, title }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-content">{children}</div>
    </div>
  );
}

// Now it can hold anything
<Card title="Welcome">
  <p>This is the card content.</p>
  <button>Click me</button>
</Card>
```

"See? The Card doesn't know it contains a paragraph and button. It just renders whatever you put between the tags. That's the `children` prop."

You try it with a media card:

```jsx
function MediaCard({ children, thumbnail, onPlay }) {
  return (
    <div className="media-card">
      <div className="thumbnail" onClick={onPlay}>
        <img src={thumbnail} alt="" />
        <PlayButton />
      </div>
      <div className="media-info">{children}</div>
    </div>
  );
}
```

Now the same component works for everything:

```jsx
// Movie card
<MediaCard thumbnail={movie.poster} onPlay={() => playMovie(movie.id)}>
  <h3>{movie.title}</h3>
  <p>{movie.year} • {movie.duration}</p>
  <RatingBadge value={movie.rating} />
</MediaCard>

// Series card - different content, same wrapper
<MediaCard thumbnail={series.poster} onPlay={() => playSeries(series.id)}>
  <h3>{series.title}</h3>
  <p>{series.seasons} Seasons • {series.episodes} Episodes</p>
  <ProgressBar current={series.watchedEpisodes} total={series.episodes} />
</MediaCard>
```

"One component, infinite variations," Sarah says. "That's the power of children."

---

## Chapter 3: When Children Aren't Enough

The dashboard layout is more complex. It needs a header, sidebar, main content, and footer—all in specific positions.

"Children works for one slot," Marcus says. "But you need four."

He shows you the slots pattern:

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
```

Each prop is a slot where you can plug in components:

```jsx
<Layout
  header={<NavBar />}
  sidebar={<Menu />}
  footer={<Copyright />}
>
  <PageContent />
</Layout>
```

You build the browse page this way:

```jsx
function BrowsePage() {
  return (
    <DashboardLayout
      header={<Navigation user={currentUser} />}
      hero={<FeaturedContent item={featured} />}
      sidebar={<GenreList genres={genres} />}
    >
      <ContentRow title="Continue Watching">
        {continueWatching.map(item => (
          <MediaCard key={item.id} {...item} />
        ))}
      </ContentRow>

      <ContentRow title="Trending Now">
        {trending.map(item => (
          <MediaCard key={item.id} {...item} />
        ))}
      </ContentRow>
    </DashboardLayout>
  );
}
```

The layout handles positioning. The content handles... content. Clean separation.

---

## Chapter 4: Components That Talk to Each Other

"Now for the tabs," Sarah says. This is the trickiest part of the dashboard—a tabbed interface for Browse, My List, and Search.

"You could pass state through props," Marcus suggests, "but watch this."

```jsx
// Create a Context for the tabs to share state
const TabsContext = createContext();

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
```

"These are compound components," Sarah explains. "They share state through Context, but the API is beautiful."

```jsx
<Tabs defaultTab="browse">
  <TabList>
    <Tab id="browse">Browse</Tab>
    <Tab id="mylist">My List</Tab>
    <Tab id="search">Search</Tab>
  </TabList>
  <TabPanel id="browse">Browse content here</TabPanel>
  <TabPanel id="mylist">Your saved items</TabPanel>
  <TabPanel id="search">Search results</TabPanel>
</Tabs>
```

"The magic is what's *not* there," Marcus points out. "No prop drilling. No wiring up callbacks. The components just... work together."

Here's how state flows through compound components:

```
┌──────────────────────────────────────────────────────────┐
│        Compound Components: Tabs State Flow              │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  <Tabs defaultTab="browse">     ← Parent holds state     │
│    │                                                     │
│    └─ State: { activeTab: "browse" }                     │
│    └─ Context: Shares state with all children            │
│         │                                                │
│         ├──→ <TabList>                                   │
│         │      │                                         │
│         │      ├──→ <Tab id="browse">                    │
│         │      │      • Reads: activeTab from context    │
│         │      │      • Renders: active if match         │
│         │      │      • Calls: setActiveTab on click     │
│         │      │                                         │
│         │      ├──→ <Tab id="mylist">                    │
│         │      │      • Reads: activeTab from context    │
│         │      │      • Renders: inactive (no match)     │
│         │      │                                         │
│         │      └──→ <Tab id="search">                    │
│         │             • Same pattern...                  │
│         │                                                │
│         ├──→ <TabPanel id="browse">                      │
│         │      • Reads: activeTab from context           │
│         │      • Renders: visible (match)                │
│         │                                                │
│         ├──→ <TabPanel id="mylist">                      │
│         │      • Renders: hidden (no match)              │
│         │                                                │
│         └──→ <TabPanel id="search">                      │
│                • Renders: hidden (no match)              │
│                                                          │
│  Key: Context eliminates prop drilling!                  │
│  All children access state without explicit passing      │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Chapter 5: Who's in Control?

The video player is your final challenge. It needs play/pause, progress tracking, and volume control. But there's a decision to make.

"Controlled or uncontrolled?" Sarah asks.

You've heard these terms but never understood the difference.

**Controlled means the parent manages state:**

```jsx
function ControlledInput({ value, onChange }) {
  return <input value={value} onChange={onChange} />;
}

// Parent owns the data
const [text, setText] = useState('');
<ControlledInput value={text} onChange={e => setText(e.target.value)} />
```

**Uncontrolled means the component manages its own state:**

```jsx
function UncontrolledInput({ defaultValue }) {
  const inputRef = useRef();
  return <input ref={inputRef} defaultValue={defaultValue} />;
}

// Read value imperatively when needed
inputRef.current.value
```

"For the video player, you want controlled," Sarah says. "You need to sync the progress bar, save watch history, handle keyboard shortcuts. The parent needs to know everything."

```jsx
function VideoPlayer({ src, currentTime, onTimeUpdate, isPlaying, onPlayPause }) {
  const videoRef = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      videoRef.current.play();
    } else {
      videoRef.current.pause();
    }
  }, [isPlaying]);

  return (
    <div className="video-player">
      <video
        ref={videoRef}
        src={src}
        onTimeUpdate={(e) => onTimeUpdate(e.target.currentTime)}
      />
      <PlayerControls
        currentTime={currentTime}
        isPlaying={isPlaying}
        onPlayPause={onPlayPause}
      />
    </div>
  );
}
```

**When to use which:**
- **Controlled**: Forms with validation, synced UI, undo/redo, when parent needs the data
- **Uncontrolled**: File inputs, third-party integrations, simple forms where you only need values on submit

---

## Chapter 6: The Dashboard Takes Shape

By Friday, you've built the entire dashboard using these patterns:

```jsx
// ContentRow.jsx - Compound pattern for scrolling sections
function ContentRow({ children, title }) {
  return (
    <section className="content-row">
      <h2>{title}</h2>
      <div className="scroll-container">{children}</div>
    </section>
  );
}

// The full browse page comes together
function BrowsePage() {
  return (
    <DashboardLayout
      header={<Navigation user={currentUser} />}
      hero={<FeaturedContent item={featured} />}
      sidebar={<GenreList genres={genres} />}
    >
      <ContentRow title="Continue Watching">
        {continueWatching.map(item => (
          <MediaCard key={item.id} thumbnail={item.poster} onPlay={() => play(item)}>
            <h3>{item.title}</h3>
            <ProgressBar value={item.progress} />
          </MediaCard>
        ))}
      </ContentRow>

      <ContentRow title="Trending Now">
        {trending.map(item => (
          <MediaCard key={item.id} thumbnail={item.poster} onPlay={() => play(item)}>
            <h3>{item.title}</h3>
            <RatingBadge value={item.rating} />
          </MediaCard>
        ))}
      </ContentRow>

      <ContentRow title="New Releases">
        {newReleases.map(item => (
          <MediaCard key={item.id} thumbnail={item.poster} onPlay={() => play(item)}>
            <h3>{item.title}</h3>
            <span className="badge">NEW</span>
          </MediaCard>
        ))}
      </ContentRow>
    </DashboardLayout>
  );
}
```

Sarah reviews your code. "Clean. Flexible. You could add a podcast section in five minutes without changing any existing components."

"That's the point, right?" you say.

She nods. "Now you're thinking in patterns."

---

## Patterns Reference

| Pattern | Use When | Example |
|---------|----------|---------|
| **Children** | Single content slot needed | Card, Modal, Container |
| **Slots** | Multiple named content areas | Layout, Dialog with header/body/footer |
| **Compound** | Components share implicit state | Tabs, Accordion, Menu |
| **Controlled** | Parent needs full control | Forms, Players, Editors |
| **Uncontrolled** | Self-contained, simple cases | File upload, Quick forms |

## Common Mistakes

1. **Over-abstracting too early** - Start simple, extract patterns when you see repetition
2. **Prop drilling through many layers** - Use Context for deep trees
3. **Mixing controlled and uncontrolled** - Pick one approach and stick with it


Example:
```jsx
// ❌ Inheritance couples components tightly
class MovieCard extends Card {}

// ✅ Composition keeps components flexible
function MovieCard({ movie }) {
  return (
    <Card>
      <CardHeader>{movie.title}</CardHeader>
      <CardBody>{movie.summary}</CardBody>
    </Card>
  );
}
```

---

## Debug This Code

Before moving to the exercises, test your debugging skills. Each snippet has bugs - can you spot and fix them?

<details>
<summary>Challenge 1: Compound Component State</summary>

```jsx
// Tabs compound component with a subtle bug
const TabsContext = createContext();

function Tabs({ children }) {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <TabsContext.Provider value={activeTab, setActiveTab}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function Tab({ index, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);

  return (
    <button
      className={activeTab === index ? 'active' : ''}
      onClick={() => setActiveTab(index)}
    >
      {children}
    </button>
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

Check how the Context value is being passed. Is the syntax correct?

</details>

<details>
<summary>Solution</summary>

**Bug**: Context value should be an object, but it's using incorrect syntax - `value={activeTab, setActiveTab}` is invalid.

**Fixed code:**

```jsx
const TabsContext = createContext();

function Tabs({ children }) {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function Tab({ index, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);

  return (
    <button
      className={activeTab === index ? 'active' : ''}
      onClick={() => setActiveTab(index)}
    >
      {children}
    </button>
  );
}
```

**Why it matters**: The Context Provider's `value` prop expects a single value. To pass multiple values, wrap them in an object.

</details>

</details>

<details>
<summary>Challenge 2: Controlled vs Uncontrolled Input</summary>

```jsx
function SearchBar({ onSearch }) {
  const [value, setValue] = useState('');

  return (
    <input
      type="text"
      value={value}
      placeholder="Search products..."
      onChange={(e) => {
        onSearch(e.target.value);
      }}
    />
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

The component has state, but is it updating correctly? What happens when you type?

</details>

<details>
<summary>Solution</summary>

**Bug**: State is declared but never updated - `setValue` is never called, so the input won't respond to typing.

**Fixed code:**

```jsx
function SearchBar({ onSearch }) {
  const [value, setValue] = useState('');

  return (
    <input
      type="text"
      value={value}
      placeholder="Search products..."
      onChange={(e) => {
        setValue(e.target.value);
        onSearch(e.target.value);
      }}
    />
  );
}
```

**Alternative** (if the input should be fully controlled by parent):

```jsx
function SearchBar({ value, onChange }) {
  return (
    <input
      type="text"
      value={value}
      placeholder="Search products..."
      onChange={onChange}
    />
  );
}
```

**Why it matters**: A controlled input with a `value` prop must also update that value in `onChange`, otherwise the input becomes read-only.

</details>

</details>

<details>
<summary>Challenge 3: Render Props Memory Leak</summary>

```jsx
function DataFetcher({ url, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [url]);

  return render({ data, loading });
}

// Usage
<DataFetcher
  url="/api/products"
  render={({ data, loading }) => (
    loading ? <Spinner /> : <ProductList products={data} />
  )}
/>
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

What happens if the component unmounts while the fetch is still in flight?

</details>

<details>
<summary>Solution</summary>

**Bug**: No cleanup for the async operation. If the component unmounts during fetch, it will try to call `setData` and `setLoading` on an unmounted component.

**Fixed code:**

```jsx
function DataFetcher({ url, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let cancelled = false;

    setLoading(true);
    fetch(url)
      .then(res => res.json())
      .then(fetchedData => {
        if (!cancelled) {
          setData(fetchedData);
          setLoading(false);
        }
      });

    return () => {
      cancelled = true;
    };
  }, [url]);

  return render({ data, loading });
}

// Usage
<DataFetcher
  url="/api/products"
  render={({ data, loading }) => (
    loading ? <Spinner /> : <ProductList products={data} />
  )}
/>
```

**Why it matters**: Setting state on unmounted components causes memory leaks and React warnings. Always cleanup async operations in useEffect.

</details>

</details>

---

## Practice Exercises

1. Build a `Modal` component using the children pattern
2. Create a compound `Accordion` component with `AccordionItem` children
3. Build a controlled `SearchInput` with debounced onChange

### Solutions

<details>
<summary>Exercise 1: Modal with Children Pattern</summary>

```jsx
function Modal({ isOpen, onClose, title, children }) {
  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        <div className="modal-header">
          <h2>{title}</h2>
          <button onClick={onClose} aria-label="Close">×</button>
        </div>
        <div className="modal-body">
          {children}
        </div>
      </div>
    </div>
  );
}

// Usage
function App() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsOpen(true)}>Open Modal</button>
      <Modal isOpen={isOpen} onClose={() => setIsOpen(false)} title="Confirm Action">
        <p>Are you sure you want to proceed?</p>
        <button onClick={() => setIsOpen(false)}>Cancel</button>
        <button onClick={handleConfirm}>Confirm</button>
      </Modal>
    </>
  );
}
```

**Key points:**
- Modal doesn't care what's inside - uses `children`
- `stopPropagation` prevents clicks inside modal from closing it
- Early return if not open (conditional rendering)
- Parent controls open/close state (controlled component)

</details>

<details>
<summary>Exercise 2: Compound Accordion Component</summary>

```jsx
const AccordionContext = createContext();

function Accordion({ children }) {
  const [openItem, setOpenItem] = useState(null);

  return (
    <AccordionContext.Provider value={{ openItem, setOpenItem }}>
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
}

function AccordionItem({ id, title, children }) {
  const { openItem, setOpenItem } = useContext(AccordionContext);
  const isOpen = openItem === id;

  return (
    <div className="accordion-item">
      <button
        className="accordion-header"
        onClick={() => setOpenItem(isOpen ? null : id)}
        aria-expanded={isOpen}
      >
        {title}
        <span>{isOpen ? '−' : '+'}</span>
      </button>
      {isOpen && (
        <div className="accordion-content">{children}</div>
      )}
    </div>
  );
}

// Usage
<Accordion>
  <AccordionItem id="1" title="Section 1">
    Content for section 1
  </AccordionItem>
  <AccordionItem id="2" title="Section 2">
    Content for section 2
  </AccordionItem>
  <AccordionItem id="3" title="Section 3">
    Content for section 3
  </AccordionItem>
</Accordion>
```

**Key points:**
- Context shares state between Accordion and AccordionItem
- Only one item open at a time (controlled by parent Accordion)
- Each item toggles itself by comparing its ID to openItem
- Clean API - no prop drilling

</details>

<details>
<summary>Exercise 3: Controlled SearchInput with Debounce</summary>

```jsx
function SearchInput({ value, onChange, placeholder = "Search..." }) {
  const [localValue, setLocalValue] = useState(value);
  const timeoutRef = useRef(null);

  // Update local value immediately for responsive UI
  const handleChange = (e) => {
    const newValue = e.target.value;
    setLocalValue(newValue);

    // Clear previous timeout
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }

    // Debounce the onChange callback (wait 300ms after user stops typing)
    timeoutRef.current = setTimeout(() => {
      onChange(newValue);
    }, 300);
  };

  // Cleanup timeout on unmount
  useEffect(() => {
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, []);

  return (
    <input
      type="search"
      value={localValue}
      onChange={handleChange}
      placeholder={placeholder}
    />
  );
}

// Usage
function ProductSearch() {
  const [searchTerm, setSearchTerm] = useState('');
  const results = useProductSearch(searchTerm); // API call happens here

  return (
    <div>
      <SearchInput value={searchTerm} onChange={setSearchTerm} />
      <SearchResults results={results} />
    </div>
  );
}
```

**Key points:**
- Local state for immediate UI updates
- `useRef` to store timeout ID (persists between renders)
- Debouncing reduces API calls - waits until user stops typing
- Cleanup function prevents memory leaks
- Parent still controls the value (controlled component)

</details>

---

## What You Learned

This module covered:

- **Composition Over Inheritance**: React's preferred way to reuse code by composing components rather than extending classes
- **Children Prop**: A flexible pattern that allows components to accept any content between their opening and closing tags
- **Multi-Slot Patterns**: Using named props to create layouts with multiple content areas (header, sidebar, footer, etc.)
- **Compound Components**: Components that work together and share implicit state through Context API
- **Controlled vs Uncontrolled**: Controlled components give parents full control over state; uncontrolled manage their own state

**Key takeaway**: Flexible components are built by composing smaller pieces together, not by creating rigid inheritance hierarchies.

---

## Real-World Application

This week at work, you might use these concepts to:

- Build a reusable Modal component where the parent controls what content appears inside
- Create a Dashboard layout with multiple named slots for header, sidebar, and main content
- Implement a Tabs component where Tab and TabPanel children automatically coordinate their state
- Design form components that can work as either controlled (for validation) or uncontrolled (for simple cases)
- Refactor a tightly-coupled component tree into smaller, composable pieces

---

## Further Reading

- [React.dev - Passing Props](https://react.dev/learn/passing-props-to-a-component)
- [Patterns.dev - Compound Pattern](https://www.patterns.dev/react/compound-pattern)

---

**Navigation**: [← Previous Module](./01-react-fundamentals.md) | [Next Module →](./03-hooks-deep-dive.md)
