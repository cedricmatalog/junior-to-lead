# Mentoring Juniors

> **Last reviewed**: 2026-01-06


## Week Fourteen: The Growth Plan

Two new juniors joined the team, and Sarah asks you to mentor one of them. Marcus reminds you that mentoring is not just answering questions -- it is about building confidence and habits. This week focuses on how to mentor effectively: set goals, teach with intention, give feedback, and balance mentoring with your own delivery.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Mentoring** | A coaching plan - guide progress over time |
| **Teaching** | A scaffold - support that fades as skills grow |
| **Feedback** | A mirror - show gaps without judgment |
| **Pairing** | A tandem bike - move together at the right pace |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 13 (Code Review Mastery) - Familiarity with feedback and team collaboration.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Structure effective mentoring relationships
- [ ] Adapt teaching to different learning styles
- [ ] Create growth plans with clear milestones
- [ ] Provide feedback that accelerates growth
- [ ] Pair program without taking over
- [ ] Balance mentoring with delivery work

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice mentoring over 6-8 weeks

---

## Chapter 1: Mentoring Mindset

Mentoring works best when you focus on growth, not gatekeeping.

### Your Role

- **Guide**, not gatekeeper
- **Enabler**, not solver
- **Teacher**, not judge

### Growth Multiplier

```
Your individual output:     1x
Training one person:        1.5x (while training)
After training complete:    2x (two productive people)
Training them to train:     nx (exponential growth)
```

## Chapter 2: Structuring Mentorship

A structured plan prevents mentorship from becoming ad-hoc support.

### Initial Assessment

```markdown
## Getting to Know Your Mentee

1. **Background**
   - Previous experience?
   - Strongest skills?
   - Biggest gaps?

2. **Goals**
   - Where do they want to be in 6 months? 1 year?
   - What skills are they most excited to learn?
   - What's blocking them?

3. **Learning Style**
   - Learn by reading? Watching? Doing?
   - Prefer detailed explanation or high-level then explore?
   - How do they handle feedback?
```

### Growth Plan

```markdown
## 90-Day Growth Plan: [Name]

### Current Level
- Good: JavaScript fundamentals, React basics
- Needs work: TypeScript, testing, async patterns

### Goals
1. Write TypeScript confidently
2. Add tests to features they build
3. Own a small feature end-to-end

### Weekly Focus
- Week 1-2: TypeScript basics (pair on converting JS file)
- Week 3-4: Testing fundamentals (write tests together)
- Week 5-6: First small feature with guidance
- Week 7-8: Second feature with less guidance
- Week 9-12: Independent work with review

### Check-in Schedule
- Daily: Quick async check (Slack)
- Weekly: 30-min sync (progress, blockers)
- Monthly: 1-hour deep dive (growth reflection)
```

## Chapter 3: Teaching Techniques

Different juniors need different teaching modes.

### The "Why" Before "How"

```tsx
// Bad: Just showing the solution
"Use useMemo here"

// Good: Explain the problem first
"Notice how clicking this button re-renders the entire list?
 That's because filterItems runs on every render, even when
 items haven't changed. useMemo can cache the result so we
 only recalculate when items or filter changes. Let's add it..."
```

### Scaffolded Challenges

```markdown
### Challenge: Build a Search Component

### Level 1 (Guided)
I'll show you how to build the input and display results.
You implement the filtering logic.

### Level 2 (Hints)
Build the whole component. Hints:
- You'll need useState for the query
- Filter items based on query
- Display "No results" when empty

### Level 3 (Independent)
Add debouncing to the search (300ms delay).
Try it yourself first, then we'll review.

### Stretch
Add keyboard navigation for results.
```

### Pair Programming

```markdown
## Chapter 4: Pairing Modes

Pairing can teach quickly when you use it intentionally.

**Driver-Navigator**
- Junior drives: Types code, thinks out loud
- You navigate: Guide direction, catch errors
- Rotate periodically

**Watch and Learn**
- You drive: Explain your thought process
- Junior watches: Asks questions
- Good for: Complex problems, showing patterns

**Ping-Pong**
- Alternate: You write test, they write code
- Good for: TDD, keeping both engaged
```

## Chapter 5: Giving Feedback

Good feedback is specific and tied to impact.

### The SBI Model

**Situation**: When/where did this happen?
**Behavior**: What specifically did they do?
**Impact**: What was the result?

```markdown
"In yesterday's PR (situation), you added comprehensive error
handling for all the API calls (behavior). This made the feature
much more robust and caught an edge case we might have missed
in production (impact)."

"During the planning meeting (situation), you jumped in with
implementation details before we understood the requirements
(behavior). This meant we had to backtrack and re-discuss later
(impact). Next time, try asking clarifying questions first."
```

### Constructive Corrections

```markdown
❌ "That's wrong"
✅ "That approach would work, but here's a potential issue..."

❌ "You should know this"
✅ "This is a common gotcha. Here's why it happens..."

❌ "Let me just do it"
✅ "Let's work through this together"
```

### Celebrating Wins

```markdown
- Acknowledge progress publicly (team chat, standups)
- Be specific about what they did well
- Connect to their growth goals
- "I noticed you caught that race condition in code review -
   three months ago you wouldn't have spotted that!"
```

## Chapter 6: Unblocking Without Solving

Solve the problem of learning, not the problem itself.

### Ask Before Telling

```markdown
"What have you tried so far?"
"What do you think is causing it?"
"Where would you look for more information?"
"What's your hypothesis?"
```

### Guide to the Answer

```markdown
"Interesting. What does the error message say?"
"Can you console.log the value at this point?"
"Have you checked if the data is arriving?"
"What would happen if you removed this line?"
```

### Know When to Just Tell

- They've been stuck for hours
- Deadline pressure
- Obscure issue (dependency bug, etc.)

## Chapter 7: Balancing Mentoring with Work

Mentoring is part of the job, but delivery still matters.

### Time Boxing

- Set dedicated mentoring hours
- Group questions into sessions
- Async feedback when possible

### Leverage Reviews

- Use PR reviews as teaching moments
- Link to docs/resources
- Explain patterns as you review

### Create Resources

```markdown
Repeated questions → Write a doc
Common mistakes → Create a lint rule
Frequent patterns → Build a snippet library
```

---

## Common Mistakes

1. **Solving everything** - Juniors learn less when you take the wheel.
2. **No goals** - Mentorship without milestones drifts.
3. **Inconsistent feedback** - Mixed signals slow growth.
4. **Overcommitting** - Mentoring still needs boundaries.

## Practice Exercises

1. Create a 90-day growth plan for a hypothetical mentee
2. Practice the SBI feedback model on past situations
3. Design a "scaffolded challenge" for a concept you know well

### Solutions

<details>
<summary>Exercise 1: 90-Day Growth Plan</summary>

```markdown
## 90-Day Plan: Frontend Mentee

### Days 1-30
- Learn codebase structure
- Ship 2 small UI fixes
- Pair twice per week
- Weekly 1:1 with feedback notes

### Days 31-60
- Own a feature end-to-end
- Write 3 unit/integration tests
- Lead a small demo

### Days 61-90
- Refactor a component safely
- Review a teammate PR
- Present a lesson learned
- Update growth plan based on strengths
```

**Key points:**
- Goals are staged and measurable.
- Work scales from small to independent.
- Feedback checkpoints are built in.

</details>

<details>
<summary>Exercise 2: SBI Feedback</summary>

```markdown
Situation: Yesterday's API bug fix
Behavior: You pushed without testing the error path
Impact: The staging environment broke for the team
Next step: Add an error-path test and run it before pushing
```

**Key points:**
- Describe facts, not opinions.
- Tie behavior to impact.
- Offer a next step.

</details>

<details>
<summary>Exercise 3: Scaffolded Challenge</summary>

```markdown
### Challenge: Build a Filtered List
Step 1: Render a static list
Step 2: Add a text input
Step 3: Filter based on input
Step 4: Add empty state messaging
Step 5: Add a small test for the filter behavior
```

**Key points:**
- Break tasks into progressive steps.
- Add constraints that teach fundamentals.
- Celebrate completion at each step.

</details>

---

## What You Learned

This module covered:

- **Mentoring mindset**: Growth-focused guidance
- **Structure**: Plans and milestones that scale
- **Teaching**: Tailored techniques for different learners
- **Feedback**: Clear, actionable coaching
- **Balance**: Mentoring while delivering work

**Key takeaway**: Mentoring multiplies impact when it is intentional and structured.

---

## Real-World Application

This week at work, you might use these concepts to:

- Create a plan for a new teammate
- Run a structured pairing session
- Give SBI-style feedback on a PR
- Turn repeated questions into documentation
- Set mentoring hours on your calendar

---

## Further Reading

- [The Manager's Path](https://www.oreilly.com/library/view/the-managers-path/9781491973882/)
- [Radical Candor](https://www.radicalcandor.com/)

---

**Navigation**: [← Previous Module](./13-code-review-mastery.md) | [Next Module →](./15-nextjs-advanced.md)
