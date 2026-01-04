# Component Patterns

## Week Two: The Dashboard Project

You survived your first week. The product catalog is live, customers are browsing, and Sarah gave you a thumbs up in standup. But now there's a new challenge on the board.

"We're building a streaming dashboard," Sarah announces, pulling up mockups. "Think Netflix meets Spotify. Content rows, tabbed sections, video players. The design team went all out."

Marcus leans over. "The trick is making components that don't fight each other. You'll see what I mean."

You look at the mockups. Dozens of cards, multiple layouts, tabs everywhere. If you hardcode each one, you'll be writing components forever.

"There's a better way," Sarah says, noticing your expression. "Let me show you some patterns."

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Composition** | Nesting gift boxes - smaller pieces inside larger ones, each complete on its own |
| **Children prop** | Gift wrapping - the wrapper doesn't care what's inside, just provides the presentation |
| **Controlled components** | A puppet on strings - parent controls every movement, full control but more work |

Keep these in mind. They'll click as we build.

---

## Chapter 1: The Inheritance Trap

You start with the media cards. Movies need one layout, TV series need another, podcasts need a third. Your first instinct: create a base card and extend it.

"I see that look," Marcus says. "You're thinking inheritance. BaseCard, MovieCard extends BaseCard..."

You nod.

"React doesn't work that way. Watch what happens."

```jsx
// Your first instinct (don't do this)
class FancyButton extends Button { ... } // Inheritance - fights against React

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

## Practice Exercises

1. Build a `Modal` component using the children pattern
2. Create a compound `Accordion` component with `AccordionItem` children
3. Build a controlled `SearchInput` with debounced onChange

## Further Reading

- [React.dev - Passing Props](https://react.dev/learn/passing-props-to-a-component)
- [Patterns.dev - Compound Pattern](https://www.patterns.dev/react/compound-pattern)

---

**Navigation**: [← Previous Module](./01-react-fundamentals.md) | [Next Module →](./03-hooks-deep-dive.md)
