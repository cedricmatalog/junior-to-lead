# Design Handoff and Collaboration

> **Last reviewed**: 2026-01-06


## Week Ten: The Spec That Didn't Match Reality

Design ships a new flow. The build looks close but feels off: spacing, colors, and behavior don't match.

"The gap isn't code. It's the handoff," Sarah says.

Marcus adds, "Strong collaboration makes design and engineering speak the same language."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Design tokens** | A shared dictionary - names for visual decisions |
| **Spec** | A blueprint - exact measurements and behaviors |
| **Component contract** | A handshake - what a component promises |
| **QA pass** | A final walkthrough - compare to the blueprint |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 09 (Design Systems) - Familiarity with component libraries and tokens.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Translate design specs into component contracts
- [ ] Use tokens to align design and code
- [ ] Establish a design QA checklist
- [ ] Handle design changes without rework
- [ ] Communicate tradeoffs with designers
- [ ] Keep handoffs consistent across teams

---

## Time Estimate

- **Reading**: 55-75 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Apply handoff practices over 4-6 weeks

---

## Chapter 1: Tokens as Shared Language

"If you name it once, you stop arguing about it," Sarah says.

```text
color.primary.600
spacing.4
radius.md
```

"Tokens let design and code agree on the same values."

---

## Chapter 2: Component Contracts

Define the API clearly.

```text
Button variants: primary | secondary | ghost
Sizes: sm | md | lg
States: default | loading | disabled
```

"If the contract is clear, the implementation stays stable."

---

## Chapter 3: Spec to Code Checklist

Marcus hands you a checklist:

- Spacing matches spec
- Typography uses tokens
- States are covered (hover, focus, disabled)
- Accessibility verified

"This turns subjective feedback into objective checks."

---

## Chapter 4: Handling Changes

Design changes happen. You need a process.

"Track changes in a single thread and update tokens first," Sarah says.

---

## Chapter 5: Collaboration Rituals

You add a weekly design-engineering review.

"Regular alignment prevents last-minute surprises."

---

## Chapter 6: QA and Sign-Off

Ship only after a joint review.

"A 15-minute QA session saves days of rework."

---

## Common Mistakes

1. **Hardcoding values** - Use tokens so design stays consistent.
2. **Ignoring states** - Hover and focus are part of the spec.
3. **Skipping QA** - Visual drift accumulates fast.
4. **No change process** - Rework becomes constant.


Example:
```css
/* ❌ One-off color */
button { color: #2563eb; }

/* ✅ Tokenized color */
button { color: var(--color-primary-600); }
```

---

## Practice Exercises

1. Create a component contract for a modal
2. Map a Figma spec to token names
3. Draft a design QA checklist for a new feature

### Solutions

<details>
<summary>Exercise 1: Modal Contract</summary>

```text
Modal sizes: sm | md | lg
Props: title, description, actions
States: open | closed | loading
```

**Key points:**
- Keep API explicit.
- Cover states.
- Align with design tokens.

</details>

<details>
<summary>Exercise 2: Token Mapping</summary>

```text
Figma color: Primary/600 -> color.primary.600
Figma spacing: 16 -> spacing.4
Figma radius: 8 -> radius.md
```

**Key points:**
- Use a consistent naming scheme.
- Map once, reuse everywhere.
- Keep tokens in version control.

</details>

<details>
<summary>Exercise 3: QA Checklist</summary>

```text
- Typography matches spec
- Spacing uses tokens
- Hover/focus states present
- A11y checks pass
```

**Key points:**
- Make QA objective.
- Include accessibility.
- Review with design.

</details>

---

## What You Learned

This module covered:

- **Tokens**: Shared language for visual decisions
- **Contracts**: Clear component APIs
- **QA**: Repeatable spec checks
- **Collaboration**: Rituals that prevent drift

**Key takeaway**: Good handoffs reduce rework and preserve design intent.

---

## Real-World Application

This week at work, you might use these concepts to:

- Map a new design system token set
- Define component contracts for a shared library
- Run a joint design QA pass
- Track design changes in a single source
- Reduce visual inconsistencies across screens

---

## Further Reading

- [Design Tokens W3C](https://design-tokens.github.io/community-group/format/)
- [Figma Dev Mode](https://help.figma.com/hc/en-us/articles/207238219-Figma-for-developers)
- [Storybook for UI Review](https://storybook.js.org/)

---

**Navigation**: [← Previous Module](./09-design-systems.md) | [Next Module →](./11-observability-monitoring.md)
