# Accessibility (a11y)

Build applications that everyone can use.

## Learning Objectives

By the end of this module, you will:
- Apply WCAG guidelines to React applications
- Use ARIA attributes correctly
- Implement keyboard navigation
- Test with screen readers

## WCAG Principles

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

## Semantic HTML

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

## ARIA Attributes

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

## Keyboard Navigation

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

## Forms

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

## Color and Contrast

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

## Images

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

## Testing

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

## Practice Exercises

1. Add keyboard navigation to a custom dropdown
2. Make a data table accessible
3. Implement a focus trap for a modal

## Further Reading

- [WCAG 2.1 Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/)
- [A11y Project](https://www.a11yproject.com/)
