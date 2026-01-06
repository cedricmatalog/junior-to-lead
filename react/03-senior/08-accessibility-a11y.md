# Accessibility (a11y)

> **Last reviewed**: 2026-01-06


## Week Eight: The Accessibility Review

An audit flagged multiple accessibility issues, and a key client requires WCAG compliance. Sarah asks you to lead the remediation plan. Marcus reminds you that accessibility is not a checklist -- it is a set of habits that shape every component. This week is about building inclusive UI: semantic HTML, ARIA usage, keyboard support, and realistic testing strategies.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **WCAG** | Building codes - minimum standards for safety |
| **Semantics** | Road signs - meaning that guides users |
| **ARIA** | Labels - fill the gaps when semantics fall short |
| **Keyboard support** | The remote control - full access without a mouse |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 06 (Security Practices) - Familiarity with safe UI defaults.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Apply WCAG principles to React components
- [ ] Use semantic HTML before relying on ARIA
- [ ] Implement keyboard navigation patterns
- [ ] Validate color contrast and focus visibility
- [ ] Build accessible forms and error messaging
- [ ] Test with screen readers and a11y tooling

---

## Time Estimate

- **Reading**: 70-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice accessibility over 6-8 weeks

---

## Chapter 1: WCAG Principles

You need a shared standard before you fix anything.

**POUR**: Perceivable, Operable, Understandable, Robust

### Level A (Minimum)
- Text alternatives for images
- Keyboard accessible
- No seizure-inducing content

### Level AA (Standard)
- Color contrast 4.5:1
- Resize text to 200%
- Focus visible
- Error identification

### Level AAA (Enhanced)
- Color contrast 7:1
- No timing
- No interruptions

## Chapter 2: Semantic HTML

Start with correct elements before adding any ARIA.

```tsx
// Bad: div soup
<div onClick={handleClick}>
  <div className="header">
    <div className="nav">
      <div onClick={navigate}>Home</div>
    </div>
  </div>
  <div className="content">
    <div className="title">Page Title</div>
  </div>
</div>

// Good: semantic elements
<main>
  <header>
    <nav>
      <a href="/">Home</a>
    </nav>
  </header>
  <article>
    <h1>Page Title</h1>
  </article>
</main>
```

## Chapter 3: ARIA Attributes

ARIA should enhance, not replace, semantics.

### When to Use ARIA

1. First: Use native HTML elements
2. Then: Add ARIA if native doesn't exist
3. Never: Override native semantics

```tsx
// Bad: Override native button semantics
<button role="link">Click me</button>

// Good: Use correct element
<a href="/page">Click me</a>

// Good: ARIA for custom widgets
<div
  role="slider"
  aria-valuemin={0}
  aria-valuemax={100}
  aria-valuenow={50}
  aria-label="Volume"
  tabIndex={0}
/>
```

### Common ARIA Patterns

```tsx
// Modal dialog
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="dialog-title"
  aria-describedby="dialog-description"
>
  <h2 id="dialog-title">Confirm Action</h2>
  <p id="dialog-description">Are you sure you want to proceed?</p>
</div>

// Tab panel
<div role="tablist" aria-label="Product tabs">
  <button role="tab" aria-selected="true" aria-controls="panel-1">
    Description
  </button>
  <button role="tab" aria-selected="false" aria-controls="panel-2">
    Reviews
  </button>
</div>
<div role="tabpanel" id="panel-1" aria-labelledby="tab-1">
  Content here
</div>

// Live regions for dynamic content
<div aria-live="polite" aria-atomic="true">
  {statusMessage}
</div>
```

## Chapter 4: Keyboard Navigation

Every action should be reachable without a mouse.

### Focus Management

```tsx
function Modal({ isOpen, onClose, children }) {
  const firstFocusRef = useRef<HTMLButtonElement>(null);
  const lastFocusRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) {
      firstFocusRef.current?.focus();
    }
  }, [isOpen]);

  const handleKeyDown = (e: KeyboardEvent) => {
    if (e.key === 'Escape') {
      onClose();
    }

    // Trap focus
    if (e.key === 'Tab') {
      if (e.shiftKey && document.activeElement === firstFocusRef.current) {
        e.preventDefault();
        lastFocusRef.current?.focus();
      } else if (!e.shiftKey && document.activeElement === lastFocusRef.current) {
        e.preventDefault();
        firstFocusRef.current?.focus();
      }
    }
  };

  return (
    <div role="dialog" onKeyDown={handleKeyDown}>
      <button ref={firstFocusRef}>First action</button>
      {children}
      <button ref={lastFocusRef} onClick={onClose}>Close</button>
    </div>
  );
}
```

### Skip Links

```tsx
function SkipLink() {
  return (
    <a
      href="#main-content"
      className="skip-link"
    >
      Skip to main content
    </a>
  );
}

// CSS
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  padding: 8px;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
```

## Chapter 5: Forms

Forms are where accessibility failures hurt the most.

```tsx
function AccessibleForm() {
  return (
    <form>
      {/* Associate label with input */}
      <label htmlFor="email">Email address</label>
      <input
        id="email"
        type="email"
        aria-describedby="email-hint email-error"
        aria-invalid={hasError}
      />
      <span id="email-hint">We'll never share your email.</span>
      {hasError && (
        <span id="email-error" role="alert">
          Please enter a valid email.
        </span>
      )}

      {/* Required fields */}
      <label htmlFor="name">
        Name <span aria-hidden="true">*</span>
      </label>
      <input id="name" required aria-required="true" />

      {/* Button with loading state */}
      <button type="submit" aria-busy={isLoading}>
        {isLoading ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

## Chapter 6: Color and Contrast

Contrast and focus are essential for readability and navigation.

```tsx
// Don't rely on color alone
// Bad
<span style={{ color: 'red' }}>Error</span>

// Good
<span style={{ color: 'red' }}>
  <ErrorIcon aria-hidden="true" /> Error
</span>

// Check contrast ratios
// Normal text: 4.5:1 minimum
// Large text (18pt+): 3:1 minimum
```

## Chapter 7: Images

Alt text is the difference between silence and meaning.

```tsx
// Informative image
<img src="chart.png" alt="Sales increased 25% in Q4" />

// Decorative image
<img src="decoration.png" alt="" role="presentation" />

// Complex image
<figure>
  <img src="complex-chart.png" alt="Quarterly sales comparison" />
  <figcaption>
    Q4 2024 showed 25% growth compared to Q3, driven by holiday sales.
  </figcaption>
</figure>
```

## Chapter 8: Testing

Automated tools are a start, but real checks require real tools.

### Automated Testing

```tsx
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

it('has no accessibility violations', async () => {
  const { container } = render(<MyComponent />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

### Manual Testing Checklist

- [ ] Navigate with keyboard only
- [ ] Test with screen reader (VoiceOver, NVDA)
- [ ] Zoom to 200%
- [ ] Test with high contrast mode
- [ ] Check focus indicators
- [ ] Verify form error announcements

---

## Common Mistakes

1. **Using divs for buttons** - Buttons and links have built-in semantics.
2. **Relying on color alone** - Provide text or icon cues too.
3. **Missing focus styles** - Keyboard users lose track without visible focus.
4. **Adding ARIA everywhere** - Use native semantics first.


Example:
```jsx
// ❌ Non-semantic click target
<div onClick={openMenu}>Menu</div>

// ✅ Semantic control
<button onClick={openMenu}>Menu</button>
```

## Practice Exercises

1. Add keyboard navigation to a custom dropdown
2. Make a data table accessible
3. Implement a focus trap for a modal

### Solutions

<details>
<summary>Exercise 1: Keyboard Dropdown</summary>

```tsx
function Dropdown({ options, onSelect }) {
  const [open, setOpen] = useState(false);
  const [index, setIndex] = useState(0);
  const listId = 'dropdown-listbox';

  const handleKeyDown = (event) => {
    if (!open && event.key === 'Enter') setOpen(true);
    if (!open) return;

    if (event.key === 'ArrowDown') setIndex((i) => Math.min(i + 1, options.length - 1));
    if (event.key === 'ArrowUp') setIndex((i) => Math.max(i - 1, 0));
    if (event.key === 'Enter') onSelect(options[index]);
    if (event.key === 'Escape') setOpen(false);
  };

  return (
    <div onKeyDown={handleKeyDown}>
      <button
        type="button"
        aria-haspopup="listbox"
        aria-expanded={open}
        aria-controls={listId}
        onClick={() => setOpen((value) => !value)}
      >
        {options[index]}
      </button>
      {open && (
        <ul id={listId} role="listbox" aria-activedescendant={`option-${index}`}>
          {options.map((option, i) => (
            <li
              id={`option-${i}`}
              key={option}
              role="option"
              aria-selected={i === index}
              onMouseDown={() => onSelect(options[i])}
            >
              {option}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

**Key points:**
- Arrow keys move focus.
- Escape closes the menu.
- ARIA roles describe the widget.

</details>

<details>
<summary>Exercise 2: Accessible Table</summary>

```tsx
<table>
  <caption>Quarterly revenue by region</caption>
  <thead>
    <tr>
      <th scope="col">Region</th>
      <th scope="col">Q1</th>
      <th scope="col">Q2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">North America</th>
      <td>$2.1M</td>
      <td>$2.4M</td>
    </tr>
  </tbody>
</table>
```

**Key points:**
- Use `caption` for context.
- `scope` associates headers correctly.
- Keep tables for tabular data only.

</details>

<details>
<summary>Exercise 3: Modal Focus Trap</summary>

```tsx
function Modal({ onClose, children }) {
  const ref = useRef(null);
  const triggerRef = useRef(document.activeElement);

  useEffect(() => {
    const focusable = ref.current.querySelectorAll('button, a, input, [tabindex]');
    const first = focusable[0];
    const last = focusable[focusable.length - 1];

    function handleKeyDown(event) {
      if (event.key === 'Escape') onClose();
      if (event.key !== 'Tab') return;
      if (event.shiftKey && document.activeElement === first) {
        event.preventDefault();
        last.focus();
      } else if (!event.shiftKey && document.activeElement === last) {
        event.preventDefault();
        first.focus();
      }
    }

    document.addEventListener('keydown', handleKeyDown);
    first?.focus();
    return () => {
      document.removeEventListener('keydown', handleKeyDown);
      triggerRef.current?.focus?.();
    };
  }, []);

  return (
    <div role="dialog" aria-modal="true">
      <div ref={ref}>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
}
```

**Key points:**
- Focus stays inside the modal.
- First focusable element gets focus on open.
- Escape closes the modal and restores focus.

</details>

---

## What You Learned

This module covered:

- **WCAG principles**: The foundation for inclusive UI
- **Semantics**: Native elements that carry meaning
- **ARIA**: Enhancements for custom components
- **Keyboard support**: Full navigation without a mouse
- **Testing**: Automated checks plus manual verification

**Key takeaway**: Accessibility is part of quality, not an afterthought.

---

## Real-World Application

This week at work, you might use these concepts to:

- Fix focus traps in dialogs and menus
- Add aria labels to custom widgets
- Improve color contrast in design tokens
- Add automated accessibility checks in CI
- Run a manual screen reader review of a critical flow

---

## Further Reading

- [WCAG 2.1 Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/)
- [A11y Project](https://www.a11yproject.com/)

---

**Navigation**: [← Previous Module](./07-supply-chain-security.md) | [Next Module →](./09-design-systems.md)
