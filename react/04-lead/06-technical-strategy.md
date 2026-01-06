# Technical Strategy

> **Last reviewed**: 2026-01-06


## Week 37: The Long Horizon

Two teams pitch new tools in the same week, and leadership wants a 12-month plan. Sarah asks you to shape a technical strategy that supports business goals without chasing every new idea. Marcus reminds you that strategy is a set of choices, not a list of technologies. This week focuses on evaluation frameworks, tech radars, roadmaps, and experiments that turn curiosity into accountable decisions.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Strategy** | A north star - focus the team on what matters most |
| **Evaluation** | A scorecard - compare options with shared criteria |
| **Tech radar** | A weather map - know what is safe and what is risky |
| **Roadmap** | A flight plan - sequence steps toward a destination |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 36 (Stakeholder Communication) - Comfort presenting trade-offs to different audiences.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Evaluate new technologies with explicit criteria
- [ ] Maintain a tech radar that signals adoption intent
- [ ] Tie technical initiatives to business outcomes
- [ ] Design experiments and POCs with success criteria
- [ ] Build a 12-18 month technical roadmap
- [ ] Communicate strategy decisions with clarity

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice strategic planning over 8-12 weeks

---

## Chapter 1: Start With the Problem

Sarah asks you to write the strategy in terms of business goals, not tooling preferences. Marcus suggests a short problem statement before any solution talk.

```markdown
## Problem Statement Template

**Problem**: What is slowing us down or creating risk?
**Impact**: Who is affected and how?
**Constraints**: Time, budget, compliance, or staffing limits
**Success**: What does a good outcome look like?
```

## Chapter 2: Evaluate Options with a Scorecard

Use a weighted scorecard to make the trade-offs explicit. This helps avoid debates that turn into opinions.

```markdown
## Technology Evaluation: [Technology Name]

### 1. Problem Statement
What problem are we trying to solve?

### 2. Current State
How do we solve this today? What is the cost of the status quo?

### 3. Proposed Solution
What is this technology and how does it solve the problem?

### 4. Evaluation Criteria

| Criterion | Weight | Score (1-5) | Weighted |
|-----------|--------|-------------|----------|
| Solves problem | 25% | | |
| Team capability | 20% | | |
| Community/support | 15% | | |
| Integration effort | 15% | | |
| Long-term viability | 15% | | |
| Cost | 10% | | |
| **Total** | 100% | | |

### 5. Risks and Mitigations
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| | | | |

### 6. Recommendation
Adopt / Trial / Assess / Hold
```

## Chapter 3: Maintain a Tech Radar

Sarah wants a single place that signals what the team should adopt, trial, or avoid. Review it quarterly.

```markdown
## Team Technology Radar

### Adopt
- React 18
- TypeScript
- Next.js App Router
- TanStack Query

### Trial
- Playwright
- Turborepo
- Module Federation

### Assess
- React Server Actions
- Bun
- Effect-TS

### Hold
- Create React App
- Enzyme
- Unmaintained UI libraries
```

## Chapter 4: Build Roadmaps Tied to Goals

Marcus suggests you connect each initiative to a business outcome. The roadmap should show sequence and dependencies, not just a wish list.

```markdown
## Frontend Technical Roadmap 2024-2025

### H1 2024: Foundation
- Migrate core pages to Next.js
- Establish shared design system
- Instrument Core Web Vitals

### H2 2024: Scale
- Improve build pipeline
- Add edge caching for product pages
- Expand integration tests

### 2025: Growth
- Multi-region support
- Personalization platform
- AI-assisted content workflows
```

```markdown
## Goal Alignment Example

**Business goal: Increase conversion by 10%**
- Performance improvements
- A/B testing platform
- Personalization capabilities

**Business goal: Reduce operational costs**
- Build time reduction
- Incident reduction via monitoring
- Dependency upgrades to reduce support load
```

## Chapter 5: Run Experiments and POCs

Sarah expects every experiment to have a hypothesis and exit criteria. POCs are tools for learning, not pre-approved decisions.

```markdown
## Experiment Template

**Hypothesis**
If we adopt X, we can reduce Y by Z.

**Success Criteria**
- Measurable outcome
- Performance constraints
- Team adoption signal

**Timeline**
Two-week spike

**Risks**
List top 3 risks and mitigation

**Go/No-Go Decision**
What evidence will trigger a decision?
```

```markdown
## POC: Real-time Collaboration

### Objective
Validate real-time collaboration for the editor.

### Scope
- Two users editing the same document
- Presence indicators
- Basic conflict resolution

### Out of Scope
- Mobile support
- Full feature parity

### Timeline
- Week 1: Prototype A
- Week 2: Prototype B
- Week 3: Comparison and recommendation

### Success Criteria
- < 100ms sync latency
- Handles 10 concurrent users
- Reasonable implementation complexity
```

## Chapter 6: Decide and Communicate

Marcus recommends a short decision memo that records the why, not just the what. This prevents reruns of the same debates.

```markdown
## Decision Memo

**Decision**: Adopt Next.js App Router for new pages
**Rationale**: Improves performance and aligns with future needs
**Risks**: Migration complexity, short-term velocity dip
**Mitigations**: Phased rollout, training, pilot project
**Next Steps**: Start pilot on marketing pages
```

---

## Common Mistakes

1. **Chasing shiny tech** - Strategy becomes a list of tools, not outcomes.
2. **No success criteria** - Experiments never end and decisions stall.
3. **Ignoring team capability** - Adoption fails when skills do not match ambition.
4. **No exit plan** - POCs become production without proving value.

## Practice Exercises

1. Create a technology radar for your team
2. Write a POC plan for a technology you are curious about
3. Draft a one-page technical strategy aligned to business goals

### Solutions

<details>
<summary>Exercise 1: Tech Radar</summary>

```markdown
Adopt: React 18, TypeScript, Next.js App Router
Trial: Playwright, Turborepo
Assess: React Server Actions
Hold: Create React App, Enzyme
```

**Key points:**
- Keep the list short and intentional.
- Review on a regular cadence.
- Use it to set adoption expectations.

</details>

<details>
<summary>Exercise 2: POC Plan</summary>

```markdown
## POC: Edge Image Optimization

Objective: Reduce image load time on product pages.
Scope: Product list and product detail images.
Timeline: Two-week spike.
Success Criteria: 30% smaller payload, no visual regressions.
Decision: Adopt if criteria met and ops cost acceptable.
```

**Key points:**
- Define success before you start.
- Keep POCs time-boxed.
- Decide based on evidence, not excitement.

</details>

<details>
<summary>Exercise 3: Strategy One-Pager</summary>

```markdown
## Frontend Strategy: Next 12 Months

Goals:
- Improve conversion by 10%
- Reduce incidents by 30%

Initiatives:
- Performance program (Core Web Vitals)
- Design system adoption
- Observability improvements

Risks:
- Migration complexity
- Team bandwidth

Ask:
Approve roadmap and staffing for Q1-Q2
```

**Key points:**
- Tie initiatives to measurable outcomes.
- Highlight risks early.
- End with a clear ask.

</details>

---

## What You Learned

This module covered:

- **Problem framing**: Start with outcomes and constraints
- **Evaluation**: Use scorecards to make trade-offs explicit
- **Tech radar**: Signal adoption intent and risk level
- **Roadmaps**: Sequence initiatives tied to business goals
- **Experiments**: Run time-boxed POCs with success criteria

**Key takeaway**: Strategy is choosing what to do and what not to do.

---

## Real-World Application

This week at work, you might use these concepts to:

- Evaluate a new framework proposal
- Publish a quarterly tech radar update
- Align a migration plan to revenue goals
- Pitch a POC with clear success criteria
- Document a strategic decision for future teams

---

## Further Reading

- [Technology Radar](https://www.thoughtworks.com/radar)
- [Good Strategy Bad Strategy](https://www.amazon.com/Good-Strategy-Bad-Difference-Matters/dp/0307886239)
- [Accelerate](https://www.oreilly.com/library/view/accelerate/9781457191435/)

---

**Navigation**: [<- Previous Module](./05-stakeholder-communication.md) | [Next Module ->](./07-incident-management.md)