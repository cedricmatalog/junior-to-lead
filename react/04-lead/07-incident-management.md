# Incident Management

> **Last reviewed**: 2026-01-06


## Week 38: The Fire Drill

A checkout outage hits during a product launch. Sarah asks you to run the incident response, and Marcus reminds you that calm leadership matters as much as the fix. You need to triage quickly, communicate clearly, and turn chaos into learning. This week focuses on severity levels, incident command, postmortems, and the runbooks that prevent repeat failures.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Severity levels** | Triage tags - decide how fast to move |
| **Incident command** | Air traffic control - coordinate without flying every plane |
| **Runbooks** | Checklists - reduce decision load under stress |
| **Postmortems** | Flight recorders - learn what happened and why |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 37 (Technical Strategy) - Comfort making trade-offs and communicating impact.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Classify incidents by severity and response time
- [ ] Lead incident response as the incident commander
- [ ] Communicate updates to stakeholders under pressure
- [ ] Run blameless postmortems that produce action
- [ ] Create runbooks that reduce time to resolution
- [ ] Improve alert quality and on-call readiness

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice incident leadership over 8-12 weeks

---

## Chapter 1: Prepare Before the Incident

Sarah wants on-call and alerting in place before the next outage. Marcus reminds you that bad alerts create bad decisions.

### On-Call Essentials

```markdown
## On-Call Rotation

- Primary: 1 week rotation
- Secondary: Shadows primary, takes over if needed
- Escalation: Team lead, then engineering manager

## Expectations
- Acknowledge alerts within 15 minutes for SEV1-2
- Keep handoff notes at rotation end
- Document fixes in runbooks
```

### Alert Quality

```markdown
## Good Alert
Title: High error rate on /api/checkout
Context: Error rate 5% (threshold 1%)
Runbook: link-to-runbook
Dashboard: link-to-dashboard

## Bad Alert
Title: Error
Context: Something is wrong
Runbook: None
Dashboard: None
```

## Chapter 2: Declare Severity and Triage

Your first move is to set severity and assemble the right response. Overreacting wastes time, underreacting amplifies damage.

| Level | Definition | Response Time | Example |
|-------|------------|---------------|---------|
| SEV1 | Service down, major revenue impact | 15 minutes | Checkout unavailable |
| SEV2 | Major feature broken, workaround exists | 1 hour | Payment retries failing |
| SEV3 | Minor feature broken, limited impact | 4 hours | Filters not working |
| SEV4 | Low priority issue | Next sprint | Minor UI bug |

## Chapter 3: Run the Incident

Sarah assigns you as incident commander. Your job is coordination and communication, not deep debugging.

```markdown
## Incident Commander Responsibilities

Do:
- Coordinate response efforts
- Communicate status updates
- Make decisions on actions
- Track the timeline
- Delegate clearly

Do not:
- Debug code yourself
- Get lost in details
- Make changes without communication
```

```markdown
## Status Update Template

[TIME] [SEV-X] Update #N

Current Status: Brief description
Impact: Who or what is affected
Actions: What is happening right now
Next Update: In X minutes
```

## Chapter 4: Resolve and Verify

Marcus reminds you that the incident is not over when the fix is deployed. You need to confirm recovery and watch for regressions.

```markdown
## Resolution Checklist

- Fix deployed or rollback complete
- Key metrics back to normal
- Logs show error rate stabilized
- Stakeholders notified of recovery
```

## Chapter 5: Learn and Improve

Sarah wants a blameless postmortem that drives change. The goal is to prevent the next incident, not assign blame.

```markdown
# Postmortem Template

## Summary
Date, duration, severity, owner

## Impact
What broke, who was affected, business impact

## Timeline
What happened, in order

## Root Cause
The technical root and the contributing factors

## What Went Well
Fast detection, good communication, etc.

## What Needs Improvement
Gaps in testing, monitoring, or process

## Action Items
| Action | Owner | Priority | Status |
|--------|-------|----------|--------|
```

## Chapter 6: Write Runbooks That Work

Marcus suggests short, action-first runbooks that are easy to scan during a crisis.

```markdown
# Runbook: High Error Rate on Checkout

## Detection
- Alert: Checkout error rate > 1%
- Dashboard: link-to-dashboard

## Diagnosis Steps
1. Check recent deploys
2. Check external dependencies
3. Check logs for error patterns

## Resolution Steps
- If recent deploy, rollback
- If dependency issue, enable fallback
- If database issue, restart connection pool

## Escalation
- Primary: On-call frontend
- Secondary: Tech lead
- Database: DBA team
```

---

## Common Mistakes

1. **No clear incident commander** - Work happens, but coordination fails.
2. **Silent incident updates** - Stakeholders lose trust and invent their own narrative.
3. **Fixing without verification** - Incidents resurface because recovery was not checked.
4. **Postmortems without follow-through** - Action items never land and the same issues repeat.

## Practice Exercises

1. Write a runbook for a common incident in your system
2. Draft a status update for a SEV1 outage
3. Outline a postmortem for a recent issue

### Solutions

<details>
<summary>Exercise 1: Runbook</summary>

```markdown
# Runbook: Login Error Spike

Detection: Alert on 5xx rate > 2%
Diagnosis: Check recent auth deploys, verify identity provider status
Resolution: Roll back latest change or enable fallback auth
Escalation: On-call, then security team
```

**Key points:**
- Start with detection and diagnosis.
- Keep steps short and actionable.
- Include escalation paths.

</details>

<details>
<summary>Exercise 2: Status Update</summary>

```markdown
15:20 SEV1 Update #2
Current Status: Checkout API returning 500s
Impact: All purchases failing
Actions: Rollback in progress, monitoring error rate
Next Update: In 15 minutes
```

**Key points:**
- Use consistent format.
- Keep impact explicit.
- Always provide next update time.

</details>

<details>
<summary>Exercise 3: Postmortem Outline</summary>

```markdown
Summary: 45-minute checkout outage after deploy
Impact: 420 failed orders, revenue loss
Root Cause: Missing null check in payment adapter
Action Items: Add integration test, add feature flag rollback, update runbook
```

**Key points:**
- Focus on learning, not blame.
- Use clear action items.
- Assign owners and timelines.

</details>

---

## What You Learned

This module covered:

- **Incident triage**: Classify severity to set response speed
- **Incident command**: Coordinate actions and communication
- **Postmortems**: Turn incidents into improvements
- **Runbooks**: Reduce time to resolution with clear steps
- **Alert quality**: Make alerts actionable and trustworthy

**Key takeaway**: Calm, structured response shortens outages and builds trust.

---

## Real-World Application

This week at work, you might use these concepts to:

- Audit noisy alerts and improve signal quality
- Create a runbook for your top incident type
- Host a tabletop incident exercise
- Improve incident updates for stakeholders
- Track postmortem action items to completion

---

## Further Reading

- [Google SRE Book - Managing Incidents](https://sre.google/sre-book/managing-incidents/)
- [Blameless Postmortems](https://www.etsy.com/codeascraft/blameless-postmortems/)

---

**Navigation**: [<- Previous Module](./06-technical-strategy.md) | [Next Module ->](./08-documentation-culture.md)