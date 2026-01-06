# Documentation Culture

> **Last reviewed**: 2026-01-06


## Week Eight: The Knowledge Gap

New hires keep asking the same questions, and key decisions live only in chat. Sarah asks you to build a documentation culture that scales with the team. Marcus reminds you that good docs are not optional; they are how a team keeps moving when people change. This week focuses on doc types, README and ADR standards, ownership, and habits that keep knowledge current.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Documentation** | Shared memory - what the team should not forget |
| **README** | A front door - orient people fast |
| **ADRs** | A decision trail - why you chose this path |
| **Docs-as-code** | A pipeline - docs ship like software |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Lead Module 05 (Stakeholder Communication) - Comfort documenting decisions and sharing updates.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Build a documentation map for your team
- [ ] Standardize README and ADR formats
- [ ] Define ownership and update cadence
- [ ] Create docs that are easy to discover and use
- [ ] Embed documentation habits into team workflows
- [ ] Treat docs as part of the delivery pipeline

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice documentation habits over 8-12 weeks

---

## Chapter 1: Make the Cost Visible

You quantify the time cost so the team feels it.

Sarah wants the team to see documentation as a time saver, not a burden. Marcus suggests quantifying the cost of missing docs.

Sarah says, "Every missing doc shows up as a meeting."

```markdown
## Missing Documentation Impact

Without documentation:
- New engineer asks 5 questions per day
- Each question takes 15 minutes from someone
- 5 engineers x 15 minutes x 5 questions = 6+ hours per day
- Onboarding takes 3 months

With documentation:
- New engineer finds 4 of 5 answers in docs
- 1 question per day, still captured in docs
- Onboarding takes 1 month
- Knowledge persists when people leave
```

## Chapter 2: Map the Doc Types

You map questions to the right doc type so people can self-serve.

Different questions require different doc types. Keep the map simple and visible.

Marcus says, "Know which docs matter most."

```markdown
## Documentation Types

Reference
- Purpose: Look up specific information
- Examples: API docs, configuration options
- Update trigger: Code changes

Tutorial
- Purpose: Learn how to do something
- Examples: Getting started, how-to guides
- Update trigger: Quarterly review

Explanation
- Purpose: Understand why or how something works
- Examples: Architecture docs, ADRs
- Update trigger: Design changes

Runbook
- Purpose: Handle specific situations
- Examples: Incident response, deployment
- Update trigger: After incidents
```

## Chapter 3: Standardize READMEs and ADRs

You make entry points predictable so people stop guessing.

Sarah wants consistent entry points so the team always knows where to look.

Sarah says, "Standards reduce guesswork."

### Project README Template

```markdown
# Project Name

Brief description of what this project does.

## Quick Start

    npm install
    npm run dev

## Prerequisites
- Node.js 18+
- Access to internal services
- Environment variables (see .env.example)

## Commands
| Command | Description |
|---------|-------------|
| npm run dev | Start development server |
| npm run build | Build for production |
| npm run test | Run tests |
| npm run lint | Check code style |

## Architecture
Brief overview and links to deeper docs.

## Team
Owner, Slack channel, on-call rotation
```

### ADR Template

```markdown
# ADR: [Decision Title]

## Context
What problem are we solving?

## Decision
What was chosen and why?

## Alternatives
What else did we consider?

## Consequences
What changes because of this decision?
```

## Chapter 4: Keep Docs Updated

You assign ownership so docs do not decay.

Marcus wants ownership and a schedule so docs do not decay.

Marcus says, "A stale doc is a broken promise."

```markdown
## Documentation Maintenance

Triggers for updates:
- Code changes that affect documented behavior
- Onboarding questions not answered by docs
- Post-incident action items
- Quarterly review cycle

Ownership:
- Each doc has an owner
- Owners review quarterly
- Team can propose changes via PR
```

## Chapter 5: Build a Documentation Culture

You make documentation part of the workflow, not an extra task.

Sarah asks you to make documentation part of the workflow, not an afterthought.

Sarah says, "Docs are a team habit, not an afterthought."

```markdown
## Documentation Practices

In pull requests:
- README updated if behavior changes
- ADR for architectural decisions
- Inline comments for complex logic

In onboarding:
- First task is to fix something in docs
- Capture questions as doc updates

Recognition:
- Celebrate great docs
- Highlight doc contributions in reviews
```

## Chapter 6: Docs as Code

You treat docs like code so they stay healthy.

Marcus suggests treating docs like software so they stay current and testable.

Marcus says, "Docs deserve the same rigor as code."

```markdown
## Docs-as-Code Practices

- Version control (git)
- Review process (PRs)
- Automated checks (links, spelling)
- CI/CD for docs publishing
```

---

## Common Mistakes

1. **Docs with no owner** - Nobody feels responsible for updates.
2. **One giant wiki page** - Information becomes hard to find and maintain.
3. **Docs that never ship** - Drafts are not useful if they stay hidden.
4. **Outdated examples** - Broken code examples erode trust quickly.

## Practice Exercises

1. Create a README template for your team
2. Write an ADR for a recent decision
3. Audit existing documentation for gaps

### Solutions

<details>
<summary>Exercise 1: README Template</summary>

```markdown
# Service Name

## Purpose
One paragraph on what this service does.

## Quick Start
npm install
npm run dev

## Commands
- npm run test
- npm run lint

## Ownership
Team: Frontend Platform
Slack: #frontend-platform
```

**Key points:**
- Put the essentials at the top.
- Keep it short and skimmable.
- Link to deeper docs.

</details>

<details>
<summary>Exercise 2: ADR</summary>

```markdown
# ADR: Adopt TanStack Query

Context: Data fetching is inconsistent across teams.
Decision: Standardize on TanStack Query.
Alternatives: Redux Thunks, SWR.
Consequences: Training and migration work required.
```

**Key points:**
- Capture the why, not just the what.
- List alternatives briefly.
- Note the trade-offs.

</details>

<details>
<summary>Exercise 3: Documentation Audit</summary>

```markdown
Audit Checklist
- Missing README for key services
- ADRs not indexed
- Outdated setup steps
- Broken links to runbooks
```

**Key points:**
- Start with the highest traffic docs.
- Track fixes as action items.
- Assign owners.

</details>

---

## What You Learned

This module covered:

- **Doc types**: Match the format to the question
- **Standards**: Consistent READMEs and ADRs
- **Ownership**: Clear accountability keeps docs alive
- **Culture**: Make docs part of the workflow
- **Docs-as-code**: Treat documentation like product work

**Key takeaway**: Documentation scales the team when people and priorities change.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add ownership to a key architecture doc
- Create a README for a critical service
- Add an ADR for a recent decision
- Set a quarterly doc review cadence
- Update runbooks after an incident

---

## Further Reading

- [Divio Documentation System](https://documentation.divio.com/)
- [Write the Docs](https://www.writethedocs.org/)

---

**Navigation**: [← Previous Module](./07-incident-management.md) | [Next Module →](./09-cross-team-collaboration.md)
