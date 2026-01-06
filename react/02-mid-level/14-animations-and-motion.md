# Animations and Motion

> **Last reviewed**: 2026-01-06


## Week Fourteen: The UI That Feels Alive

The design team wants the dashboard to feel more "alive." Buttons should respond, cards should glide into place, and loading states should feel intentional.

"Motion is communication," Sarah says. "It tells users what changed and what to focus on."

Marcus adds, "But motion done wrong is noise. The goal is clarity, not fireworks."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Transition** | A door swing - smooth movement between states |
| **Animation** | A pulse - continuous motion with purpose |
| **Easing** | Gravity - motion feels natural when it slows down |
| **Reduced motion** | A volume knob - turn motion down when needed |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 07 (Styling Approaches) - Comfort with component styling and UI structure.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Use CSS transitions for common UI state changes
- [ ] Create keyframe animations for attention cues
- [ ] Apply easing curves to make motion feel natural
- [ ] Use Framer Motion for complex interactions
- [ ] Respect `prefers-reduced-motion` for accessibility
- [ ] Avoid performance pitfalls with motion

---

## Time Estimate

- **Reading**: 55-75 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice motion patterns over 3-4 weeks

---

## Chapter 1: Transitions for State Changes

You start with hover and focus states.

```css
.button {
  transition: transform 120ms ease, box-shadow 120ms ease;
}

.button:hover {
  transform: translateY(-1px);
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.12);
}
```

"Small transitions make a UI feel responsive without distracting the user," Sarah says.

---

## Chapter 2: Keyframes for Attention

Loading states need gentle motion.

```css
@keyframes pulse {
  0% { opacity: 0.6; }
  50% { opacity: 1; }
  100% { opacity: 0.6; }
}

.skeleton {
  animation: pulse 1.2s ease-in-out infinite;
}
```

"A subtle pulse says 'working' without yelling," Marcus says.

---

## Chapter 3: Easing Matters

A linear animation feels robotic.

```css
.card {
  transition: transform 180ms cubic-bezier(0.22, 1, 0.36, 1);
}
```

"Easing makes motion feel real. Use it intentionally."

---

## Chapter 4: Framer Motion for Complex UI

A sidebar should slide in and out smoothly.

```jsx
import { motion, AnimatePresence } from 'framer-motion';

function Sidebar({ open }) {
  return (
    <AnimatePresence>
      {open && (
        <motion.aside
          initial={{ x: -280, opacity: 0 }}
          animate={{ x: 0, opacity: 1 }}
          exit={{ x: -280, opacity: 0 }}
          transition={{ duration: 0.2 }}
        >
          <Nav />
        </motion.aside>
      )}
    </AnimatePresence>
  );
}
```

"Use a library when motion affects layout or multiple states," Sarah says.

---

## Chapter 5: Reduced Motion

Some users prefer less motion. Respect that.

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation: none !important;
    transition: none !important;
  }
}
```

"Accessibility includes motion. Give users control."

---

## Chapter 6: Performance Pitfalls

"Animate transforms and opacity. Avoid layout thrash," Marcus says.

```css
/* ✅ GPU-friendly */
.card { transform: translateY(0); }

/* ❌ Triggers layout */
.card { top: 0; }
```

---

## Common Mistakes

1. **Animating layout properties** - Use `transform` and `opacity` instead.
2. **Overusing motion** - Too many animations feel chaotic.
3. **Ignoring reduced motion** - Always honor user preferences.
4. **Long durations** - Keep UI motion quick and subtle.


Example:
```css
/* ❌ Slow, distracting animation */
.modal { transition: transform 800ms ease; }

/* ✅ Short and snappy */
.modal { transition: transform 200ms ease; }
```

---

## Practice Exercises

1. Add hover and focus transitions to a card component
2. Build a loading skeleton with keyframe animation
3. Create a sliding drawer using Framer Motion

### Solutions

<details>
<summary>Exercise 1: Card Transitions</summary>

```css
.card {
  transition: transform 160ms ease, box-shadow 160ms ease;
}

.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
}
```

**Key points:**
- Short durations feel responsive.
- Transform keeps animations smooth.
- Use subtle elevation for focus.

</details>

<details>
<summary>Exercise 2: Skeleton Loader</summary>

```css
@keyframes shimmer {
  0% { background-position: -200px 0; }
  100% { background-position: 200px 0; }
}

.skeleton {
  background: linear-gradient(90deg, #eee, #f5f5f5, #eee);
  background-size: 400px 100%;
  animation: shimmer 1.2s infinite;
}
```

**Key points:**
- Keep animation subtle.
- Use gradients for shimmer.
- Prefer CSS animations for simple effects.

</details>

<details>
<summary>Exercise 3: Sliding Drawer</summary>

```jsx
function Drawer({ open }) {
  return (
    <AnimatePresence>
      {open && (
        <motion.div
          initial={{ x: '100%' }}
          animate={{ x: 0 }}
          exit={{ x: '100%' }}
          transition={{ duration: 0.2 }}
        >
          <DrawerContent />
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

**Key points:**
- Use `AnimatePresence` for exit animations.
- Keep durations short.
- Animate transforms for smoothness.

</details>

---

## What You Learned

This module covered:

- **Transitions**: Smooth state changes
- **Keyframes**: Light attention cues
- **Easing**: Natural motion feel
- **Framer Motion**: Complex, multi-state animations
- **Reduced motion**: Accessibility for motion sensitivity
- **Performance**: Avoid layout-heavy animations

**Key takeaway**: Motion is effective when it clarifies, not when it decorates.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add polished hover states to interactive cards
- Use skeleton loaders for async data
- Animate a drawer or modal without layout jank
- Respect reduced-motion preferences
- Audit motion to remove distracting animations

---

## Further Reading

- [MDN: CSS Transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/transition)
- [MDN: CSS Animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations)
- [Framer Motion Docs](https://www.framer.com/motion/)

---

**Navigation**: [← Previous Module](./13-e2e-testing-with-playwright.md) | [Next Module →](./15-deployment-and-ci.md)
