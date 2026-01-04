# Code Review Mastery

Provide reviews that improve code, share knowledge, and build team culture.

## Learning Objectives

By the end of this module, you will:
- Give constructive, actionable feedback
- Conduct architectural reviews
- Balance thoroughness with speed
- Build a review culture that helps everyone grow

## The Purpose of Code Review

1. **Catch bugs** before they reach production
2. **Share knowledge** across the team
3. **Maintain standards** for code quality
4. **Mentor** team members
5. **Validate architecture** decisions

## What to Review

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

## Providing Feedback

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

## Review Workflow

### Before Reviewing

1. Understand the context (read ticket/description)
2. Check if tests pass
3. Look at the overall scope

### During Review

```markdown
## Review Summary

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

## Architectural Reviews

For large changes, review the approach before implementation.

```markdown
## Architecture Review: Feature X

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

## Receiving Feedback

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

## Building Review Culture

### As a Team

- Agree on review turnaround time (e.g., 24 hours)
- Define what's blocking vs suggestion
- Rotate reviewers for knowledge sharing
- Celebrate good reviews

### Review Agreements

```markdown
## Our Review Standards

1. **Response time**: First review within 24 hours
2. **Size**: PRs should be < 400 lines
3. **Blocking**: Security, correctness, breaking changes
4. **Non-blocking**: Style preferences, minor improvements
5. **Required approvals**: 1 for standard, 2 for infrastructure
```

## Common Review Patterns

### The Rubber Stamp
Problem: Approving without reading
Fix: Require substantive comments

### The Nitpick Marathon
Problem: Too many minor comments
Fix: Use linters, batch nits together

### The Stalemate
Problem: PR sits for days
Fix: Set SLAs, escalation path

## Practice Exercises

1. Review 3 PRs using the categorization system
2. Write architectural review for a major feature
3. Create review guidelines for your team

## Further Reading

- [Google's Code Review Guidelines](https://google.github.io/eng-practices/review/)
- [How to Make Your Code Reviewer Fall in Love with You](https://mtlynch.io/code-review-love/)
