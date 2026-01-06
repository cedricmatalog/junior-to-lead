# Production Debugging and Triage

> **Last reviewed**: 2026-01-06


## Week Twelve: The Bug Nobody Can Reproduce

A critical user journey breaks in production, but every developer says, "It works on my machine."

"Production debugging is a different skill," Sarah says. "You need a system."

Marcus adds, "Triage is about narrowing the blast radius, not solving everything at once."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Triage** | An emergency room - prioritize before you treat |
| **Signal** | A flashlight - focus on the right evidence |
| **Mitigation** | A tourniquet - stop the bleeding first |
| **Root cause** | The map - how you prevent repeats |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Senior Module 11 (Observability and Monitoring) - Familiarity with logs, errors, and metrics.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Triage incidents by severity and scope
- [ ] Use logs and traces to reproduce issues
- [ ] Mitigate user impact quickly
- [ ] Isolate regressions to deployments or features
- [ ] Communicate updates during incidents
- [ ] Document follow-ups for prevention

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice over 4-6 weeks

---

## Chapter 1: The First Five Minutes

"Start with impact," Sarah says.

You write the impact questions before you chase the root cause.

- Who is affected?
- What is broken?
- Is there a workaround?

"This tells you whether to mitigate or investigate first."

---

## Chapter 2: Gather Signals

Pull error logs, traces, and recent deploys.

"Signal beats guesswork," Marcus says.

---

## Chapter 3: Mitigate Fast

If users are blocked, reduce impact first.

"Mitigate first, analyze second," Sarah says.

- Disable a feature flag
- Roll back a release
- Hide a broken component

---

## Chapter 4: Reproduce with Real Data

"Most prod bugs are data bugs," Sarah says.

Replay the request with production-like payloads. Compare against working cases.

---

## Chapter 5: Narrow the Change Set

Check the release diff.

"Changes are clues," Marcus says.

"If it broke yesterday, look at what changed yesterday."

---

## Chapter 6: Communicate and Document

Provide updates and track follow-ups.

"Communication is part of the fix," Sarah says.

"People forgive incidents. They don't forgive silence."

---

## Common Mistakes

1. **Skipping mitigation** - Fix later, stop the bleeding now.
2. **Assuming it's a frontend bug** - Check API and data changes too.
3. **No communication** - Stakeholders need updates.
4. **No follow-up** - Bugs repeat when lessons are lost.


Example:
```text
# ❌ Jump to root cause
"Let's rewrite the feature."

# ✅ Mitigate first
"Disable the feature flag, then investigate."
```

---

## Practice Exercises

1. Draft a triage checklist for a broken checkout flow
2. Write a mitigation plan using feature flags
3. Create a short incident update template

### Solutions

<details>
<summary>Exercise 1: Triage Checklist</summary>

```text
1) Confirm impact and scope
2) Check recent deploys
3) Review error logs
4) Attempt reproduction with prod data
```

**Key points:**
- Start with impact.
- Use evidence, not guesses.
- Move from broad to narrow.

</details>

<details>
<summary>Exercise 2: Mitigation Plan</summary>

```text
1) Disable checkout promo banner
2) Roll back the last UI release
3) Confirm conversion recovers
```

**Key points:**
- Mitigation first.
- Validate impact reduction.
- Communicate changes.

</details>

<details>
<summary>Exercise 3: Update Template</summary>

```text
Status: Investigating
Impact: Checkout blocked for 5% of users
Next update: 30 minutes
```

**Key points:**
- Keep updates short.
- Include impact and timing.
- Be consistent.

</details>

---

## What You Learned

This module covered:

- **Triage**: Prioritize by impact
- **Signals**: Use logs and traces
- **Mitigation**: Reduce harm quickly
- **Communication**: Keep stakeholders informed

**Key takeaway**: Production debugging is about impact control first, root cause second.

---

## Real-World Application

This week at work, you might use these concepts to:

- Triage a broken signup flow
- Use feature flags to mitigate a production bug
- Communicate incident updates clearly
- Identify a regression by comparing deploys
- Write a follow-up action list

---

## Further Reading

- [Google SRE Incident Response](https://sre.google/sre-book/incident-response/)
- [Atlassian Incident Management](https://www.atlassian.com/incident-management)
- [Sentry Incident Management](https://docs.sentry.io/product/alerts/incident-management/)

---

**Navigation**: [← Previous Module](./11-observability-monitoring.md) | [Next Module →](./13-code-review-mastery.md)
