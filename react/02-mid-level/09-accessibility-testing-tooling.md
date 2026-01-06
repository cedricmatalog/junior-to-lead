# Accessibility Testing Tooling

> **Last reviewed**: 2026-01-06


## Week Nine: The Audit That Never Ends

The app passes manual keyboard checks, but a customer report shows missing labels in a new flow.

"Manual checks are necessary, not sufficient," Sarah says. "Automated tooling keeps you honest."

Marcus adds, "Think of a11y tests as smoke detectors. Quiet most days, critical when they beep."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Automated audit** | A spellcheck - fast and consistent, but not perfect |
| **Ruleset** | Building codes - pass/fail checks with reasons |
| **CI gate** | A turnstile - no pass without standards |
| **Manual review** | A human inspection - judgement beyond rules |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Mid-Level Module 08 (Testing Strategies) - Familiarity with test setup and CI workflows.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Run axe audits locally and interpret results
- [ ] Add jest-axe for unit and integration tests
- [ ] Use Playwright + axe for E2E accessibility checks
- [ ] Set CI gates for critical accessibility violations
- [ ] Combine automated and manual a11y testing
- [ ] Report and track accessibility regressions

---

## Time Estimate

- **Reading**: 60-75 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Build a11y test habits over 3-4 weeks

---

## Chapter 1: Local Audits with axe

Start with fast feedback.

Sarah wants a quick pass before every PR.

```text
1) Open DevTools
2) Run axe DevTools scan
3) Fix critical and serious issues
```

"This catches missing labels and ARIA issues before code review," Sarah says.

---

## Chapter 2: jest-axe for Components

Automate a11y checks in component tests.

Marcus wants it running with the unit suite.

```ts
import { render } from '@testing-library/react';
import { axe } from 'jest-axe';

it('has no a11y violations', async () => {
  const { container } = render(<LoginForm />);
  expect(await axe(container)).toHaveNoViolations();
});
```

"Treat a11y like unit tests: fast and frequent," Marcus says.

---

## Chapter 3: Playwright + axe for E2E

Run audits on real flows.

Sarah wants proof on the user journeys that matter.

```ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('checkout is accessible', async ({ page }) => {
  await page.goto('/checkout');
  const results = await new AxeBuilder({ page }).analyze();
  expect(results.violations).toEqual([]);
});
```

"This catches issues that only show up in integrated UI," Sarah says.

---

## Chapter 4: CI Gates

Fail the build on critical violations.

Marcus insists the gate is real, not a warning.

```yaml
- name: A11y tests
  run: npm test -- --runInBand a11y
```

"Gate on severity. Let warnings through while you fix the backlog," Marcus says.

---

## Chapter 5: Manual Checks Still Matter

Automated tools miss real usability problems.

Sarah still does a keyboard walk-through before launch.

"Keyboard flows, focus order, and screen-reader UX still need humans."

---

## Chapter 6: Reporting and Tracking

Add a ticket template for regressions.

Marcus wants regressions tracked like any other bug.

```text
A11y Regression:
- URL:
- Steps:
- Expected:
- Actual:
- Severity:
```

"A clear report keeps fixes fast and scoped."

---

## Common Mistakes

1. **Treating automated checks as complete** - Manual testing is still required.
2. **Ignoring severity levels** - Start with critical and serious issues.
3. **Running audits only once** - Accessibility is a continuous practice.
4. **Testing without real data** - Many issues appear only in real UI states.


Example:
```ts
// ❌ Only run audits manually
// (no automated coverage)

// ✅ Add jest-axe to tests
expect(await axe(container)).toHaveNoViolations();
```

---

## Practice Exercises

1. Add jest-axe to one component test
2. Run a Playwright + axe audit on a key flow
3. Add an a11y CI gate for critical violations

### Solutions

<details>
<summary>Exercise 1: jest-axe Setup</summary>

```ts
import { render } from '@testing-library/react';
import { axe } from 'jest-axe';

it('card is accessible', async () => {
  const { container } = render(<ProductCard />);
  expect(await axe(container)).toHaveNoViolations();
});
```

**Key points:**
- Use in unit/integration tests.
- Keep checks fast and isolated.
- Fix violations immediately.

</details>

<details>
<summary>Exercise 2: Playwright + axe</summary>

```ts
const results = await new AxeBuilder({ page }).analyze();
expect(results.violations).toEqual([]);
```

**Key points:**
- Run on real flows.
- Fail fast on critical issues.
- Use it for smoke tests.

</details>

<details>
<summary>Exercise 3: CI Gate</summary>

```yaml
- name: A11y tests
  run: npm test -- --runInBand a11y
```

**Key points:**
- Add to the main CI pipeline.
- Gate on severity.
- Keep it consistent across teams.

</details>

---

## What You Learned

This module covered:

- **Local audits**: axe DevTools for quick checks
- **Unit a11y tests**: jest-axe in component tests
- **E2E audits**: Playwright + axe
- **CI gates**: Block critical regressions
- **Manual testing**: Humans still validate UX

**Key takeaway**: Automated a11y tooling keeps standards consistent and regressions rare.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add an a11y gate to your CI pipeline
- Catch missing labels before release
- Audit a high-traffic checkout flow
- Create a regression template for violations
- Combine manual audits with automated checks

---

## Further Reading

- [axe-core](https://github.com/dequelabs/axe-core)
- [jest-axe](https://github.com/nickcolley/jest-axe)
- [Playwright + axe](https://playwright.dev/docs/accessibility-testing)

---

**Navigation**: [← Previous Module](./08-testing-strategies.md) | [Next Module →](./10-typescript-mastery.md)
