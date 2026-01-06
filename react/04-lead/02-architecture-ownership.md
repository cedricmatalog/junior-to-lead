# Architecture Ownership

> **Last reviewed**: 2026-01-06


## Week Two: Owning the Long View

You are now accountable for the long-term health of the frontend platform. Sarah asks you to make architecture decisions visible, keep debt from accumulating, and plan migrations without disrupting delivery. Marcus reminds you that ownership means creating systems that survive team changes. This week focuses on ADRs, tech debt management, and platform thinking.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **ADRs** | A flight log - record why choices were made |
| **Tech debt** | Interest - it compounds if ignored |
| **Migrations** | A bridge - move safely without collapse |
| **Platform thinking** | Utilities - shared infrastructure for everyone |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Lead Module 01 (Technical Leadership) - Comfort setting vision and documenting decisions.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Write ADRs that capture decisions and trade-offs
- [ ] Build a tech debt inventory and prioritize it
- [ ] Plan migrations with phases and rollbacks
- [ ] Define platform services that scale across teams
- [ ] Communicate architecture ownership to stakeholders

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice architecture ownership over 8-12 weeks

---

## Chapter 1: Architecture Decision Records (ADRs)

Sarah says, "If it's not written down, it's not a decision."

You open a fresh ADR template so the decision is explicit.

### ADR Template

```markdown
# ADR-001: Use Next.js App Router

### Status
Accepted

### Date
2024-01-15

### Context
We need to rebuild our frontend with better performance, SEO,
and developer experience. The current CRA-based app has
performance issues and no SSR capability.

### Decision
We will use Next.js with the App Router for our new frontend.

### Considered Alternatives

### 1. Next.js Pages Router
- Pros: Stable, well-documented
- Cons: Being superseded, less performant

### 2. Remix
- Pros: Great DX, web standards
- Cons: Smaller ecosystem, team unfamiliar

### 3. Keep CRA
- Pros: No migration cost
- Cons: Doesn't solve core issues

### Consequences

### Positive
- Server Components reduce client bundle
- Built-in performance optimizations
- Better SEO capabilities
- Vercel deployment simplicity

### Negative
- Team learning curve
- Migration effort required
- Some community packages not compatible yet

### Neutral
- Different mental model for data fetching
- New project structure

### References
- [Next.js App Router Docs](https://nextjs.org/docs/app)
- Team RFC: Frontend Migration Proposal
```

### ADR Organization

```
docs/
└── architecture/
    └── decisions/
        ├── README.md           # Index of all ADRs
        ├── 0001-nextjs-app-router.md
        ├── 0002-zustand-state.md
        ├── 0003-testing-strategy.md
        └── template.md         # Template for new ADRs
```

## Chapter 2: Technical Debt Management

Marcus says, "Debt is fine when it's intentional and tracked."

You categorize debt before deciding what to pay down.

### Categorizing Debt

| Type | Description | Example |
|------|-------------|---------|
| Deliberate | Conscious trade-off | Ship MVP without tests |
| Accidental | Didn't know better | Used deprecated API |
| Bit rot | Environment changed | Outdated dependencies |
| Cruft | Accumulation | Dead code, unused features |

### Tech Debt Register

You keep a living register so trade-offs stay visible.

```markdown
### Tech Debt Inventory

### High Priority
| Item | Impact | Effort | Owner | Target |
|------|--------|--------|-------|--------|
| React 17 → 18 | Blocking new features | 2 weeks | @alice | Q1 |
| Remove Redux | Complexity, bundle | 3 weeks | @bob | Q1 |

### Medium Priority
| Item | Impact | Effort | Owner | Target |
|------|--------|--------|-------|--------|
| Test coverage | Regression risk | Ongoing | Team | Q2 |
| API types | Type errors | 1 week | @carol | Q2 |

### Low Priority
| Item | Impact | Effort | Owner | Target |
|------|--------|--------|-------|--------|
| Dead code cleanup | Confusion | 2 days | Anyone | Anytime |
```

### Debt Payment Strategy

You set a budget so debt work does not get crowded out.

```
┌─────────────────────────────────────────────┐
│            Sprint Allocation                 │
├─────────────────────────────────────────────┤
│  Features │ Bugs  │ Tech Debt │ Innovation │
│    60%    │  15%  │    20%    │     5%     │
└─────────────────────────────────────────────┘

Adjust based on situation:
- Crunch time: Reduce debt to 10%
- Stability focus: Increase debt to 30%
- Post-incident: Focus on root cause
```

## Chapter 3: Migration Strategies

Sarah says, "Migrate in slices, not in a single leap."

You map the slices so the team can move safely.

### The Strangler Fig Pattern

```
Phase 1: New system alongside old
┌─────────────────────────────────────────┐
│                Router                    │
│    /new/* → New App    /* → Old App     │
└─────────────────────────────────────────┘

Phase 2: Migrate routes incrementally
┌─────────────────────────────────────────┐
│                Router                    │
│  /new/*, /users/* → New    /* → Old    │
└─────────────────────────────────────────┘

Phase 3: Old app becomes fallback
┌─────────────────────────────────────────┐
│                Router                    │
│        /* → New (most routes)           │
│        /legacy/* → Old (few routes)     │
└─────────────────────────────────────────┘

Phase 4: Decommission old
┌─────────────────────────────────────────┐
│              New App Only                │
└─────────────────────────────────────────┘
```

### Migration Plan Template

You write a phased plan before touching code.

```markdown
# Migration Plan: Redux to Zustand

### Overview
Replace Redux with Zustand for simpler state management.

### Scope
- Auth state
- User preferences
- Shopping cart
- NOT: Server state (already using TanStack Query)

### Phases

### Phase 1: Setup (Week 1)
- [ ] Add Zustand to project
- [ ] Create stores parallel to Redux slices
- [ ] Add feature flag for switching

### Phase 2: Migrate Auth (Week 2)
- [ ] Convert auth slice to Zustand store
- [ ] Update all auth consumers
- [ ] Test auth flows thoroughly
- [ ] Monitor for issues

### Phase 3: Migrate Remaining (Week 3-4)
- [ ] User preferences
- [ ] Shopping cart
- [ ] Remove Redux dependencies

### Phase 4: Cleanup (Week 5)
- [ ] Remove Redux packages
- [ ] Update documentation
- [ ] Team training

### Rollback Plan
Feature flag allows instant rollback to Redux.

### Success Criteria
- All tests pass
- No regression in functionality
- Bundle size reduced by 10kb
- Team comfortable with new patterns
```

## Chapter 4: Platform Thinking

Marcus says, "Platforms reduce repeat work across teams."

You frame platform work as leverage, not overhead.

### Platform vs Product

```
Product Thinking:        Platform Thinking:
"Build feature X"        "Enable teams to build features like X"

Build one thing          Build reusable capabilities
Solve one problem        Solve class of problems
Single team uses         Multiple teams use
```

### Internal Platforms

You list the shared services teams will use every week.

## Chapter 5: Frontend Platform Services

Sarah says, "Services should remove friction, not add gates."

You break the platform into services the team can own.

### Design System
- Component library
- Design tokens
- Documentation site
- Self-service: Teams pick components

### Build Infrastructure
- CI/CD pipelines
- Preview deployments
- Performance budgets
- Self-service: Teams configure via yaml

### Observability
- Error tracking
- Performance monitoring
- Analytics
- Self-service: Teams add dashboards

### Developer Experience
- Local development setup
- Code generators
- Documentation
- Self-service: Teams contribute

### Platform Team Model

```
┌─────────────────────────────────────────┐
│            Feature Teams                 │
│  Team A    Team B    Team C    Team D   │
└─────────────┬───────────────────────────┘
              │ Use
              ▼
┌─────────────────────────────────────────┐
│          Platform Team                   │
│  - Design System                         │
│  - Build Tools                           │
│  - Shared Libraries                      │
│  - Developer Experience                  │
└─────────────────────────────────────────┘
```

---

## Common Mistakes

1. **Decisions without documentation** - Context disappears and teams repeat debates.
2. **Debt without owners** - Backlogs grow when nobody is accountable.
3. **Migration without rollback** - Risk spikes when plans are irreversible.
4. **Platform without adoption** - Shared services fail if teams cannot onboard easily.

## Practice Exercises

1. Write an ADR for a recent technical decision
2. Create a tech debt inventory and prioritization
3. Plan a migration for a real codebase

### Solutions

<details>
<summary>Exercise 1: ADR</summary>

```markdown
# ADR-014: Adopt TanStack Query

## Context
We need consistent caching for server data across multiple features.

## Decision
Adopt TanStack Query for all new server-state features.

## Consequences
- Train team on query patterns
- Migrate existing fetch logic over 2 quarters
```

**Key points:**
- Capture context, decision, and consequences.
- Keep the ADR short and searchable.
- Link to relevant discussions.

</details>

<details>
<summary>Exercise 2: Tech Debt Inventory</summary>

```markdown
| Item | Impact | Effort | Priority |
|------|--------|--------|----------|
| Legacy router v5 | High | Medium | P1 |
| Unused feature flags | Medium | Low | P2 |
| Missing error boundaries | High | High | P1 |
```

**Key points:**
- Use impact and effort to prioritize.
- Review quarterly with the team.
- Assign owners to top items.

</details>

<details>
<summary>Exercise 3: Migration Plan</summary>

```markdown
Phase 1: Build parallel route in new stack
Phase 2: Mirror traffic (read-only)
Phase 3: Gradual rollout (10% -> 100%)
Rollback: Shift traffic back, keep data compatible
```

**Key points:**
- Phase work to reduce risk.
- Rollback plan is required, not optional.
- Measure success criteria at each phase.

</details>

---

## What You Learned

This module covered:

- **ADRs**: Documenting decisions and trade-offs
- **Tech debt**: Visible prioritization and ownership
- **Migrations**: Phased execution with rollback
- **Platform thinking**: Shared services for scale

**Key takeaway**: Architecture ownership is stewardship over time.

---

## Real-World Application

This week at work, you might use these concepts to:

- Publish an ADR for a tooling decision
- Create a tech debt backlog with priorities
- Define a migration plan with success metrics
- Propose a platform service for shared analytics

---
## Further Reading

- [ADR GitHub Organization](https://adr.github.io/)
- [Team Topologies](https://teamtopologies.com/)

---

**Navigation**: [← Previous Module](./01-technical-leadership.md) | [Next Module →](./03-team-processes.md)
