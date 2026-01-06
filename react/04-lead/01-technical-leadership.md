# Technical Leadership

> **Last reviewed**: 2026-01-06


## Week 32: The Technical Vision Reset

The team is shipping, but direction is fuzzy. Sarah asks you to define a technical vision that aligns with product goals and reduces risk. Marcus reminds you that leadership is not about writing every line -- it is about setting direction, making trade-offs, and creating clarity. This week focuses on the systems and decisions that guide a team at scale.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Technical vision** | A compass - keeps the team pointed the same way |
| **Decision making** | A filter - weigh trade-offs, not opinions |
| **Roadmaps** | A route plan - sequence steps toward a destination |
| **Tech radar** | A weather report - know what is safe to adopt |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 31 (Framework Comparison) - Comfort evaluating trade-offs and documenting decisions.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Set a clear technical vision tied to business goals
- [ ] Build roadmaps that balance delivery and risk
- [ ] Make buy vs build decisions with explicit criteria
- [ ] Assess and communicate technical risk
- [ ] Document decisions so teams can move faster later
- [ ] Balance innovation with reliable delivery

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice technical leadership habits over 8-12 weeks

---

## Chapter 1: The Lead Developer Role

### What It Is

- Setting technical direction for your team/area
- Making architectural decisions
- Growing engineers and building culture
- Bridging engineering and business

### What It Isn't

- Writing all the code
- Making all the decisions
- Managing people (that's EM)
- Being the smartest person

## Chapter 2: Setting Technical Vision

### Vision Document Structure

```markdown
# Frontend Technical Vision 2024-2025

## Current State
- React 17, CRA-based monolith
- Performance issues on mobile
- Growing team (5 → 12 engineers)

## Target State
- Next.js App Router
- Core Web Vitals in "Good" range
- Feature teams with clear ownership

## Strategic Priorities
1. Migration to Next.js (Q1-Q2)
2. Design system v2 (Q2)
3. Performance monitoring (Q3)

## Success Metrics
- LCP < 2.5s on 75th percentile
- Developer velocity: 20% improvement
- Zero critical production issues

## Key Decisions
- ADR-001: Next.js over Remix
- ADR-002: Zustand over Redux
- ADR-003: Vercel deployment
```

### Communicating Vision

- **Executives**: Business impact, ROI, risk
- **Engineers**: Technical benefits, learning opportunities
- **Product**: Feature velocity, reliability improvements
- **Design**: Design system, consistency

## Chapter 3: Technical Roadmaps

### Roadmap Structure

```markdown
### Q1 2024 Roadmap

### Theme: Foundation

#### Migration to Next.js
- Week 1-2: Proof of concept
- Week 3-4: Core infrastructure
- Week 5-8: Migrate auth flows
- Week 9-12: Migrate dashboard

#### Dependencies
- DevOps: Vercel setup
- Backend: API versioning

#### Risks
- Team learning curve (medium)
- Production parity (low)
- Timeline slip (medium)

#### Exit Criteria
- All existing features work
- Performance regression tests pass
- Team trained on new patterns
```

### Balancing the Roadmap

```
┌──────────────────────────────────────────────┐
│                 Roadmap Mix                   │
├──────────────────────────────────────────────┤
│  Features (60%)    │ Tech Debt (25%) │ Innovation (15%)  │
│  - User value      │ - Refactoring   │ - POCs           │
│  - Business needs  │ - Upgrades      │ - New tech       │
│  - Revenue drivers │ - Cleanup       │ - Experiments    │
└──────────────────────────────────────────────┘
```

## Chapter 4: Buy vs Build Decisions

### Decision Framework

```markdown
### Evaluation: Auth Solution

### Options
1. **Build**: Custom auth system
2. **Buy**: Auth0
3. **Open Source**: NextAuth.js

### Criteria

| Factor | Build | Auth0 | NextAuth |
|--------|-------|-------|----------|
| Time to implement | 3 months | 2 weeks | 1 month |
| Ongoing maintenance | High | Low | Medium |
| Cost (year 1) | $50k dev time | $24k | $10k |
| Cost (year 2+) | $20k/year | $24k/year | $5k/year |
| Customization | Full | Limited | Good |
| Security expertise | Required | Included | Partial |

### Recommendation
Auth0 for now:
- Faster time to market
- Security handled by experts
- Can migrate later if needed

### Risks
- Vendor lock-in
- Cost at scale
- Feature limitations
```

## Chapter 5: Risk Assessment

### Risk Categories

| Category | Examples | Mitigation |
|----------|----------|------------|
| Technical | New framework, scaling | POCs, gradual rollout |
| Timeline | Dependencies, scope creep | Buffers, scope cuts |
| Team | Knowledge gaps, turnover | Training, documentation |
| External | Vendor changes, API deprecation | Contracts, abstraction |

### Risk Register

```markdown
| Risk | Probability | Impact | Score | Mitigation | Owner |
|------|-------------|--------|-------|------------|-------|
| Framework migration delays | Medium | High | 6 | Buffer weeks, parallel paths | Tech Lead |
| Key engineer leaves | Low | High | 4 | Documentation, pair programming | Tech Lead |
| API breaking changes | Medium | Medium | 4 | Version pinning, abstraction layer | Backend |
```

## Chapter 6: Making Decisions

### Decision Types

- **One-way door**: Hard to reverse, need careful analysis
- **Two-way door**: Easy to reverse, decide quickly

### Decision Log

```markdown
### Decision: State Management Library

**Date**: 2024-01-15
**Status**: Decided
**Deciders**: Tech Lead, Senior Engineers

**Context**
Need global state for auth, theme, and feature flags.

**Options Considered**
1. Redux Toolkit
2. Zustand
3. Jotai

**Decision**
Zustand

**Rationale**
- Simpler API reduces learning curve
- Small bundle size (1kb vs 11kb)
- TypeScript-first
- Sufficient for our use cases

**Consequences**
- Team needs training
- Create migration guide from current Context usage
```

## Chapter 7: Balancing Innovation and Delivery

### The 70-20-10 Rule

- **70%**: Core product work
- **20%**: Improvements and tech debt
- **10%**: Innovation and experiments

### Managing Tech Exploration

```markdown
## Tech Radar

### Adopt (Use in production)
- Next.js App Router
- TanStack Query
- Tailwind CSS

### Trial (Limited production use)
- Server Components
- Turborepo
- Playwright

### Assess (Evaluate in side projects)
- Bun
- Effect-TS
- RSC-only apps

### Hold (Don't start new projects)
- Create React App
- Redux (for new projects)
- Enzyme
```

---

## Common Mistakes

1. **Vision without metrics** - If success is not measurable, alignment fades quickly.
2. **Roadmaps without risk buffers** - Delivery timelines collapse when surprises hit.
3. **Undocumented decisions** - Teams re-argue the same choices every quarter.
4. **Innovation without guardrails** - Experiments consume time without clear exit criteria.

## Practice Exercises

1. Create a technical vision document for a migration
2. Build a decision matrix for a build vs buy scenario
3. Develop a tech radar for your team

### Solutions

<details>
<summary>Exercise 1: Vision Document</summary>

```markdown
# Frontend Vision 2025

## Current State
- Monolith build times > 20 minutes
- Mobile LCP at 4.0s (P75)

## Target State
- Modular architecture with owned features
- LCP < 2.5s and CLS < 0.1

## Strategic Priorities
1. Next.js migration (Q1-Q2)
2. Design system adoption (Q2)
3. Performance budgets (Q3)

## Success Metrics
- Build time < 10 minutes
- 20% increase in release frequency
```

**Key points:**
- Vision ties to measurable metrics.
- Priorities are staged, not simultaneous.
- Targets align with product outcomes.

</details>

<details>
<summary>Exercise 2: Decision Matrix</summary>

```markdown
| Criteria | Build | Buy | Notes |
|----------|------:|----:|------|
| Time to ship | 2 | 5 | Vendor is ready |
| Customization | 5 | 2 | Edge cases need support |
| Total cost | 3 | 4 | License vs dev time |
| Risk | 3 | 4 | Vendor has SOC2 |

Decision: Buy (score 15 vs 13)
```

**Key points:**
- Use weighted criteria if needed.
- Capture notes so the decision is auditable.
- Document why the winning option won.

</details>

<details>
<summary>Exercise 3: Tech Radar</summary>

```markdown
Adopt: Next.js App Router, TanStack Query
Trial: Playwright, Module Federation
Assess: Bun, Effect-TS
Hold: Create React App, Enzyme
```

**Key points:**
- Radar categories signal adoption intent.
- Keep the list short to avoid noise.
- Review and update quarterly.

</details>

---

## What You Learned

This module covered:

- **Vision**: Direction that aligns with business goals
- **Roadmaps**: Sequenced delivery with risk awareness
- **Decisions**: Trade-offs captured in writing
- **Risk**: Explicit assessment before committing
- **Innovation**: Experiments with guardrails

**Key takeaway**: Technical leadership is clarity at scale.

---

## Real-World Application

This week at work, you might use these concepts to:

- Draft a technical vision for the next quarter
- Build a buy vs build matrix for a vendor tool
- Define success metrics for a migration
- Publish a tech radar update
- Lead a roadmap review with stakeholders

---
## Further Reading

- [Staff Engineer](https://staffeng.com/book)
- [The Staff Engineer's Path](https://www.oreilly.com/library/view/the-staff-engineers/9781098118723/)

---

**Navigation**: [Next Module ->](./02-architecture-ownership.md)