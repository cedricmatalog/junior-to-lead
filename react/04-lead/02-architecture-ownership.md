# Architecture Ownership

Own the technical direction and long-term health of systems.

## Learning Objectives

By the end of this module, you will:
- Write and maintain Architecture Decision Records
- Manage technical debt strategically
- Plan and execute migrations
- Think in platforms, not just features

## Architecture Decision Records (ADRs)

### ADR Template

```markdown
# ADR-001: Use Next.js App Router

## Status
Accepted

## Date
2024-01-15

## Context
We need to rebuild our frontend with better performance, SEO,
and developer experience. The current CRA-based app has
performance issues and no SSR capability.

## Decision
We will use Next.js with the App Router for our new frontend.

## Considered Alternatives

### 1. Next.js Pages Router
- Pros: Stable, well-documented
- Cons: Being superseded, less performant

### 2. Remix
- Pros: Great DX, web standards
- Cons: Smaller ecosystem, team unfamiliar

### 3. Keep CRA
- Pros: No migration cost
- Cons: Doesn't solve core issues

## Consequences

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

## References
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

## Technical Debt Management

### Categorizing Debt

| Type | Description | Example |
|------|-------------|---------|
| Deliberate | Conscious trade-off | Ship MVP without tests |
| Accidental | Didn't know better | Used deprecated API |
| Bit rot | Environment changed | Outdated dependencies |
| Cruft | Accumulation | Dead code, unused features |

### Tech Debt Register

```markdown
## Tech Debt Inventory

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

## Migration Strategies

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

```markdown
# Migration Plan: Redux to Zustand

## Overview
Replace Redux with Zustand for simpler state management.

## Scope
- Auth state
- User preferences
- Shopping cart
- NOT: Server state (already using TanStack Query)

## Phases

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

## Rollback Plan
Feature flag allows instant rollback to Redux.

## Success Criteria
- All tests pass
- No regression in functionality
- Bundle size reduced by 10kb
- Team comfortable with new patterns
```

## Platform Thinking

### Platform vs Product

```
Product Thinking:        Platform Thinking:
"Build feature X"        "Enable teams to build features like X"

Build one thing          Build reusable capabilities
Solve one problem        Solve class of problems
Single team uses         Multiple teams use
```

### Internal Platforms

```markdown
## Frontend Platform Services

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
```

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

## Practice Exercises

1. Write an ADR for a recent technical decision
2. Create a tech debt inventory and prioritization
3. Plan a migration for a real codebase

## Further Reading

- [ADR GitHub Organization](https://adr.github.io/)
- [Team Topologies](https://teamtopologies.com/)
