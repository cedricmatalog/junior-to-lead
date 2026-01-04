# Stakeholder Communication

Communicate technical concepts effectively to any audience.

## Learning Objectives

By the end of this module, you will:
- Adapt technical communication to your audience
- Write clear technical documents
- Present effectively to leadership
- Manage expectations proactively

## Know Your Audience

### Communication Matrix

| Audience | Care About | Avoid | Format |
|----------|-----------|-------|--------|
| Executives | Business impact, ROI, risk | Implementation details | Summary + ask |
| Product | Timeline, scope, trade-offs | Technical jargon | Options + recommendation |
| Design | Feasibility, constraints | Deep tech details | Visual + collaborative |
| Engineers | How it works, why | Hand-waving | Detailed + interactive |

### Tailoring the Message

```markdown
## Same Update, Different Audiences

### To Engineering
"We're migrating from Redux to Zustand. This simplifies our state
management, reduces bundle size by 11kb, and aligns with modern
React patterns. Migration plan in the RFC - please review by Friday."

### To Product
"We're updating our state management library. This will take 3 sprints
but reduces ongoing maintenance and makes new features faster to build.
No user-facing changes - all existing functionality preserved."

### To Executives
"We're investing 3 sprints in frontend infrastructure improvements.
This reduces technical risk, improves developer velocity by ~15%,
and positions us better for the mobile initiative next quarter."
```

## Technical Writing

### Writing for Clarity

```markdown
## Before (Too Technical)
"We need to implement SSR with ISR to improve TTFB and LCP scores,
which are currently in the 'poor' range according to CrUX data.
This requires migrating from CRA to Next.js."

## After (Clear)
"Our website is slow to load, which hurts user experience and SEO.
The solution is to move to a new framework (Next.js) that renders
pages faster. This is a 3-month project with significant benefits."

## Key Changes:
- Explained acronyms or avoided them
- Started with the problem, not the solution
- Gave concrete impact and timeline
```

### Document Structures

**RFC (Request for Comments)**
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

**Status Update**
```markdown
# Project Status: Week 3

## Summary
On track for Q1 delivery. One risk item to discuss.

## Progress
- ✅ Phase 1 complete (auth migration)
- 🔄 Phase 2 in progress (dashboard)
- ⏳ Phase 3 not started (settings)

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

## Presenting to Leadership

### The Pyramid Principle

```
Start with the answer/recommendation
              │
              ▼
     Support with key points
              │
              ▼
   Details only if asked
```

### Executive Presentation Template

```markdown
## Slide 1: The Ask
"We're requesting approval for a 3-month infrastructure project
that will improve performance and developer productivity."

## Slide 2: The Why
- Current state: Slow page loads, frustrated users
- Business impact: Higher bounce rate, lower conversion
- Opportunity: 20% performance improvement, 15% faster feature delivery

## Slide 3: The How (High Level)
- Migrate to modern framework
- Implement caching strategy
- Improve build pipeline

## Slide 4: The Investment
- 3 months, 2 engineers
- $X in tooling costs
- Opportunity cost: Delay Feature Y by 1 sprint

## Slide 5: The Return
- LCP improvement: 4s → 2s
- Developer velocity: +15%
- SEO ranking improvement expected

## Slide 6: The Timeline
[Simple Gantt chart]

## Slide 7: The Ask (Repeated)
"Do we have approval to proceed?"
```

### Handling Questions

```markdown
## Common Executive Questions

**"Can we do it faster?"**
"We could cut scope to X, but that means losing Y benefit.
Alternatively, we could add resources, but that has Z trade-off."

**"What's the risk if we don't do this?"**
"We'll continue to see performance degrade, which impacts
user satisfaction and SEO. Competitor A recently made this
investment and saw 20% improvement in conversion."

**"Why now?"**
"The current architecture is blocking Feature Z, which is
on our roadmap for Q2. Addressing this now clears that path."

**"What could go wrong?"**
"Main risks are timeline slip and temporary regression.
We're mitigating with phased rollout and feature flags."
```

## Managing Expectations

### Setting Expectations Early

```markdown
## Project Kickoff: Setting Expectations

"For this migration:

**What will change:**
- Faster page loads for users
- New patterns for developers to learn
- Some temporary slowdown in feature velocity

**What won't change:**
- All existing features will work
- User-facing URLs stay the same
- No changes to our data or security

**Timeline:**
- Week 1-4: Foundation (behind the scenes)
- Week 5-8: Visible changes start
- Week 9-12: Migration complete

**What I need from you:**
- Feedback on each phase
- Quick decisions on trade-offs
- Understanding that some things will break and we'll fix them"
```

### Delivering Bad News

```markdown
## Delivering Bad News Template

1. **State it directly**
   "We're going to miss the Q1 deadline for the migration."

2. **Own it**
   "I underestimated the complexity of the auth flow."

3. **Explain (briefly)**
   "We discovered legacy code that wasn't documented."

4. **Present the path forward**
   "We can still deliver the core migration in Q1, with auth
   following 2 weeks later. Or we can delay everything to Q2."

5. **Ask for what you need**
   "I recommend Option 1. Do you agree?"
```

## Practice Exercises

1. Rewrite a technical document for a non-technical audience
2. Prepare a 5-minute presentation for executives
3. Practice delivering bad news constructively

## Further Reading

- [The Pyramid Principle](https://www.amazon.com/Pyramid-Principle-Logic-Writing-Thinking/dp/0273710516)
- [Writing for Engineers](https://www.heinrichhartmann.com/posts/writing/)
