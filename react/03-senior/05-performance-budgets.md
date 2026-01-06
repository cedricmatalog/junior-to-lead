# Performance Budgets and Governance

> **Last reviewed**: 2026-01-06


## Week Five: The Slow Creep

The app shipped fast. Then week by week, pages got heavier. LCP slipped from 2.2s to 3.1s.

"Performance doesn't fail all at once," Sarah says. "It erodes."\

Marcus adds, "Budgets make performance visible and enforceable."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Budget** | A speed limit - acceptable bounds for performance |
| **Guardrail** | A highway barrier - prevents silent regressions |
| **Regression** | A leak - small losses over time |
| **Ownership** | A scoreboard - someone must watch the numbers |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 04 (Advanced Performance) - Familiarity with Web Vitals and profiling.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Define performance budgets for metrics and bundle size
- [ ] Choose tools to measure budgets consistently
- [ ] Add CI checks for performance regressions
- [ ] Create a process for performance exceptions
- [ ] Assign ownership for ongoing performance health
- [ ] Report performance trends to stakeholders

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Use budgets over 4-6 weeks

---

## Chapter 1: Choose the Metrics

Sarah lists the core metrics:

- LCP
- INP
- CLS
- Bundle size

"Budgets should cover user experience and payload size."

---

## Chapter 2: Set Realistic Targets

Start with baselines, then improve.

```text
LCP: <= 2.5s
INP: <= 200ms
CLS: <= 0.1
JS bundle: <= 200KB
```

"Budgets should be strict enough to matter, loose enough to be achievable."

---

## Chapter 3: Tooling for Budgets

You add Lighthouse CI and bundlesize checks.

```yaml
- name: Lighthouse CI
  run: npx lhci autorun
- name: Bundle size
  run: npx bundlesize
```

"Consistency matters more than perfection."

---

## Chapter 4: CI Gates and Exceptions

Not every release can pass budgets perfectly.

"Use exception paths with time limits," Marcus says.

```text
Exception: +15KB for analytics
Expiry: 30 days
Owner: Frontend Team
```

---

## Chapter 5: Ownership and Reporting

You create a weekly performance report.

"If nobody owns it, nobody fixes it," Sarah says.

---

## Chapter 6: Keeping It Sustainable

Budgets become part of the release checklist.

"Make it routine. Then it stays healthy."

---

## Common Mistakes

1. **Using no budgets** - Regressions go unnoticed.
2. **Setting unrealistic targets** - Teams ignore impossible rules.
3. **No exception path** - Teams bypass tooling instead.
4. **No owner** - Performance becomes nobody's job.


Example:
```text
# ❌ No tracking
"Perf feels fine."

# ✅ Budgeted tracking
LCP <= 2.5s, JS <= 200KB
```

---

## Practice Exercises

1. Define a budget for a product page
2. Add a bundlesize check to CI
3. Write a performance exception template

### Solutions

<details>
<summary>Exercise 1: Budget Definition</summary>

```text
LCP <= 2.5s
INP <= 200ms
CLS <= 0.1
JS bundle <= 200KB
```

**Key points:**
- Use real metrics.
- Keep budgets measurable.
- Align with business goals.

</details>

<details>
<summary>Exercise 2: Bundlesize CI</summary>

```yaml
- name: Bundle size
  run: npx bundlesize
```

**Key points:**
- Keep thresholds in config.
- Fail on regressions.
- Track trends over time.

</details>

<details>
<summary>Exercise 3: Exception Template</summary>

```text
Exception: +20KB for feature X
Expiry: 2 weeks
Owner: Team Y
Reason: Critical launch
```

**Key points:**
- Exceptions should expire.
- Assign ownership.
- Keep it visible.

</details>

---

## What You Learned

This module covered:

- **Budgets**: Define measurable limits
- **Tooling**: Automate checks in CI
- **Exceptions**: Controlled deviations
- **Ownership**: Assign accountability

**Key takeaway**: Performance budgets turn speed into a managed product requirement.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add budgets to a critical route
- Catch a bundle regression before release
- Report performance trends to stakeholders
- Create an exception policy for launches
- Assign ownership for performance metrics

---

## Further Reading

- [Lighthouse CI](https://github.com/GoogleChrome/lighthouse-ci)
- [Web Vitals](https://web.dev/vitals/)
- [Performance Budgets](https://web.dev/performance-budgets-101/)

---

**Navigation**: [← Previous Module](./04-advanced-performance.md) | [Next Module →](./06-security-practices.md)
