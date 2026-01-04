# Accessibility Fundamentals

## Week Ten: The Audit

Everything is going well. The features are polished, the tests are passing, the team is hitting deadlines. Then Monday morning, Sarah calls an emergency meeting.

"We have a problem," she says, pulling up an email. "GovServe wants to adopt our platform for their citizen services portal. But their procurement requires WCAG 2.1 AA compliance."

"What's WCAG?" you ask.

"Web Content Accessibility Guidelines. It's how we ensure people with disabilities can use our app." She pulls up an attached PDF. "They hired an accessibility consultant to audit us. Here are the results."

The report is sobering. 47 failures. Screen readers can't navigate the dashboard. Forms have no labels. The modal traps keyboard users with no escape. Color contrast fails on half the buttons.

Marcus looks uncomfortable. "I've never tested with a screen reader."

"Neither have I," Sarah admits. "But we have two weeks to fix this. Let's learn."

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Screen readers** | A tour guide reading aloud - they describe what's on screen for users who can't see it |
| **Semantic HTML** | Proper road signs - they tell assistive tech what each element actually is |
| **ARIA** | Translator notes - extra context when HTML alone isn't enough |
| **Focus management** | Spotlight control - guides keyboard users through the interface |

Keep these in mind. They'll click as we build.

---

## Chapter 1: Understanding the Audit

The consultant, Maya, joins a video call to walk through the failures.

"Let's start with severity levels," she says.

**WCAG Levels:**
- **Level A** - Minimum accessibility (must fix)
- **Level AA** - Standard compliance (required by most regulations)
- **Level AAA** - Enhanced accessibility (nice to have)

"GovServe requires Level AA. That's the standard for government and enterprise."

She shares her screen, showing the audit categories:

| Category | Failures | Severity |
|----------|----------|----------|
| Keyboard Navigation | 12 | Critical |
| Screen Reader | 15 | Critical |
| Forms & Labels | 8 | High |
| Color Contrast | 7 | High |
| Focus Management | 5 | Medium |

"The good news? Most of these are fixable with small changes. The bad news? You've been building inaccessible patterns for months."

---

## Chapter 2: Semantic HTML First

"Before we touch ARIA," Maya says, "let's fix the HTML. Most accessibility issues come from using the wrong elements."

```jsx
// BAD: Div soup - screen readers see nothing meaningful
<div className="nav">
  <div className="nav-item" onClick={handleClick}>Home</div>
  <div className="nav-item" onClick={handleClick}>Products</div>
</div>

// GOOD: Semantic HTML - screen readers understand the structure
<nav>
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/products">Products</a></li>
  </ul>
</nav>
```

"A screen reader announces 'navigation landmark' for `<nav>`. For the div version? Nothing. The user has no idea it's navigation."

**Semantic elements you should use:**

| Instead of... | Use... | Why |
|---------------|--------|-----|
| `<div>` for clickable | `<button>` | Keyboard accessible, announced as button |
| `<div>` for navigation | `<nav>` | Creates navigation landmark |
| `<div>` for main content | `<main>` | Creates main landmark |
| `<div>` for sections | `<section>` with heading | Creates document structure |
| `<span>` for links | `<a>` | Announced as link, keyboard accessible |

**Heading hierarchy matters too:**

```jsx
// BAD: Headings chosen for styling
<h1>Welcome</h1>
<h4>Latest Products</h4>  {/* Skipped h2, h3! */}
<h2>Contact Us</h2>

// GOOD: Logical hierarchy
<h1>Welcome</h1>
<h2>Latest Products</h2>
<h2>Contact Us</h2>
```

"Screen reader users navigate by headings. Skipping levels breaks their mental map of the page."

---

## Chapter 3: ARIA — When HTML Isn't Enough

"ARIA is powerful," Maya warns, "but it's also easy to misuse. The first rule of ARIA: don't use ARIA if you can use HTML."

```jsx
// BAD: ARIA where HTML would work
<div role="button" tabIndex={0} onClick={handleClick}>
  Submit
</div>

// GOOD: Just use a button
<button onClick={handleClick}>
  Submit
</button>
```

"But sometimes HTML isn't enough. That's when ARIA helps."

### ARIA Roles

```jsx
// Custom component needs role
<div role="alert">
  Your session will expire in 5 minutes.
</div>

// Tab interface
<div role="tablist">
  <button role="tab" aria-selected="true">Tab 1</button>
  <button role="tab" aria-selected="false">Tab 2</button>
</div>
<div role="tabpanel">Tab 1 content</div>
```

### ARIA States and Properties

```jsx
// Expanded/collapsed state
<button aria-expanded={isOpen} aria-controls="menu">
  Menu
</button>
<ul id="menu" hidden={!isOpen}>
  <li>Option 1</li>
</ul>

// Loading state
<button aria-busy={isLoading} disabled={isLoading}>
  {isLoading ? 'Saving...' : 'Save'}
</button>

// Descriptions and labels
<input
  aria-label="Search products"
  aria-describedby="search-hint"
/>
<p id="search-hint">Enter product name or SKU</p>
```

### Common ARIA Patterns

| Pattern | ARIA Attributes |
|---------|-----------------|
| Toggle button | `aria-pressed="true/false"` |
| Expandable section | `aria-expanded`, `aria-controls` |
| Modal dialog | `role="dialog"`, `aria-modal="true"`, `aria-labelledby` |
| Live updates | `aria-live="polite/assertive"` |
| Required field | `aria-required="true"` |
| Invalid field | `aria-invalid="true"`, `aria-describedby` |

---

## Chapter 4: Keyboard Navigation

"12 keyboard failures," Maya says. "This is critical. Many users can't use a mouse—motor disabilities, visual impairments, power users who prefer keyboard."

### Everything Must Be Reachable

```jsx
// BAD: Click-only interaction
<div className="card" onClick={() => setSelected(id)}>
  <img src={image} />
  <h3>{title}</h3>
</div>

// GOOD: Keyboard accessible
<button
  className="card"
  onClick={() => setSelected(id)}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      setSelected(id);
    }
  }}
>
  <img src={image} alt="" />
  <h3>{title}</h3>
</button>
```

"Actually, if you use `<button>`, you get keyboard handling for free. Enter and Space work automatically."

### Tab Order

```jsx
// tabIndex values
<button>First (natural order)</button>
<button tabIndex={0}>Also in natural order</button>
<button tabIndex={-1}>Removed from tab order (but focusable via JS)</button>
<button tabIndex={1}>DON'T DO THIS - positive tabIndex breaks flow</button>
```

**The rule: Never use positive tabIndex. It creates maintenance nightmares.**

### Skip Links

"Keyboard users shouldn't have to tab through 50 nav items to reach content."

```jsx
function Layout({ children }) {
  return (
    <>
      <a href="#main-content" className="skip-link">
        Skip to main content
      </a>
      <nav>
        {/* Navigation items */}
      </nav>
      <main id="main-content">
        {children}
      </main>
    </>
  );
}
```

```css
.skip-link {
  position: absolute;
  left: -10000px;
}

.skip-link:focus {
  left: 10px;
  top: 10px;
  z-index: 1000;
}
```

---

## Chapter 5: Focus Management and Modals

"Your modal is a focus trap," Maya says, "but not the good kind."

The current modal lets focus escape to the background. Users tab through invisible elements behind the overlay.

### Proper Focus Trapping

```jsx
function Modal({ isOpen, onClose, title, children }) {
  const modalRef = useRef(null);
  const previousFocus = useRef(null);

  useEffect(() => {
    if (isOpen) {
      // Save current focus
      previousFocus.current = document.activeElement;
      // Focus the modal
      modalRef.current?.focus();
    } else {
      // Restore focus when closing
      previousFocus.current?.focus();
    }
  }, [isOpen]);

  // Close on Escape
  useEffect(() => {
    function handleKeyDown(e) {
      if (e.key === 'Escape' && isOpen) {
        onClose();
      }
    }
    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        tabIndex={-1}
        onClick={(e) => e.stopPropagation()}
        className="modal"
      >
        <h2 id="modal-title">{title}</h2>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
}
```

### The `inert` Attribute

"There's an easier way to trap focus now," Maya adds.

```jsx
function App() {
  const [modalOpen, setModalOpen] = useState(false);

  return (
    <>
      <div inert={modalOpen ? '' : undefined}>
        {/* Main app content - inert when modal is open */}
        <header>...</header>
        <main>...</main>
      </div>

      {modalOpen && <Modal onClose={() => setModalOpen(false)} />}
    </>
  );
}
```

"The `inert` attribute makes content unfocusable and invisible to assistive tech. It's like the content doesn't exist while the modal is open."

---

## Chapter 6: Forms and Error Messages

"8 form failures," Maya continues. "Forms are where accessibility often breaks down."

### Labels Are Required

```jsx
// BAD: No label
<input type="email" placeholder="Enter email" />

// BAD: Placeholder is not a label (disappears when typing)
<input type="email" placeholder="Email address" />

// GOOD: Visible label
<label htmlFor="email">Email address</label>
<input type="email" id="email" />

// GOOD: Label wrapping (no id needed)
<label>
  Email address
  <input type="email" />
</label>

// OK: Visually hidden label (when design requires it)
<label htmlFor="search" className="visually-hidden">Search</label>
<input type="search" id="search" placeholder="Search..." />
```

### Error Messages

```jsx
function FormField({ label, error, children }) {
  const id = useId();
  const errorId = `${id}-error`;

  return (
    <div className="form-field">
      <label htmlFor={id}>{label}</label>
      {React.cloneElement(children, {
        id,
        'aria-invalid': error ? 'true' : undefined,
        'aria-describedby': error ? errorId : undefined,
      })}
      {error && (
        <span id={errorId} className="error" role="alert">
          {error}
        </span>
      )}
    </div>
  );
}

// Usage
<FormField label="Email" error={errors.email}>
  <input type="email" {...register('email')} />
</FormField>
```

### Live Regions for Dynamic Content

```jsx
// Announce changes to screen readers
<div aria-live="polite" aria-atomic="true">
  {submitStatus === 'success' && 'Form submitted successfully!'}
  {submitStatus === 'error' && 'Submission failed. Please try again.'}
</div>
```

**Live region options:**
- `aria-live="polite"` - Announces after current speech finishes
- `aria-live="assertive"` - Interrupts immediately (use sparingly)
- `aria-atomic="true"` - Announces entire region, not just changes

---

## Chapter 7: Testing Accessibility

"You can't fix what you can't find," Maya says. "Here's how to test."

### Manual Testing Checklist

```markdown
## Keyboard Testing
- [ ] Can you reach all interactive elements with Tab?
- [ ] Can you activate buttons with Enter/Space?
- [ ] Can you close modals with Escape?
- [ ] Is focus visible at all times?
- [ ] Is tab order logical?

## Screen Reader Testing
- [ ] Are all images described (or marked decorative)?
- [ ] Do form fields have labels?
- [ ] Are error messages announced?
- [ ] Can you navigate by headings?
- [ ] Are live updates announced?

## Visual Testing
- [ ] Is color contrast sufficient (4.5:1 for text)?
- [ ] Does the page work at 200% zoom?
- [ ] Is information conveyed by more than just color?
```

### Automated Testing with axe-core

```jsx
// In your test setup
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('Button', () => {
  it('should have no accessibility violations', async () => {
    const { container } = render(<Button>Click me</Button>);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});

// Testing specific components
describe('LoginForm', () => {
  it('should have accessible form fields', async () => {
    const { container } = render(<LoginForm />);
    const results = await axe(container);

    expect(results).toHaveNoViolations();
  });

  it('should announce errors to screen readers', async () => {
    const { container, getByRole } = render(<LoginForm />);

    // Submit empty form
    fireEvent.click(getByRole('button', { name: 'Login' }));

    // Check error is announced
    expect(getByRole('alert')).toHaveTextContent('Email is required');
  });
});
```

### Browser DevTools

```jsx
// Chrome DevTools - Accessibility panel
// 1. Open DevTools (F12)
// 2. Go to Elements panel
// 3. Select "Accessibility" tab
// 4. Inspect any element to see its accessibility tree

// Lighthouse accessibility audit
// 1. Open DevTools
// 2. Go to Lighthouse panel
// 3. Check "Accessibility" category
// 4. Run audit
```

### Screen Reader Testing

"Test with real screen readers," Maya emphasizes.

| Platform | Screen Reader | Cost |
|----------|---------------|------|
| macOS | VoiceOver | Free (built-in) |
| Windows | NVDA | Free |
| Windows | JAWS | Paid |
| iOS | VoiceOver | Free (built-in) |
| Android | TalkBack | Free (built-in) |

"Start with VoiceOver on Mac. Cmd+F5 to toggle. Then try NVDA on Windows."

---

## Chapter 8: Color and Contrast

"7 contrast failures," Maya says. "These are easy to fix but easy to miss."

### WCAG Contrast Requirements

| Text Size | Minimum Ratio (AA) | Enhanced Ratio (AAA) |
|-----------|-------------------|---------------------|
| Normal text | 4.5:1 | 7:1 |
| Large text (18px+ bold, 24px+) | 3:1 | 4.5:1 |
| UI components | 3:1 | 3:1 |

```jsx
// BAD: Light gray on white - 2.5:1 contrast
<p style={{ color: '#999999', background: '#ffffff' }}>
  Hard to read
</p>

// GOOD: Darker gray - 7:1 contrast
<p style={{ color: '#595959', background: '#ffffff' }}>
  Easy to read
</p>
```

### Don't Rely on Color Alone

```jsx
// BAD: Error indicated only by color
<input style={{ borderColor: hasError ? 'red' : 'gray' }} />

// GOOD: Error indicated by color AND icon AND text
<div className="form-field">
  <input
    style={{ borderColor: hasError ? 'red' : 'gray' }}
    aria-invalid={hasError}
  />
  {hasError && (
    <span className="error">
      <ErrorIcon aria-hidden="true" />
      {errorMessage}
    </span>
  )}
</div>
```

### Focus Indicators

```css
/* BAD: Removing focus outline */
*:focus {
  outline: none; /* NEVER DO THIS */
}

/* GOOD: Custom focus style */
*:focus {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}

/* BETTER: Focus-visible for keyboard only */
*:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```

---

## Chapter 9: The Fixes Come Together

By the end of the second week, you've addressed all 47 failures:

```jsx
// Before: Inaccessible dashboard card
<div className="card" onClick={() => selectCard(id)}>
  <div className="card-image">
    <img src={image} />
  </div>
  <div className="card-title">{title}</div>
  <div className="card-status" style={{ color: status.color }}>
    {status.label}
  </div>
</div>

// After: Fully accessible
<article className="card">
  <h3 className="card-title">{title}</h3>
  <img src={image} alt={`${title} preview`} />
  <p className="card-status">
    <StatusIcon status={status.type} aria-hidden="true" />
    <span className={`status-${status.type}`}>{status.label}</span>
  </p>
  <button
    onClick={() => selectCard(id)}
    aria-describedby={`card-${id}-status`}
  >
    View details
  </button>
  <span id={`card-${id}-status`} className="visually-hidden">
    Status: {status.label}
  </span>
</article>
```

Maya runs the audit again. Zero failures.

"Congratulations," she says. "You've built an accessible application. But remember—accessibility isn't a checkbox. It's an ongoing practice."

Sarah adds: "From now on, we test with keyboard and screen reader before every release."

Marcus nods. "I'm adding jest-axe to our CI pipeline today."

The GovServe deal closes the following week.

---

## Accessibility Checklist

| Area | Quick Check |
|------|-------------|
| Semantic HTML | Using `<button>`, `<nav>`, `<main>`, `<header>`? |
| Headings | Logical hierarchy (h1 → h2 → h3)? |
| Images | Alt text for meaningful, `alt=""` for decorative? |
| Forms | All inputs have labels? |
| Errors | Announced with `role="alert"` or `aria-live`? |
| Keyboard | All interactive elements reachable with Tab? |
| Focus | Visible focus indicator on all elements? |
| Contrast | 4.5:1 for text, 3:1 for UI? |
| Modals | Focus trapped, Escape closes, focus restored? |

## Common Mistakes

1. **Using divs for everything** - Use semantic HTML first
2. **Hiding focus outlines** - Users need to see where focus is
3. **Placeholder as label** - Placeholders disappear when typing
4. **Color as only indicator** - Use icons and text too
5. **Skipping heading levels** - Breaks screen reader navigation
6. **Positive tabIndex** - Breaks natural tab order

## Practice Exercises

1. Run an axe audit on a page and fix the failures
2. Navigate a complex form using only keyboard
3. Use VoiceOver or NVDA to browse a website you built

## Further Reading

- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [MDN Accessibility Guide](https://developer.mozilla.org/en-US/docs/Web/Accessibility)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Deque University](https://dequeuniversity.com/)
