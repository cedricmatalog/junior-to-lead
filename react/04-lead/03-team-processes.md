# Team Processes

Build processes that help teams ship quality software efficiently.

## Learning Objectives

By the end of this module, you will:
- Adapt agile practices to your team
- Run effective planning and estimation
- Facilitate productive retrospectives
- Create sustainable team rhythms

## Agile in Practice

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
## Our Team's Agile Practices

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

## Sprint Planning

### Pre-Planning

```markdown
## Before Sprint Planning

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
## Sprint Planning Agenda (2 hours)

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
## T-Shirt Sizing Guide

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

## Standups

### Async Standups

```markdown
## Daily Update Template (Slack/written)

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

## Retrospectives

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
## Retro Action Items

| Action | Owner | Due | Status |
|--------|-------|-----|--------|
| Set up pair programming schedule | @alice | Next sprint | ✅ |
| Create PR template with checklist | @bob | This week | 🔄 |
| Schedule design sync earlier | @carol | Ongoing | 📋 |

**Rule**: Max 3 action items, must have owner and deadline
```

## Definition of Done

```markdown
## Definition of Done

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

## Team Rhythms

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

## Practice Exercises

1. Design a sprint planning agenda for your team
2. Create a Definition of Done tailored to your project
3. Run a retrospective using a new format

## Further Reading

- [Shape Up](https://basecamp.com/shapeup) (alternative to Scrum)
- [Agile Manifesto](https://agilemanifesto.org/)
