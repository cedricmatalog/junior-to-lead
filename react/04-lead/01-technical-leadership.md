# Technical Leadership

Lead through expertise, vision, and influence.

## Learning Objectives

By the end of this module, you will:
- Set technical vision and direction
- Make buy vs build decisions
- Assess and manage technical risk
- Balance innovation with delivery

## The Lead Developer Role

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

## Setting Technical Vision

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

## Technical Roadmaps

### Roadmap Structure

```markdown
## Q1 2024 Roadmap

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

## Buy vs Build Decisions

### Decision Framework

```markdown
## Evaluation: Auth Solution

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

## Risk Assessment

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

## Making Decisions

### Decision Types

- **One-way door**: Hard to reverse, need careful analysis
- **Two-way door**: Easy to reverse, decide quickly

### Decision Log

```markdown
## Decision: State Management Library

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

## Balancing Innovation and Delivery

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

## Practice Exercises

1. Create a technical vision document for a migration
2. Build a decision matrix for a build vs buy scenario
3. Develop a tech radar for your team

## Further Reading

- [Staff Engineer](https://staffeng.com/book)
- [The Staff Engineer's Path](https://www.oreilly.com/library/view/the-staff-engineers/9781098118723/)
