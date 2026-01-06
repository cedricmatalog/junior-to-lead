# Code Review Mastery

> **Last reviewed**: 2026-01-06


## Week 28: The Review Bottleneck

Pull requests are piling up and quality is slipping. Sarah wants faster review cycles without sacrificing rigor. Marcus says the best reviewers do more than find bugs -- they teach, align, and protect the architecture. This week is about mastering code reviews: how to evaluate quality, communicate clearly, and build a culture where feedback improves the whole team.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Code review** | A safety net - catch issues before they ship |
| **Feedback** | A compass - point toward a better direction |
| **Architecture review** | A blueprint check - ensure structure holds |
| **Review culture** | A team sport - everyone improves together |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 27 (Observability and Monitoring) - Familiarity with production risks and trade-offs.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Give constructive, actionable feedback
- [ ] Review correctness, design, and maintainability
- [ ] Conduct architectural reviews for major changes
- [ ] Balance thoroughness with delivery speed
- [ ] Receive feedback without defensiveness
- [ ] Build a sustainable review culture

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice review leadership over 6-8 weeks

---

## Chapter 1: The Purpose of Code Review

Before reviewing code, align on why the review exists.

1. **Catch bugs** before they reach production
2. **Share knowledge** across the team
3. **Maintain standards** for code quality
4. **Mentor** team members
5. **Validate architecture** decisions

## Chapter 2: What to Review

A good review checks correctness, design, and risk.

### Correctness
- Does it solve the problem?
- Edge cases handled?
- Error handling complete?

### Design
- Right abstraction level?
- Follows existing patterns?
- Will it scale?

### Maintainability
- Readable without comments?
- Well-named variables/functions?
- Reasonable complexity?

### Security
- Input validation?
- XSS prevention?
- Sensitive data handled properly?

### Performance
- Unnecessary re-renders?
- Large bundle impact?
- Expensive operations?

## Chapter 3: Providing Feedback

Feedback should be clear, actionable, and respectful.

### Be Specific and Actionable

```markdown
❌ Bad: "This is confusing"
✅ Good: "Consider extracting the date formatting logic into a
         `formatDate` utility function for reusability and testing"

❌ Bad: "Fix the naming"
✅ Good: "Rename `data` to `userProfiles` to clarify what this
         array contains"
```

### Use Suggestions, Not Demands

```markdown
❌ "You need to add error handling here"
✅ "What happens if the API call fails? Consider adding a try/catch
    and displaying an error message to the user"

❌ "This is wrong"
✅ "I think this might cause issues when X happens. What do you think
    about handling it this way?"
```

### Explain the Why

```markdown
❌ "Don't use useEffect for this"
✅ "This derived value can be calculated during render instead of
    useEffect. useEffect should be for syncing with external systems.
    See: https://react.dev/learn/you-might-not-need-an-effect"
```

### Categorize Comments

```markdown
[nit]: Consider using `const` instead of `let` here since it's never reassigned

[suggestion]: This could be simplified with optional chaining: `user?.profile?.name`

[question]: Why are we fetching this data here instead of in the parent?

[blocking]: This will cause a security vulnerability - user input isn't sanitized
```

## Chapter 4: Review Workflow

The process matters just as much as the comments.

### Before Reviewing

1. Understand the context (read ticket/description)
2. Check if tests pass
3. Look at the overall scope

### During Review

```markdown
### Review Summary

**What I checked:**
- Component logic and state management
- Error handling
- Accessibility

**What looks good:**
- Clean separation of concerns
- Good TypeScript types
- Comprehensive tests

**Suggestions:**
1. [Line 45] Consider memoizing this callback
2. [Line 78] Add loading state for better UX

**Questions:**
- How will this perform with 1000+ items?
```

### Levels of Review

| Type | Focus | Time |
|------|-------|------|
| Quick | Typos, formatting, obvious issues | 5-10 min |
| Standard | Logic, patterns, edge cases | 15-30 min |
| Deep | Architecture, security, performance | 30-60 min |

## Chapter 5: Architectural Reviews

Architecture reviews prevent long-term pain.

For large changes, review the approach before implementation.

```markdown
### Architecture Review: Feature X

### Changes Proposed
- New state management approach
- API endpoint changes
- New shared components

### Questions to Answer
1. Does this align with our technical direction?
2. Will this scale with expected growth?
3. How does this affect existing code?
4. What are the migration risks?

### Decision
Approved with conditions:
- Add ADR documenting the decision
- Include migration plan for existing features
```

## Chapter 6: Receiving Feedback

How you receive feedback shapes the team culture.

### As the Author

1. **Don't take it personally** - Review is about code, not you
2. **Assume good intent** - Reviewers want to help
3. **Ask for clarification** - If you don't understand
4. **Push back respectfully** - With reasoning

```markdown
> Consider using Redux for this state

Thanks for the suggestion! I considered Redux, but since this state
is only used in this component tree, I think Context is simpler and
sufficient. What do you think?
```

### Handling Disagreements

1. Discuss, don't argue
2. Focus on trade-offs, not opinions
3. Propose experiments if needed
4. Escalate only if blocking

## Chapter 7: Building Review Culture

Culture determines whether reviews help or hurt.

### As a Team

- Agree on review turnaround time (e.g., 24 hours)
- Define what's blocking vs suggestion
- Rotate reviewers for knowledge sharing
- Celebrate good reviews

### Review Agreements

```markdown
### Our Review Standards

1. **Response time**: First review within 24 hours
2. **Size**: PRs should be < 400 lines
3. **Blocking**: Security, correctness, breaking changes
4. **Non-blocking**: Style preferences, minor improvements
5. **Required approvals**: 1 for standard, 2 for infrastructure
```

### Common Review Patterns

### The Rubber Stamp
Problem: Approving without reading
Fix: Require substantive comments

### The Nitpick Marathon
Problem: Too many minor comments
Fix: Use linters, batch nits together

### The Stalemate
Problem: PR sits for days
Fix: Set SLAs, escalation path

---

## Common Mistakes

1. **Reviewing too late** - Slow feedback blocks delivery.
2. **Focusing on style** - Linters can handle most formatting issues.
3. **Missing architecture risk** - Big changes need big-picture checks.
4. **Unclear feedback** - Vague comments waste time.

## Practice Exercises

1. Review 3 PRs using the categorization system
2. Write architectural review for a major feature
3. Create review guidelines for your team

### Solutions

<details>
<summary>Exercise 1: Review Categorization</summary>

```text
PR #1: Correctness (blocking) - missing error handling
PR #2: Maintainability (suggestion) - rename variable for clarity
PR #3: Performance (blocking) - N+1 query pattern
```

**Key points:**
- Separate blocking vs suggestion.
- Attach a reason to each category.
- Keep feedback concise and actionable.

</details>

<details>
<summary>Exercise 2: Architecture Review</summary>

```markdown
## Architecture Review: Feature X

### Scope
New search pipeline with caching and personalization.

### Constraints
- Must support multi-tenant data separation
- SLO: P95 < 300ms

### Risks
- Shared cache keys across tenants
- Tight coupling to user profile service

### Recommendation
Introduce cache namespaces and a service abstraction layer.
```

**Key points:**
- Identify risks early.
- Provide a concrete alternative.
- Keep scope focused.

</details>

<details>
<summary>Exercise 3: Review Guidelines</summary>

```markdown
### Review Guidelines
1. First review within 24 hours
2. Blockers must include reasoning
3. Suggestions labeled as non-blocking
4. Large PRs require architecture review
5. Tests required for new behavior
```

**Key points:**
- Set expectations for response time.
- Separate blocking from optional feedback.
- Encourage smaller PRs.

</details>

---

## What You Learned

This module covered:

- **Purpose**: Reviews protect quality and share knowledge
- **Focus areas**: Correctness, design, maintainability, security
- **Feedback**: Clear, actionable communication
- **Workflow**: Efficient review cycles
- **Culture**: Standards that help the team grow

**Key takeaway**: Great reviews improve both code and people.

---

## Real-World Application

This week at work, you might use these concepts to:

- Create a review checklist for your team
- Categorize feedback as blocking vs non-blocking
- Lead an architecture review for a complex change
- Reduce PR cycle time with review SLAs
- Coach a teammate on review quality

---

## Further Reading

- [Google's Code Review Guidelines](https://google.github.io/eng-practices/review/)
- [How to Make Your Code Reviewer Fall in Love with You](https://mtlynch.io/code-review-love/)

---

**Navigation**: [<- Previous Module](./07-observability-monitoring.md) | [Next Module ->](./09-mentoring-juniors.md)