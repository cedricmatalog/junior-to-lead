# Stakeholder Communication

> **Last reviewed**: 2026-01-06


## Week 36: The Alignment Gap

The roadmap is moving fast, but the room is not aligned. Sarah asks you to brief leadership on a risky migration, and Marcus warns that great work fails when it is misunderstood. You need to translate technical decisions into business outcomes, keep partners informed, and surface trade-offs early. This week focuses on audience-aware messaging, crisp docs, and the habit of delivering news with options instead of surprises.

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Audience framing** | A lens - adjust the view for the listener |
| **Technical writing** | A map - guide people to decisions |
| **Status updates** | A dashboard - highlight green, yellow, red |
| **Expectations** | A contract - align before delivery |

Keep these in mind. They'll click as we build.

---

## Prerequisites

Module 35 (Hiring and Onboarding) - Comfort with structured communication.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Adapt communication to stakeholder needs
- [ ] Write clear, actionable technical documents
- [ ] Present to leadership with a clear ask and options
- [ ] Set expectations and capture trade-offs early
- [ ] Deliver bad news with recovery paths
- [ ] Maintain alignment through consistent updates

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice stakeholder communication over 8-12 weeks

---

## Chapter 1: Map the Audience

Sarah asks for a quick update for the executive team and a deeper walkthrough for product. Marcus reminds you that the same message lands differently depending on who is listening. Start by mapping what each group cares about and the level of detail they need.

### Communication Matrix

| Audience | Care About | Avoid | Format |
|----------|-----------|-------|--------|
| Executives | Business impact, risk, ROI | Implementation details | Summary and ask |
| Product | Timeline, scope, trade-offs | Deep technical jargon | Options and recommendation |
| Design | Feasibility, constraints | Low-level implementation | Visuals and collaboration |
| Engineers | How it works, why | Hand-waving | Detailed and interactive |

## Chapter 2: Translate Tech Into Impact

Marcus shows you how to turn a technical decision into a business outcome. Start with the problem, then explain the impact, then propose the change.

```markdown
## Same Update, Different Audiences

### To Engineering
"We're migrating from Redux to Zustand. This simplifies state
management, reduces bundle size by 11kb, and aligns with modern
React patterns. Migration plan in the RFC. Please review by Friday."

### To Product
"We're updating our state management library. This will take 3 sprints
but reduces ongoing maintenance and makes new features faster to build.
No user-facing changes. All existing functionality is preserved."

### To Executives
"We're investing 3 sprints in frontend infrastructure improvements.
This reduces technical risk, improves developer velocity by about 15%,
and positions us for the mobile initiative next quarter."
```

## Chapter 3: Write Docs That Drive Decisions

Sarah wants decisions captured in a format that is easy to scan and hard to misinterpret. Keep docs short, lead with the recommendation, and make the ask explicit.

### RFC Template

```markdown
# RFC: Migrate to Next.js

## Summary
One paragraph explaining the proposal.

## Motivation
Why are we doing this? What problem does it solve?

## Detailed Design
How will it work? What are the specifics?

## Alternatives Considered
What else did we consider? Why not those?

## Rollout Plan
How do we get from here to there?

## Open Questions
What don't we know yet?
```

### Status Update Template

```markdown
# Project Status: Week 3

## Summary
On track for Q1 delivery. One risk item to discuss.

## Progress
- Phase 1 complete (auth migration)
- Phase 2 in progress (dashboard)
- Phase 3 not started (settings)

## Key Metrics
- 12 of 24 routes migrated
- Zero production incidents
- Team velocity stable

## Risks
- API dependency on backend team (Medium)
  - Mitigation: Daily sync, shared timeline

## Next Week
- Complete dashboard migration
- Begin settings migration
```

## Chapter 4: Present to Leadership

Marcus suggests you lead with the answer and offer details only if asked. Keep the ask crisp and repeat it at the end.

### The Pyramid Principle

```
Answer / recommendation
        |
    Key points
        |
   Supporting details
```

### Executive Presentation Outline

```markdown
## Slide 1: The Ask
"We are requesting approval for a 3-month infrastructure project
that will improve performance and developer productivity."

## Slide 2: The Why
- Current state: Slow page loads, frustrated users
- Business impact: Higher bounce rate, lower conversion
- Opportunity: 20% performance improvement, 15% faster feature delivery

## Slide 3: The How (High Level)
- Migrate to a modern framework
- Implement caching strategy
- Improve build pipeline

## Slide 4: The Investment
- 3 months, 2 engineers
- $X in tooling costs
- Opportunity cost: Delay Feature Y by 1 sprint

## Slide 5: The Return
- LCP improvement: 4s to 2s
- Developer velocity: +15%
- SEO ranking improvement expected

## Slide 6: The Timeline
Simple Gantt chart with major milestones

## Slide 7: The Ask (Repeated)
"Do we have approval to proceed?"
```

## Chapter 5: Set Expectations and Deliver Bad News

Sarah wants expectations set early, before trade-offs become surprises. When bad news hits, share it quickly with options.

### Setting Expectations Early

```markdown
## Project Kickoff: Setting Expectations

"For this migration:

**What will change:**
- Faster page loads for users
- New patterns for developers to learn
- Temporary slowdown in feature velocity

**What will not change:**
- All existing features still work
- User-facing URLs stay the same
- No changes to data or security

**Timeline:**
- Week 1-4: Foundation (behind the scenes)
- Week 5-8: Visible changes start
- Week 9-12: Migration complete

**What I need from you:**
- Feedback on each phase
- Quick decisions on trade-offs
- Understanding that some things will break and we will fix them"
```

### Delivering Bad News

```markdown
## Delivering Bad News Template

1. **State it directly**
   "We are going to miss the Q1 deadline for the migration."

2. **Own it**
   "I underestimated the complexity of the auth flow."

3. **Explain briefly**
   "We discovered legacy code that was not documented."

4. **Present the path forward**
   "We can still deliver core migration in Q1, with auth
   following 2 weeks later. Or we can delay everything to Q2."

5. **Ask for what you need**
   "I recommend option 1. Do you agree?"
```

---

## Common Mistakes

1. **One-size-fits-all updates** - The same message does not work for every audience.
2. **Burying the ask** - If the decision is unclear, alignment stalls.
3. **Hiding risks** - Surprises break trust faster than delays.
4. **Overloading status** - Too much detail makes the signal hard to find.

## Practice Exercises

1. Rewrite a technical update for product and executive audiences
2. Draft a one-page decision doc for a migration
3. Deliver a bad-news update with two options

### Solutions

<details>
<summary>Exercise 1: Audience Rewrite</summary>

```markdown
### Engineering
"We will replace Redux with Zustand to reduce boilerplate and simplify
state. Migration is scoped to the checkout flow and will happen over
2 sprints. Review the RFC by Friday."

### Product
"We are updating our state management tooling. The change takes
2 sprints and reduces long-term maintenance so features move faster.
No user-visible changes are expected."

### Executives
"We are investing 2 sprints in frontend infrastructure to reduce
technical risk and improve delivery speed by about 10-15%."
```

**Key points:**
- Lead with the outcome, not the implementation.
- Use different levels of detail per audience.
- Keep the ask explicit and time-bound.

</details>

<details>
<summary>Exercise 2: One-Page Decision Doc</summary>

```markdown
# Decision: Migrate to Next.js

## Summary
Move from CRA to Next.js to improve performance and SEO.

## Recommendation
Adopt Next.js App Router for new and migrated pages.

## Benefits
- 2s faster LCP on mobile
- Server rendering for SEO-sensitive pages
- Shared routing conventions

## Costs
- 3 months of migration
- Team training and tooling updates

## Risks
- Migration complexity
- Temporary velocity dip

## Decision Needed
Approve migration plan and resourcing.
```

**Key points:**
- One page, no hidden asks.
- Trade-offs are explicit.
- Decision is easy to find.

</details>

<details>
<summary>Exercise 3: Bad-News Update</summary>

```markdown
"We will miss the Q1 deadline for the migration.
The auth system is more complex than expected.

Option A: Ship core migration in Q1, auth completes in Q2.
Option B: Delay full migration to Q2 for a single cutover.

I recommend option A so we still deliver performance gains this quarter.
Can we align on that today?"
```

**Key points:**
- State the issue clearly.
- Offer options, not just problems.
- Close with a concrete ask.

</details>

---

## What You Learned

This module covered:

- **Audience framing**: Tailor detail level to who is listening
- **Technical writing**: Docs that drive decisions instead of debate
- **Executive communication**: Lead with the answer and repeat the ask
- **Expectation management**: Align on scope, risk, and trade-offs early
- **Bad-news delivery**: Share problems quickly with recovery paths

**Key takeaway**: Clear communication prevents surprises and keeps trust intact.

---

## Real-World Application

This week at work, you might use these concepts to:

- Draft a leadership update for a migration
- Write a short RFC for a new platform decision
- Reframe a technical issue in business terms
- Run a project kickoff that sets expectations
- Deliver a schedule slip with options

---

## Further Reading

- [The Pyramid Principle](https://www.amazon.com/Pyramid-Principle-Logic-Writing-Thinking/dp/0273710516)
- [Writing for Engineers](https://www.heinrichhartmann.com/posts/writing/)
- [PRFAQ](https://www.allthingsdistributed.com/2006/11/working_backwards.html)

---

**Navigation**: [<- Previous Module](./04-hiring-onboarding.md) | [Next Module ->](./06-technical-strategy.md)
