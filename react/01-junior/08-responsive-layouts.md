# Responsive Layouts

> **Last reviewed**: 2026-01-06


## Week Eight: The Dashboard on Every Screen

The analytics dashboard looks perfect on your laptop. Then Sarah opens it on her phone.

The sidebar crushes the chart. Buttons overlap. Text wraps in awkward places.

"It has to work everywhere," she says. "Phones, tablets, kiosks. Responsive is not optional."

Marcus smiles. "Good responsive design is invisible. Bad responsive design is all anyone sees."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Breakpoints** | Traffic lanes - change layout when the road narrows |
| **Flexbox** | Elastic bands - items stretch and wrap |
| **Grid** | Graph paper - precise rows and columns |
| **Fluid sizing** | Water level - content rises and falls with the container |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 07 (Styling Approaches) - Comfort with CSS modules or utility classes.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Build layouts that adapt across mobile, tablet, and desktop
- [ ] Use Flexbox for flexible rows, columns, and alignment
- [ ] Use CSS Grid for page-level layout and cards
- [ ] Apply fluid sizing with `clamp()` and relative units
- [ ] Make media and images responsive without distortion
- [ ] Test responsive behavior with browser tools

---

## Time Estimate

- **Reading**: 55-75 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice responsive patterns over 2-3 weeks

---

## Chapter 1: Mobile-First Thinking

"Start small, then expand," Sarah says. "If it works on a phone, it scales up."

```css
.card {
  padding: 16px;
  font-size: 1rem;
}

@media (min-width: 768px) {
  .card {
    padding: 24px;
    font-size: 1.125rem;
  }
}
```

"Mobile-first keeps the base simple and avoids overrides."

---

## Chapter 2: Flexbox for Components

You need a header that stacks on mobile and aligns on desktop.

```css
.header {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

@media (min-width: 768px) {
  .header {
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
  }
}
```

"Flexbox is perfect for one-dimensional layouts," Marcus says.

---

## Chapter 3: Grid for Page Layouts

A dashboard needs a grid.

```css
.dashboard {
  display: grid;
  gap: 16px;
  grid-template-columns: 1fr;
}

@media (min-width: 1024px) {
  .dashboard {
    grid-template-columns: 280px 1fr;
  }
}
```

"Grid is the blueprint. Flexbox is the furniture."

---

## Chapter 4: Fluid Sizing

Hardcoded pixels break on unusual screens.

```css
h1 {
  font-size: clamp(1.5rem, 2vw + 1rem, 2.5rem);
}

.container {
  width: min(90vw, 1200px);
  margin: 0 auto;
}
```

"Let the layout breathe."

---

## Chapter 5: Responsive Media

Images should fit without stretching or overflowing.

```css
.product-image {
  width: 100%;
  height: auto;
  aspect-ratio: 4 / 3;
  object-fit: cover;
}
```

"`object-fit` keeps images clean, even when containers change size."

---

## Chapter 6: The Dashboard Fix

You refactor the dashboard layout:

- Sidebar collapses into a top bar on mobile
- Cards stack in one column, then two
- Charts expand on desktop

Sarah checks it on her phone. It finally works.

---

## Common Mistakes

1. **Only testing on desktop** - Use mobile emulation early.
2. **Fixed widths everywhere** - Use `%`, `vw`, and `min/max`.
3. **Ignoring touch targets** - Buttons need space and padding.
4. **Forgetting images** - Media must scale with the layout.


Example:
```css
/* ❌ Fixed widths break on small screens */
.card { width: 600px; }

/* ✅ Fluid widths adapt */
.card { width: min(100%, 600px); }
```

---

## Debug This Code

Before moving to the exercises, test your debugging skills. Each snippet has bugs - can you spot and fix them?

<details>
<summary>Challenge 1: Flex Row That Never Stacks</summary>

```css
.header {
  display: flex;
  flex-direction: row;
}

@media (min-width: 768px) {
  .header {
    flex-direction: row;
  }
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

Where is the mobile-first change?

</details>

<details>
<summary>Solution</summary>

**Bug**: The base layout should be `column`, not `row`.

**Fixed code:**

```css
.header {
  display: flex;
  flex-direction: column;
}

@media (min-width: 768px) {
  .header {
    flex-direction: row;
  }
}
```

**Why it matters**: Mobile-first layouts should start stacked.

</details>
</details>

<details>
<summary>Challenge 2: Grid That Overflows</summary>

```css
.grid {
  display: grid;
  grid-template-columns: 300px 300px 300px;
  gap: 24px;
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

What happens on a 375px-wide screen?

</details>

<details>
<summary>Solution</summary>

**Bug**: Fixed columns overflow small screens.

**Fixed code:**

```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 24px;
}
```

**Why it matters**: `auto-fit` adapts the number of columns to the space.

</details>
</details>

<details>
<summary>Challenge 3: Image That Stretches</summary>

```css
.hero img {
  width: 100%;
  height: 300px;
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

How can you preserve aspect ratio?

</details>

<details>
<summary>Solution</summary>

**Bug**: Fixed height distorts images.

**Fixed code:**

```css
.hero img {
  width: 100%;
  height: auto;
}
```

**Why it matters**: Flexible heights keep images from squashing.

</details>
</details>

---

## Practice Exercises

1. Build a responsive card grid that goes 1 → 2 → 3 columns
2. Create a header that collapses into a stacked layout on mobile
3. Make a product image responsive with `object-fit`

### Solutions

<details>
<summary>Exercise 1: Responsive Card Grid</summary>

```css
.grid {
  display: grid;
  gap: 16px;
  grid-template-columns: 1fr;
}

@media (min-width: 640px) {
  .grid { grid-template-columns: repeat(2, 1fr); }
}

@media (min-width: 1024px) {
  .grid { grid-template-columns: repeat(3, 1fr); }
}
```

**Key points:**
- Start with one column.
- Increase columns at sensible breakpoints.
- Keep gaps consistent.

</details>

<details>
<summary>Exercise 2: Responsive Header</summary>

```css
.header {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

@media (min-width: 768px) {
  .header {
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
  }
}
```

**Key points:**
- Stack on mobile.
- Align and space on desktop.
- Use `gap` for spacing.

</details>

<details>
<summary>Exercise 3: Responsive Image</summary>

```css
.product-image {
  width: 100%;
  height: auto;
  aspect-ratio: 4 / 3;
  object-fit: cover;
}
```

**Key points:**
- Use `width: 100%` for scaling.
- `aspect-ratio` keeps proportions.
- `object-fit` avoids distortion.

</details>

---

## What You Learned

This module covered:

- **Breakpoints**: Scale layouts for different screens
- **Flexbox**: Build flexible component layouts
- **Grid**: Structure page-level layouts
- **Fluid sizing**: Use `clamp` and relative units
- **Media**: Keep images responsive and clean
- **Testing**: Validate layouts with device emulation

**Key takeaway**: Responsive design is the difference between a demo and a product.

---

## Real-World Application

This week at work, you might use these concepts to:

- Make a dashboard usable on tablets and phones
- Convert a fixed layout into a flexible grid
- Add responsive spacing and typography to a marketing page
- Fix image distortion in product cards
- Audit touch target sizes across the UI

---

## Further Reading

- [MDN: Responsive Design](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Responsive_Design)
- [MDN: CSS Grid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout)
- [MDN: Flexbox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox)

---

**Navigation**: [← Previous Module](./07-styling-approaches.md) | [Next Module →](./09-tooling-fundamentals.md)
