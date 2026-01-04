# Mentoring Juniors

Help others grow while multiplying your own impact.

## Learning Objectives

By the end of this module, you will:
- Structure effective mentoring relationships
- Adapt teaching to different learning styles
- Provide feedback that accelerates growth
- Balance mentoring with your own work

## Mentoring Mindset

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

## Structuring Mentorship

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

## Teaching Techniques

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
## Challenge: Build a Search Component

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
## Pairing Modes

**Driver-Navigator**
- Junior drives: Types code, thinks out loud
- You navigate: Guide direction, catch errors
- Switch periodically

**Watch and Learn**
- You drive: Explain your thought process
- Junior watches: Asks questions
- Good for: Complex problems, showing patterns

**Ping-Pong**
- Alternate: You write test, they write code
- Good for: TDD, keeping both engaged
```

## Giving Feedback

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

## Unblocking Without Solving

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

## Balancing Mentoring with Work

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

## Practice Exercises

1. Create a 90-day growth plan for a hypothetical mentee
2. Practice the SBI feedback model on past situations
3. Design a "scaffolded challenge" for a concept you know well

## Further Reading

- [The Manager's Path](https://www.oreilly.com/library/view/the-managers-path/9781491973882/)
- [Radical Candor](https://www.radicalcandor.com/)
