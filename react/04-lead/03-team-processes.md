# Team Processes

> **Last reviewed**: 2026-01-06


## Week Three: The Process Tune-Up

The team is shipping, but the rhythm feels chaotic. Sarah asks you to tune the process so delivery is predictable and sustainable. Marcus reminds you that process is not ceremony for its own sake -- it is a system that protects focus and clarity. This week is about adapting agile practices, running planning and retros, and building team rhythms that work.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Agile practices** | Guardrails - keep the team on track |
| **Planning** | A calendar - align work to capacity |
| **Retrospectives** | A mirror - see what to improve |
| **Team rhythms** | A heartbeat - steady cadence over time |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Lead Module 02 (Architecture Ownership) - Comfort with cross-team alignment and documentation.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Adapt agile practices to your team context
- [ ] Run effective planning and estimation
- [ ] Facilitate retrospectives that produce action
- [ ] Define and enforce a clear definition of done
- [ ] Establish sustainable team rhythms

---

## Time Estimate

- **Reading**: 50-70 minutes
- **Exercises**: 2-3 hours
- **Mastery**: Practice process leadership over 6-8 weeks

---

## Chapter 1: Agile in Practice

You want process to feel like momentum, not meetings.

Sarah says, "Process should serve delivery, not the other way around."

### What Actually Matters

| Principle | Practice |
|-----------|----------|
| Deliver frequently | CI/CD, small PRs |
| Welcome change | Flexible planning, modular code |
| Working software | Definition of done, demos |
| Collaboration | Pairing, code review, standups |
| Sustainable pace | WIP limits, no hero culture |
| Continuous improvement | Retrospectives, experiments |

### Adapting to Your Team

```markdown
### Our Team's Agile Practices

### What We Keep
- Daily standups (async option available)
- 2-week sprints
- Sprint planning and review
- Retrospectives

### What We Skip
- Velocity tracking (focus on outcomes)
- Story points (use t-shirt sizes)
- Burndown charts (trust over metrics)

### What We Added
- Tech debt budget (20%)
- Weekly architecture sync
- Monthly innovation day
```

## Chapter 2: Sprint Planning

You pull the next sprint into focus and cut the noise.

Marcus says, "Plan for reality, not perfection."

### Pre-Planning

```markdown
### Before Sprint Planning

**PM/Design Prep:**
- [ ] User stories written and prioritized
- [ ] Designs ready for top items
- [ ] Dependencies identified

**Tech Lead Prep:**
- [ ] Technical feasibility reviewed
- [ ] Spikes needed identified
- [ ] Tech debt items ready
```

### Planning Session

```markdown
### Sprint Planning Agenda (2 hours)

### Part 1: What (45 min)
1. Sprint goal discussion (10 min)
2. Walk through top backlog items (30 min)
3. Clarifying questions (5 min)

### Break (10 min)

### Part 2: How (45 min)
1. Task breakdown for each story
2. Identify dependencies
3. Flag risks

### Part 3: Commitment (20 min)
1. Capacity check
2. Final sprint scope
3. Confirm sprint goal
```

### Estimation

```markdown
### T-Shirt Sizing Guide

**XS**: Few hours, single file, well-understood
  Example: Fix typo, update copy, add logging

**S**: Half day to 1 day, few files, clear path
  Example: Add form field, simple API integration

**M**: 2-3 days, multiple components, some unknowns
  Example: New feature section, refactor component

**L**: 4-5 days, cross-cutting, significant unknowns
  Example: New page with data, auth flow

**XL**: More than a week, should be broken down
  Action: Split into smaller stories
```

## Chapter 3: Standups

You set the standup tone for the week.

Sarah says, "Standups are for blockers, not status theater."

### Async Standups

```markdown
### Daily Update Template (Slack/written)

**Yesterday**: What I completed
**Today**: What I'm working on
**Blockers**: Anything slowing me down

Example:
> ✅ Finished user profile API integration
> 🔨 Starting on profile edit form
> 🚧 Waiting on design review for avatar upload
```

### Sync Standups (When Needed)

- Keep to 15 minutes max
- Focus on blockers, not status
- Park discussions for after
- Not everyone needs to speak daily

## Chapter 4: Retrospectives

You pick a retro format that matches the team's energy.

Marcus says, "Retros are how you buy your time back."

### Retrospective Formats

**Start/Stop/Continue**
```markdown
### Start
- More pair programming
- Earlier design involvement

### Stop
- Late-breaking requirements
- Skipping code review

### Continue
- Friday demos
- Tech debt allocation
```

**4Ls: Liked, Learned, Lacked, Longed For**
```markdown
### Liked
- Team collaboration on the migration

### Learned
- Server Components are trickier than expected

### Lacked
- Clear acceptance criteria upfront

### Longed For
- More buffer for unexpected issues
```

### Action Items

```markdown
### Retro Action Items

| Action | Owner | Due | Status |
|--------|-------|-----|--------|
| Set up pair programming schedule | @alice | Next sprint | ✅ |
| Create PR template with checklist | @bob | This week | 🔄 |
| Schedule design sync earlier | @carol | Ongoing | 📋 |

**Rule**: Max 3 action items, must have owner and deadline
```

## Chapter 5: Definition of Done

You define what "done" means before the sprint starts.

Sarah says, "Done means shippable, not just merged."

```markdown
### Definition of Done

### Code
- [ ] Implements acceptance criteria
- [ ] Follows coding standards
- [ ] No linting errors
- [ ] Self-reviewed before PR

### Testing
- [ ] Unit tests for new logic
- [ ] Integration tests for flows
- [ ] Manual testing completed
- [ ] Edge cases covered

### Review
- [ ] Code review approved
- [ ] Design review (if UI)
- [ ] No unresolved comments

### Documentation
- [ ] README updated if needed
- [ ] API docs updated if needed
- [ ] Changelog entry added

### Deployment
- [ ] Merged to main
- [ ] Deployed to staging
- [ ] Smoke tested
```

## Chapter 6: Team Rhythms

You set a rhythm the team can sustain.

Marcus says, "Rhythm makes teams sustainable."

### Weekly Schedule

```
Monday:    Sprint planning (every other), Week kickoff
Tuesday:   Focused work, Office hours for questions
Wednesday: Mid-week sync, Pair programming
Thursday:  Focused work, Architecture discussions
Friday:    Sprint review, Retro, Learning time
```

### Meeting Hygiene

| Practice | Why |
|----------|-----|
| Default 25/50 min | Buffer between meetings |
| Agendas required | Know if you need to attend |
| Start on time | Respect everyone's time |
| End with actions | Clear next steps |
| Optional attendees | People can skip |

---

## Common Mistakes

1. **Process without purpose** - Rituals that do not change outcomes waste time.
2. **Over-planning** - Detailed plans collapse when reality shifts.
3. **No action items** - Retros become venting instead of improvement.
4. **Undefined done** - Work quality varies without clear criteria.

## Practice Exercises

1. Design a sprint planning agenda for your team
2. Create a Definition of Done tailored to your project
3. Run a retrospective using a new format

### Solutions

<details>
<summary>Exercise 1: Sprint Planning Agenda</summary>

```markdown
1. Review sprint goal (10 min)
2. Capacity check (10 min)
3. Backlog review and sizing (60 min)
4. Commit to sprint scope (20 min)
5. Risks and dependencies (10 min)
```

**Key points:**
- Keep focus on goals and capacity.
- Reserve time for risks.
- End with an explicit commitment.

</details>

<details>
<summary>Exercise 2: Definition of Done</summary>

```markdown
- Code merged with tests passing
- Lint/type checks green
- Feature documented
- UX reviewed
- Staging smoke test complete
```

**Key points:**
- Make quality gates visible.
- Keep it short and enforceable.
- Review quarterly.

</details>

<details>
<summary>Exercise 3: Retro Format</summary>

```markdown
Start: What should we start?
Stop: What should we stop?
Continue: What should we continue?
```

**Key points:**
- Use a simple format for clarity.
- Capture action items with owners.
- Review follow-ups next retro.

</details>

---

## What You Learned

This module covered:

- **Agile practices**: Adapted to real team needs
- **Planning**: Capacity-aware commitments
- **Standups**: Short updates that unblock work
- **Retros**: Continuous improvement with actions
- **Rhythms**: Sustainable meeting cadence

**Key takeaway**: Processes exist to protect focus and delivery, not to add ceremony.

---

## Real-World Application

This week at work, you might use these concepts to:

- Redesign sprint planning for clarity
- Publish a new Definition of Done
- Run a retro with action item tracking
- Adjust meeting cadence to protect focus time

---
## Further Reading

- [Shape Up](https://basecamp.com/shapeup) (alternative to Scrum)
- [Agile Manifesto](https://agilemanifesto.org/)

---

**Navigation**: [← Previous Module](./02-architecture-ownership.md) | [Next Module →](./04-hiring-onboarding.md)
