# Incident Management

Handle production issues calmly and learn from them.

## Learning Objectives

By the end of this module, you will:
- Respond to incidents effectively
- Lead postmortems that drive improvement
- Build runbooks and reduce MTTR
- Create a blameless incident culture

## Incident Response

### Severity Levels

| Level | Definition | Response Time | Examples |
|-------|------------|---------------|----------|
| SEV1 | Service down, major revenue impact | 15 min | Site completely down |
| SEV2 | Major feature broken, workaround exists | 1 hour | Checkout failing |
| SEV3 | Minor feature broken, limited impact | 4 hours | Filter not working |
| SEV4 | Low priority issue | Next sprint | Minor UI bug |

### Incident Response Process

```markdown
## Incident Response Flow

### 1. Detection (0-5 min)
- Alert fires or user reports
- Acknowledge in on-call tool
- Initial assessment

### 2. Triage (5-15 min)
- Determine severity
- Decide if escalation needed
- Notify stakeholders

### 3. Response (Ongoing)
- Incident Commander assigned
- Create incident channel
- Begin investigation

### 4. Resolution
- Implement fix
- Verify fix works
- Monitor for recurrence

### 5. Post-Incident
- Incident report
- Schedule postmortem
- Track action items
```

### Incident Commander Role

```markdown
## Incident Commander Responsibilities

**Do:**
- Coordinate response efforts
- Communicate status updates
- Make decisions on actions
- Keep track of timeline
- Delegate tasks clearly

**Don't:**
- Debug code yourself
- Get lost in details
- Forget to update stakeholders
- Make changes without communication

## Communication Template

"[TIME] [SEV-X] [Status Update]

**Current Status**: Brief description
**Impact**: Who/what is affected
**Actions**: What we're doing
**Next Update**: In X minutes

Example:
14:32 SEV1 Update #3
Current Status: Identified database connection issue
Impact: Checkout flow failing for all users
Actions: Ops team restarting connection pool
Next Update: In 15 minutes"
```

## On-Call Best Practices

### On-Call Setup

```markdown
## On-Call Rotation

### Schedule
- Primary: 1 week rotation
- Secondary: Shadows primary, takes over if needed
- Escalation: Team lead, then engineering manager

### Expectations
- Acknowledge alerts within 15 min (SEV1-2)
- Available during on-call hours (defined per team)
- Handoff at rotation end

### Compensation
- On-call stipend: $X per week
- Overtime for incident response
- Comp time for major incidents

### Tools
- PagerDuty for alerting
- Slack #incidents channel
- Runbooks in Notion
```

### Alert Quality

```markdown
## Good vs Bad Alerts

### Good Alert
**Title**: High error rate on /api/checkout
**Context**: Error rate 5% (threshold 1%)
**Runbook**: link-to-runbook
**Dashboard**: link-to-dashboard

### Bad Alert
**Title**: Error
**Context**: Something is wrong
**Runbook**: None
**Dashboard**: None

## Alert Hygiene
- Review alerts monthly
- Delete or fix noisy alerts
- Every alert should be actionable
- If you regularly ignore an alert, fix or remove it
```

## Postmortems

### Postmortem Template

```markdown
# Incident Postmortem: [Title]

## Summary
**Date**: 2024-01-15
**Duration**: 2 hours 15 minutes
**Severity**: SEV1
**Author**: @tech-lead

## Impact
- Checkout unavailable for 2 hours
- ~500 failed transactions
- Estimated revenue impact: $X

## Timeline (All times UTC)
| Time | Event |
|------|-------|
| 14:00 | Deploy of PR #1234 |
| 14:15 | First error alerts |
| 14:18 | On-call acknowledges |
| 14:25 | SEV1 declared, IC assigned |
| 14:45 | Root cause identified |
| 15:10 | Fix deployed |
| 15:30 | Monitoring confirms resolution |
| 16:15 | Incident closed |

## Root Cause
The deployment included a change to the payment API client that
used a new field not yet available in production. This caused a
null pointer exception on every checkout attempt.

## Contributing Factors
1. Change wasn't tested against production API
2. Staging environment had different data
3. No canary deployment for this service

## What Went Well
- Fast detection (15 min)
- Clear incident communication
- Quick rollback decision

## What Could Be Improved
- Testing with production-like data
- Canary deployment would have limited blast radius
- Better error messages would have sped up debugging

## Action Items
| Action | Owner | Priority | Status |
|--------|-------|----------|--------|
| Add integration test with prod API | @alice | P1 | Open |
| Implement canary deployments | @bob | P1 | Open |
| Improve error logging in payment client | @carol | P2 | Open |
| Update runbook with this scenario | @tech-lead | P2 | Complete |

## Lessons Learned
- Production parity in staging is critical
- Canary deployments should be standard for payment flows
- Error messages should include context about what field/value failed
```

### Running a Postmortem Meeting

```markdown
## Postmortem Meeting Agenda (60 min)

### Setup (5 min)
- State purpose: Learning, not blame
- Review timeline together

### What Happened (15 min)
- Walk through events
- Fill in gaps
- Correct inaccuracies

### Root Cause Analysis (15 min)
- Use 5 Whys technique
- Identify contributing factors
- Look for systemic issues

### What We'll Change (20 min)
- Brainstorm improvements
- Prioritize actions
- Assign owners

### Wrap-up (5 min)
- Review action items
- Schedule follow-ups
- Thank participants
```

### 5 Whys Example

```markdown
**Problem**: Checkout was down for 2 hours

**Why?** The payment API returned null for a required field
**Why?** We deployed code that expected a new field
**Why?** The field wasn't in production API yet
**Why?** Staging had mock data with the field
**Why?** We don't test against production-like data

**Root Cause**: Staging environment doesn't match production

**Action**: Implement production API integration tests
```

## Runbooks

### Runbook Template

```markdown
# Runbook: High Error Rate on Checkout

## Overview
This runbook covers how to respond to elevated error rates
on the checkout endpoint.

## Detection
- Alert: "Checkout error rate > 1%"
- Dashboard: [link]

## Symptoms
- Users unable to complete purchases
- Error rate spike in monitoring
- Customer complaints

## Diagnosis Steps

### 1. Check Recent Deployments
```bash
git log --oneline --since="2 hours ago"
```
If recent deploy, consider rollback.

### 2. Check External Dependencies
- Payment provider status: [link]
- Database status: [link]

### 3. Check Application Logs
```bash
kubectl logs -l app=checkout --since=1h | grep ERROR
```

## Resolution Steps

### If Recent Deploy
1. Notify team in #incidents
2. Rollback: `deploy rollback checkout`
3. Monitor for 15 minutes

### If External Dependency
1. Check provider status page
2. Enable fallback if available
3. Communicate to customers

### If Database Issue
1. Check connection pool
2. Restart if needed: `kubectl rollout restart`
3. Escalate to database team

## Escalation
- Primary: On-call frontend
- Secondary: @tech-lead
- Database: @dba-team
```

## Practice Exercises

1. Write a runbook for a common incident in your system
2. Practice incident commander role in a tabletop exercise
3. Conduct a postmortem for a past incident

## Further Reading

- [Google SRE Book - Incident Management](https://sre.google/sre-book/managing-incidents/)
- [Blameless Postmortems](https://www.etsy.com/codeascraft/blameless-postmortems/)
