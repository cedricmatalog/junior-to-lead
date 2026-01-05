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

## Prerequisites

Module 09 (Git Workflows) - Understanding collaboration practices and code review to ship accessible features.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Use semantic HTML elements (nav, main, button) instead of divs for better screen reader support
- [ ] Apply ARIA attributes (roles, labels, live regions) when HTML isn't sufficient
- [ ] Ensure all interactive elements are keyboard accessible with proper focus management
- [ ] Implement focus trapping in modals and restore focus on close
- [ ] Create accessible forms with proper labels, error messages, and validation feedback
- [ ] Test with automated tools (axe, Lighthouse) and manual screen reader testing

---

## Time Estimate

- **Reading**: 65-80 minutes
- **Exercises**: 4-5 hours
- **Mastery**: Build accessibility into every feature over 6-8 weeks to make it habitual

---

## Chapter 1: Understanding the Audit

The consultant, Maya, joins a video call to walk through the failures.

"Let's start with severity levels," she says.

**WCAG Levels:**
- **Level A** - Minimum accessibility (must fix)
- **Level AA** - Standard compliance (required by most regulations)
- **Level AAA** - Enhanced accessibility (nice to have)

"GovServe requires Level AA. That's the standard for government and enterprise."

Here's the WCAG compliance pyramid:

```
┌──────────────────────────────────────────────────────────┐
│            WCAG Compliance Pyramid                       │
├──────────────────────────────────────────────────────────┤
│                        /\                                │
│                       /  \                               │
│                      /    \      Level AAA               │
│                     / En   \     Enhanced                │
│                    / hanced \   • Highest standards      │
│                   /          \  • Sign language          │
│                  /    AAA     \ • Extended audio desc.   │
│                 /──────────────\                         │
│                /                \                        │
│               /    Level AA      \                       │
│              /                    \                      │
│             /   Standard Complianc \                     │
│            /  • Required by most    \                    │
│           /    regulations           \                   │
│          /  • 4.5:1 contrast          \                  │
│         /   • Keyboard accessible      \                 │
│        /    • ARIA labels               \                │
│       /──────────────────────────────────\               │
│      /                                    \              │
│     /           Level A                    \             │
│    /         Minimum                        \            │
│   /  • Basic accessibility                   \           │
│  /   • Alt text for images                    \          │
│ /    • Keyboard navigation                     \         │
│/       • Proper heading structure               \        │
│──────────────────────────────────────────────────        │
│                                                          │
│  Most organizations require: Level AA                    │
│  Government contracts often mandate: WCAG 2.1 AA         │
│                                                          │
│  Each level includes all lower levels                    │
│  (AA includes all A requirements)                        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

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

Here's how keyboard navigation should flow through your app:

```
┌──────────────────────────────────────────────────────────┐
│         Keyboard Navigation Flow                         │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  User presses Tab key:                                   │
│       │                                                  │
│       ↓                                                  │
│  ┌─────────────────────────┐                             │
│  │ Skip to main content    │ ← Skip link (Tab 1)         │
│  └─────────────────────────┘                             │
│       │ (User presses Enter to skip, or Tab to nav)      │
│       ↓                                                  │
│  ┌──────────────────────────────────────────┐            │
│  │  Navigation                              │            │
│  │  ├─ Logo (link)         Tab 2            │            │
│  │  ├─ Home (link)         Tab 3            │            │
│  │  ├─ Products (link)     Tab 4            │            │
│  │  ├─ About (link)        Tab 5            │            │
│  │  └─ Search (input)      Tab 6            │            │
│  └──────────────────────────────────────────┘            │
│       │                                                  │
│       ↓                                                  │
│  ┌──────────────────────────────────────────┐            │
│  │  Main Content (id="main-content")        │            │
│  │  ├─ Heading (not focusable)              │            │
│  │  ├─ "Add to Cart" button    Tab 7        │            │
│  │  ├─ Quantity select         Tab 8        │            │
│  │  ├─ "Buy Now" button        Tab 9        │            │
│  │  └─ Product link            Tab 10       │            │
│  └──────────────────────────────────────────┘            │
│       │                                                  │
│       ↓                                                  │
│  ┌──────────────────────────────────────────┐            │
│  │  Footer                                  │            │
│  │  ├─ Privacy link          Tab 11         │            │
│  │  └─ Contact link          Tab 12         │            │
│  └──────────────────────────────────────────┘            │
│       │                                                  │
│       ↓                                                  │
│  Back to top (loops to skip link)                        │
│                                                          │
│  Key interactions:                                       │
│  • Tab      → Next focusable element                     │
│  • Shift+Tab → Previous focusable element                │
│  • Enter    → Activate links/buttons                     │
│  • Space    → Activate buttons, toggle checkboxes        │
│  • Escape   → Close modals/menus                         │
│  • Arrows   → Navigate within components (tabs, menu)    │
│                                                          │
│  Focus must be visible at all times!                     │
│  Order must follow visual/logical flow                   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

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

### Solutions

<details>
<summary>Exercise 1: Run axe Audit and Fix Failures</summary>

**Install axe DevTools:**
1. Install axe DevTools extension in Chrome or Firefox
2. Open your app in the browser
3. Open DevTools (F12)
4. Click on the "axe DevTools" tab
5. Click "Scan ALL of my page"

**Common failures and fixes:**

**Failure 1: Images missing alt text**
```jsx
// BEFORE (Fails)
<img src="/logo.png" />

// AFTER (Passes)
<img src="/logo.png" alt="Company Name Logo" />

// Decorative images
<img src="/decoration.svg" alt="" role="presentation" />
```

**Failure 2: Form inputs without labels**
```jsx
// BEFORE (Fails)
<input type="email" placeholder="Email" />

// AFTER (Passes) - Option 1: Visible label
<label htmlFor="email">Email Address</label>
<input type="email" id="email" />

// Option 2: aria-label
<input type="email" aria-label="Email Address" />

// Option 3: Visually hidden label
<label htmlFor="email" className="sr-only">Email Address</label>
<input type="email" id="email" placeholder="you@example.com" />

// CSS for sr-only:
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

**Failure 3: Low color contrast**
```jsx
// BEFORE (Fails) - 2.5:1 contrast ratio
<p style={{ color: '#999', background: '#fff' }}>Low contrast text</p>

// AFTER (Passes) - 7:1 contrast ratio
<p style={{ color: '#595959', background: '#fff' }}>High contrast text</p>

// Use tools like webaim.org/resources/contrastchecker/
```

**Failure 4: Buttons without accessible names**
```jsx
// BEFORE (Fails)
<button><TrashIcon /></button>

// AFTER (Passes)
<button aria-label="Delete item">
  <TrashIcon aria-hidden="true" />
</button>

// Or with visible text
<button>
  <TrashIcon aria-hidden="true" />
  <span>Delete</span>
</button>
```

**Failure 5: Heading levels skipped**
```jsx
// BEFORE (Fails)
<h1>Page Title</h1>
<h3>Section Title</h3>  {/* Skipped h2! */}

// AFTER (Passes)
<h1>Page Title</h1>
<h2>Section Title</h2>
<h3>Subsection Title</h3>
```

**Failure 6: Links with non-descriptive text**
```jsx
// BEFORE (Fails)
<a href="/products/123">Click here</a>
<a href="/download">Read more</a>

// AFTER (Passes)
<a href="/products/123">View Wireless Headphones details</a>
<a href="/download">Download product specification PDF</a>
```

**Comprehensive test file:**
```jsx
// AccessibilityTest.jsx
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('Accessibility', () => {
  it('should not have any automatically detectable accessibility violations', async () => {
    const { container } = render(<MyComponent />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});
```

**Key points:**
- Run axe audits regularly during development
- Fix violations as you code, not at the end
- Test with keyboard navigation and screen readers too
- Automated tools catch ~30-50% of accessibility issues
- Manual testing is still necessary

</details>

<details>
<summary>Exercise 2: Keyboard Navigation in Complex Form</summary>

**Testing checklist for keyboard navigation:**

```jsx
function ComplexRegistrationForm() {
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    interests: [],
    agreeToTerms: false,
  });

  return (
    <form>
      {step === 1 && (
        <fieldset>
          <legend>Account Information</legend>

          {/* Email input - Tab stop 1 */}
          <label htmlFor="email">Email Address</label>
          <input
            id="email"
            type="email"
            value={formData.email}
            onChange={(e) => setFormData({ ...formData, email: e.target.value })}
            required
          />

          {/* Password input - Tab stop 2 */}
          <label htmlFor="password">Password</label>
          <input
            id="password"
            type="password"
            value={formData.password}
            onChange={(e) => setFormData({ ...formData, password: e.target.value })}
            required
            aria-describedby="password-hint"
          />
          <p id="password-hint">Must be at least 8 characters</p>

          {/* Next button - Tab stop 3 */}
          <button type="button" onClick={() => setStep(2)}>
            Next
          </button>
        </fieldset>
      )}

      {step === 2 && (
        <fieldset>
          <legend>Interests (Select all that apply)</legend>

          {/* Checkbox group - Tab stops 4-7 */}
          <label>
            <input
              type="checkbox"
              value="tech"
              checked={formData.interests.includes('tech')}
              onChange={(e) => {
                const interests = e.target.checked
                  ? [...formData.interests, 'tech']
                  : formData.interests.filter(i => i !== 'tech');
                setFormData({ ...formData, interests });
              }}
            />
            Technology
          </label>

          <label>
            <input
              type="checkbox"
              value="sports"
              checked={formData.interests.includes('sports')}
              onChange={(e) => {
                const interests = e.target.checked
                  ? [...formData.interests, 'sports']
                  : formData.interests.filter(i => i !== 'sports');
                setFormData({ ...formData, interests });
              }}
            />
            Sports
          </label>

          {/* Navigation buttons - Tab stops 8-9 */}
          <button type="button" onClick={() => setStep(1)}>
            Back
          </button>
          <button type="button" onClick={() => setStep(3)}>
            Next
          </button>
        </fieldset>
      )}

      {step === 3 && (
        <fieldset>
          <legend>Review and Submit</legend>

          <div>
            <strong>Email:</strong> {formData.email}
          </div>
          <div>
            <strong>Interests:</strong> {formData.interests.join(', ')}
          </div>

          {/* Checkbox - Tab stop 10 */}
          <label>
            <input
              type="checkbox"
              checked={formData.agreeToTerms}
              onChange={(e) => setFormData({ ...formData, agreeToTerms: e.target.checked })}
              required
            />
            I agree to the terms and conditions
          </label>

          {/* Navigation buttons - Tab stops 11-12 */}
          <button type="button" onClick={() => setStep(2)}>
            Back
          </button>
          <button
            type="submit"
            disabled={!formData.agreeToTerms}
          >
            Submit
          </button>
        </fieldset>
      )}
    </form>
  );
}
```

**Keyboard navigation test procedure:**

1. **Tab key navigation:**
   - Press Tab to move forward through interactive elements
   - Press Shift+Tab to move backward
   - Verify logical tab order (top to bottom, left to right)
   - Ensure focus indicator is always visible

2. **Checkbox/Radio interaction:**
   - Use Space to toggle checkboxes
   - Use Arrow keys to select radio buttons in a group
   - Verify selection state is announced

3. **Button activation:**
   - Press Enter or Space to activate buttons
   - Verify disabled buttons can't be activated

4. **Form submission:**
   - Press Enter in text field to submit
   - Verify validation errors appear and are focusable

5. **Modal/Dialog interaction:**
   - Focus should trap inside modal
   - Escape key should close modal
   - Focus returns to trigger element on close

**Testing script:**
```javascript
// Manual testing checklist
const keyboardTestChecklist = [
  '✓ Can tab to all interactive elements',
  '✓ Tab order is logical (follows visual flow)',
  '✓ Focus indicator is visible on all elements',
  '✓ Can activate buttons with Enter and Space',
  '✓ Can toggle checkboxes with Space',
  '✓ Can navigate radio groups with arrows',
  '✓ Can submit form by pressing Enter',
  '✓ Skip links work (jump to main content)',
  '✓ Modal traps focus and closes on Escape',
  '✓ No keyboard traps (can always navigate away)',
];
```

**Common keyboard issues and fixes:**

```jsx
// Issue: Non-interactive element made clickable
// BEFORE (Fails keyboard nav)
<div onClick={handleClick}>Click me</div>

// AFTER (Works with keyboard)
<button onClick={handleClick}>Click me</button>

// Issue: Custom components not keyboard accessible
// BEFORE
<div className="dropdown" onClick={() => setOpen(!open)}>

// AFTER
<button
  aria-expanded={open}
  aria-controls="dropdown-menu"
  onClick={() => setOpen(!open)}
>
  Menu
</button>
<ul id="dropdown-menu" role="menu">
  <li role="menuitem">
    <button onClick={handleAction}>Action</button>
  </li>
</ul>
```

**Key points:**
- Never use positive tabIndex values
- Only interactive elements should be focusable
- Focus indicator must always be visible
- Logical tab order follows visual order
- Test with real keyboard, not just Tab key
- Document expected keyboard interactions

</details>

<details>
<summary>Exercise 3: Screen Reader Testing</summary>

**macOS VoiceOver Testing:**

```jsx
// Sample component to test
function ProductCard({ product }) {
  const { name, price, rating, image, inStock } = product;

  return (
    <article aria-label={`${name} product card`}>
      {/* Screen reader hears: "Product image, Wireless Headphones" */}
      <img src={image} alt={name} />

      <h3>{name}</h3>

      {/* Screen reader hears: "4.5 out of 5 stars" */}
      <div aria-label={`${rating} out of 5 stars`}>
        {'★'.repeat(Math.floor(rating))}{'☆'.repeat(5 - Math.floor(rating))}
      </div>

      <p>
        <span className="sr-only">Price:</span>
        ${price.toFixed(2)}
      </p>

      {/* Screen reader announces stock status */}
      <p aria-live="polite">
        {inStock ? 'In stock' : 'Out of stock'}
      </p>

      {/* Screen reader hears: "Add Wireless Headphones to cart, button" */}
      <button aria-label={`Add ${name} to cart`}>
        <CartIcon aria-hidden="true" />
        Add to Cart
      </button>
    </article>
  );
}
```

**VoiceOver testing procedure (macOS):**

1. **Enable VoiceOver:**
   - Press Cmd + F5
   - Or: System Preferences → Accessibility → VoiceOver

2. **Basic navigation:**
   - VO + Right Arrow: Next item
   - VO + Left Arrow: Previous item
   - VO + Space: Activate element
   - VO = Control + Option

3. **Rotor navigation:**
   - VO + U: Open rotor
   - Left/Right arrows: Switch categories (headings, links, form controls)
   - Up/Down arrows: Navigate within category

4. **Test checklist:**
```javascript
const screenReaderChecklist = [
  '✓ All images have appropriate alt text',
  '✓ Form inputs have labels announced',
  '✓ Headings create logical document outline',
  '✓ Buttons have clear, descriptive names',
  '✓ Links describe destination clearly',
  '✓ Error messages are announced',
  '✓ Dynamic content changes are announced',
  '✓ Tables have proper headers',
  '✓ No random text is skipped',
  '✓ Language is natural and makes sense',
];
```

**Windows NVDA Testing:**

1. **Install NVDA:**
   - Download from nvaccess.org (free)
   - Install and run

2. **Basic navigation:**
   - Down Arrow: Next line
   - Up Arrow: Previous line
   - Tab: Next focusable element
   - H: Next heading
   - B: Next button
   - F: Next form field

**Improved component with better screen reader support:**

```jsx
function AccessibleProductCard({ product }) {
  const { name, price, rating, ratingCount, image, inStock, description } = product;

  return (
    <article>
      {/* Provide context for the image */}
      <img
        src={image}
        alt={`${name} product image showing ${description}`}
      />

      <h3 id={`product-${product.id}-name`}>{name}</h3>

      {/* Announce rating clearly */}
      <div
        role="img"
        aria-label={`Rated ${rating} out of 5 stars based on ${ratingCount} reviews`}
      >
        <span aria-hidden="true">
          {'★'.repeat(Math.floor(rating))}{'☆'.repeat(5 - Math.floor(rating))}
        </span>
      </div>

      <p>
        {/* Price announced clearly */}
        <span className="sr-only">Price: </span>
        <span aria-label={`${price} dollars`}>
          ${price.toFixed(2)}
        </span>
      </p>

      {/* Stock status with appropriate live region */}
      <p aria-live="polite" aria-atomic="true">
        <span className={inStock ? 'in-stock' : 'out-of-stock'}>
          {inStock ? (
            <>
              <span className="sr-only">This item is </span>
              In stock
            </>
          ) : (
            <>
              <span className="sr-only">This item is </span>
              Out of stock
            </>
          )}
        </span>
      </p>

      {/* Clear button action with context */}
      <button
        aria-label={`Add ${name} to shopping cart`}
        aria-describedby={`product-${product.id}-name`}
        disabled={!inStock}
      >
        <CartIcon aria-hidden="true" />
        <span>Add to Cart</span>
      </button>

      {/* Additional info for screen readers */}
      <div className="sr-only">
        {description}. {inStock ? 'Available for immediate purchase.' : 'Currently unavailable.'}
      </div>
    </article>
  );
}
```

**Common screen reader issues and fixes:**

```jsx
// Issue: Icon-only buttons
// BEFORE (Screen reader hears: "button")
<button><TrashIcon /></button>

// AFTER (Screen reader hears: "Delete item, button")
<button aria-label="Delete item">
  <TrashIcon aria-hidden="true" />
</button>

// Issue: Loading states not announced
// BEFORE
{loading && <Spinner />}

// AFTER
{loading && (
  <div role="status" aria-live="polite">
    <Spinner aria-hidden="true" />
    <span className="sr-only">Loading content, please wait</span>
  </div>
)}

// Issue: Form errors not announced
// BEFORE
{errors.email && <span className="error">{errors.email}</span>}

// AFTER
{errors.email && (
  <span
    id="email-error"
    role="alert"
    className="error"
  >
    {errors.email}
  </span>
)}
<input
  type="email"
  aria-invalid={!!errors.email}
  aria-describedby={errors.email ? 'email-error' : undefined}
/>
```

**Key points:**
- Test with real screen readers, not just code review
- VoiceOver (macOS) and NVDA (Windows) are free
- Listen to entire flow, not just individual elements
- Ensure context is provided for all content
- Dynamic updates should use aria-live
- Hide decorative content with aria-hidden
- Test forms, modals, and complex interactions thoroughly

</details>

---

## What You Learned

This module covered:

- **Semantic HTML**: Use proper elements (button, nav, main, section) for structure screen readers understand
- **ARIA Attributes**: Add roles, labels, and states when HTML alone isn't descriptive enough
- **Keyboard Navigation**: Make all interactive elements reachable and operable with Tab, Enter, Space, Escape
- **Focus Management**: Trap focus in modals, restore focus on close, provide visible focus indicators
- **Accessible Forms**: Label all inputs, announce errors with aria-live, indicate required and invalid states
- **Testing**: Automate with jest-axe and Lighthouse, verify manually with screen readers (VoiceOver, NVDA)

**Key takeaway**: Accessibility isn't a checklist - it's designing for how all users experience your app, including those using assistive technology.

---

## Real-World Application

This week at work, you might use these concepts to:

- Audit your checkout flow with axe and fix keyboard navigation issues
- Ensure all form errors are announced to screen readers with aria-live
- Add skip links so keyboard users can bypass repetitive navigation
- Test your modal implementation with VoiceOver to verify focus trapping
- Achieve WCAG 2.1 AA compliance for a client contract or procurement requirement

---

## Further Reading

- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [MDN Accessibility Guide](https://developer.mozilla.org/en-US/docs/Web/Accessibility)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Deque University](https://dequeuniversity.com/)

---

**Navigation**: [← Previous Module](./09-git-workflows.md)
