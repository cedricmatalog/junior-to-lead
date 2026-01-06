# Architecture Decision Records (ADRs)

> **Last reviewed**: 2026-01-06


## Week Two: The Decision You Can't Remember

Six months ago, the team chose a new state library. Today, nobody remembers why.

"That's not a memory problem," Sarah says. "That's a documentation problem."

Marcus adds, "ADRs make decisions discoverable. If you can't explain it later, you didn't really decide it."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **ADR** | A receipt - proof of purchase and why it was made |
| **Decision status** | A traffic light - proposed, accepted, deprecated |
| **Context** | The weather report - why the decision mattered |
| **Consequences** | The fine print - what you trade for the choice |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 01 (Architecture Patterns) - Comfort with architecture decisions and tradeoffs.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Explain when to write an ADR
- [ ] Use a lightweight ADR template
- [ ] Capture context, options, and consequences clearly
- [ ] Link ADRs to code and migrations
- [ ] Maintain ADR status over time
- [ ] Avoid decision drift in teams

---

## Time Estimate

- **Reading**: 55-75 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice ADRs over 4-6 weeks

---

## Chapter 1: Why Decisions Disappear

A new hire asks why the project uses module federation. Nobody knows.

"If it isn't written down, it's lore," Sarah says.

---

## Chapter 2: The ADR Template

Marcus shares a short template:

```text
# ADR-012: Use React Query for server state

## Context
Problem and constraints.

## Decision
What you chose and why.

## Alternatives
What you considered.

## Consequences
Tradeoffs and follow-up work.
```

"Short is the point. Write it while the decision is fresh."

---

## Chapter 3: When to Write One

"Write ADRs for high-impact, hard-to-reverse choices," Sarah says.

Examples:
- New framework
- API design shifts
- Data ownership changes
- Large refactors

---

## Chapter 4: Status and Lifecycle

Decisions evolve.

```text
Status: Proposed → Accepted → Deprecated → Superseded
```

"If you replace a decision, reference the old ADR instead of deleting it."

---

## Chapter 5: Linking ADRs to Code

Tie decisions to pull requests and migration plans.

"If a decision changes code, the code should link back," Marcus says.

---

## Chapter 6: Keeping ADRs Alive

You add a lightweight review step.

"Review ADRs quarterly, update status, and archive what no longer applies."

---

## Common Mistakes

1. **Writing essays** - ADRs should be short and scoped.
2. **Skipping alternatives** - Without options, you lose context.
3. **Hiding ADRs** - Put them near the code, not in a private doc.
4. **Never updating status** - Stale ADRs mislead future teams.


Example:
```text
# ❌ No alternatives
Decision: Use GraphQL because it's modern.

# ✅ Alternatives listed
Decision: Use GraphQL to reduce over-fetching.
Alternatives: REST + BFF, tRPC.
```

---

## Practice Exercises

1. Draft an ADR for a new state tool
2. Add an ADR status update after a migration
3. Link an ADR to a PR description

### Solutions

<details>
<summary>Exercise 1: Draft ADR</summary>

```text
# ADR-021: Adopt TanStack Query

## Context
Server state is duplicated across components.

## Decision
Use TanStack Query for caching and refetching.

## Alternatives
Redux Toolkit RTK Query, SWR.

## Consequences
Adds a dependency; simplifies loading states.
```

**Key points:**
- Keep it short.
- Record alternatives.
- Capture tradeoffs.

</details>

<details>
<summary>Exercise 2: Status Update</summary>

```text
Status: Superseded by ADR-022
Reason: New API gateway replaced the previous design.
```

**Key points:**
- Never delete old ADRs.
- Reference the new decision.
- Keep history intact.

</details>

<details>
<summary>Exercise 3: PR Link</summary>

```text
This change implements ADR-021 (TanStack Query adoption).
```

**Key points:**
- Trace decisions to code.
- Make reviews faster.
- Keep context visible.

</details>

---

## What You Learned

This module covered:

- **Purpose**: ADRs preserve decision context
- **Template**: Context, decision, alternatives, consequences
- **Lifecycle**: Statuses keep decisions current
- **Linking**: Tie ADRs to code and migration plans

**Key takeaway**: ADRs turn architectural decisions into durable team knowledge.

---

## Real-World Application

This week at work, you might use these concepts to:

- Document a framework or library decision
- Record why a migration was chosen
- Update an ADR after a major change
- Share decision context in a PR
- Prevent repeated debates on settled choices

---

## Further Reading

- [ADR GitHub Repo](https://github.com/joelparkerhenderson/architecture-decision-record)
- [ThoughtWorks: ADRs](https://www.thoughtworks.com/radar/techniques/lightweight-architecture-decision-records)
- [Michael Nygard: ADRs](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions.html)

---

**Navigation**: [← Previous Module](./01-architecture-patterns.md) | [Next Module →](./03-system-design.md)
