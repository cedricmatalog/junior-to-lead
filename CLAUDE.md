# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **React Developer Curriculum** - a comprehensive learning resource that guides developers from junior to lead-level positions. It contains **63 educational modules** organized into 4 progressive levels (Junior → Mid-Level → Senior → Lead), covering both technical React skills and leadership competencies.

**This is a documentation/curriculum repository, not a code project.** There is no build system, test suite, or application code. All content is markdown-based learning materials.

## Repository Structure

```
junior-to-lead/
├── react/
│   ├── 01-junior/        # 13 modules: React fundamentals through responsive layouts
│   ├── 02-mid-level/     # 17 modules: Advanced patterns, performance, a11y tooling
│   ├── 03-senior/        # 21 modules: Architecture, system design, governance
│   ├── 04-lead/          # 12 modules: Technical leadership, team management
│   └── README.md         # Main curriculum overview with navigation
├── docs/
│   └── react-audit-report.md  # Known issues and improvement tracking
└── AGENTS.md             # Generic repository guidelines (legacy)
```

## Content Architecture

### Module Format
Each module follows a consistent storytelling structure with these standard sections:

#### 1. Title & Introduction (50-100 words)
- **H1**: Module number and descriptive title (e.g., "# React Fundamentals")
- **Opening hook**: Brief story scenario that sets up the week's challenge
- **Problem statement**: What workplace problem will this module solve?
- **Format**: Narrative setup introducing the situation without heavy technical detail

#### 2. Mental Models Table (3-4 concepts)
```markdown
| Concept | Mental Model |
|---------|-------------|
| **Concept Name** | Memorable analogy - concrete comparison that makes the abstract tangible |
```
- **Purpose**: Quick cognitive anchors for complex concepts
- **Style**: Use physical analogies (LEGO bricks, recipe cards, gift boxes)
- **Placement**: Immediately after introduction, before prerequisites
- **End with**: "Keep these in mind. They'll click as we build."

#### 3. Prerequisites (1-2 sentences)
- **Format**: `Module XX (Title) - Brief description of required knowledge.`
- **Example**: `Module 01 (React Fundamentals) - Understanding JSX, props, and basic components.`
- **First module only**: "None - this is the starting point."

#### 4. Learning Objectives (5-6 checkboxes)
```markdown
By the end of this module, you'll be able to:

- [ ] Specific skill with measurable outcome
- [ ] Another actionable skill
```
- **Start with action verbs**: Write, Build, Implement, Test, Debug, Apply
- **Be specific**: Not "understand hooks" but "Build custom hooks that encapsulate reusable logic"
- **Test orientation**: Each should be something you can verify through practice

#### 5. Time Estimate (3 lines)
```markdown
- **Reading**: 45-80 minutes
- **Exercises**: 2-5 hours
- **Mastery**: Practice timeline (1-2 weeks for basics, 4-8 weeks for complex topics)
```
- **Reading time**: Module length / 200 words per minute
- **Exercises**: Realistic time for 3 full solutions
- **Mastery**: How long to internalize through real work

#### 6. Chapters (5-8 narrative sections)
Each chapter follows this structure:

**Chapter Title Format**:
- Use "Chapter N:" prefix for main teaching sections
- **Descriptive H2**: Focus on the concept, not "Introduction" or "Overview"
- **Examples**: "Chapter 1: The Inheritance Trap", "Chapter 2: The Magic of Children"

**Chapter Content Pattern**:
```markdown
## Chapter N: Descriptive Title

[Narrative setup: Story introducing the problem]

[Dialogue: Character explains the concept]

[Code Example: Show the pattern]

[Visual Diagram: ASCII art for complex flows (optional)]

[Explanation: Why it matters, when to use it]
```

**Code Example Standards**:
```markdown
// ❌ BAD: Brief explanation of anti-pattern
[bad code example]

// ✅ GOOD: Brief explanation of best practice
[good code example]
```

**Visual Diagrams** (when needed):
- Use ASCII box drawings for: component hierarchies, data flow, state flow, architecture
- Width: 60 characters max
- Include legends/keys explaining symbols
- Example structure:
```
┌──────────────────────────────────────────────┐
│            Diagram Title                     │
├──────────────────────────────────────────────┤
│  [content with arrows, boxes, flow]          │
│  • Legend item                               │
└──────────────────────────────────────────────┘
```

**Chapter Closing**: Transition to next chapter with narrative continuity

#### 7. Common Mistakes (3-4 items)
```markdown
## Common Mistakes

1. **Mistake name** - Brief description and how to avoid
2. **Another mistake** - Description and fix
```
- **Placement**: After all chapters, before Debug section
- **Format**: Bold mistake name, dash, explanation
- **Optional**: Can use table format if comparing multiple aspects

#### 8. Debug This Code (3 challenges, 8/10 modules)
```markdown
## Debug This Code

Before moving to the exercises, test your debugging skills. Each snippet has bugs - can you spot and fix them?

<details>
<summary>Challenge N: Descriptive Title</summary>

[Buggy code snippet - syntactically valid, logically flawed]

**How many bugs can you find?** (Answer: X bugs)

<details>
<summary>Hint</summary>

[Conceptual hint, not the answer]

</details>

<details>
<summary>Solution</summary>

**Bug**: Description of the bug

**Fixed code:**

[Corrected code]

**Why it matters**: Real-world impact explanation

</details>

</details>
```
- **Placement**: Immediately before Practice Exercises
- **Bug count**: 1-3 bugs per challenge, clearly stated
- **Bugs**: Real mistakes developers make (missing keys, stale closures, mutation, etc.)
- **Nested details**: Hints and solutions inside challenge details tag

#### 9. Practice Exercises (3 exercises with solutions)
```markdown
## Practice Exercises

1. Brief description of exercise 1
2. Brief description of exercise 2
3. Brief description of exercise 3

### Solutions

<details>
<summary>Exercise 1: Title</summary>

[Full working code solution]

**Key points:**
- Bullet explaining important concept
- Another key learning point

</details>
```
- **Exercise descriptions**: One sentence each, numbered list
- **Solutions**: Complete, runnable code in collapsible details
- **Key points**: 3-5 bullets explaining why/how the solution works

#### 10. What You Learned (Summary)
```markdown
## What You Learned

This module covered:

- **Concept Name**: One-sentence explanation
- **Another Concept**: Brief summary
- **Third Concept**: Description

**Key takeaway**: One memorable sentence capturing the core lesson.
```
- **Format**: Bullet list with bold concept names
- **Key takeaway**: Bold label, single powerful sentence
- **Purpose**: Reinforce main concepts before moving forward

#### 11. Real-World Application (This week at work...)
```markdown
## Real-World Application

This week at work, you might use these concepts to:

- Specific workplace scenario using concept 1
- Another practical application
- Third real-world use case
```
- **Format**: Bullet list of 4-5 concrete scenarios
- **Start with**: "This week at work, you might use these concepts to:"
- **Be specific**: Reference actual features (checkout flow, user dashboard, etc.)
- **Make it immediate**: Frame as things to try this week

#### 12. Further Reading (2-4 resources)
```markdown
## Further Reading

- [Resource Title](URL)
- [Another Resource](URL)
```
- **Quality over quantity**: 2-4 carefully selected resources
- **Prefer**: Official docs (React.dev), established patterns (patterns.dev), trusted authors
- **Format**: Markdown links, brief descriptive titles

#### 13. Navigation (Previous/Next links)
```markdown
---

**Navigation**: [← Previous Module](./XX-previous.md) | [Next Module →](./YY-next.md)
```
- **Separator**: Horizontal rule before navigation
- **Format**: Bold "Navigation:", arrows in link text
- **First module**: Only "Next →"
- **Last module**: Only "← Previous"

### Storytelling Elements

**Characters (consistent across all modules):**
- **You (the reader)**: Junior/mid/senior developer navigating workplace challenges
- **Sarah**: Your tech lead - experienced, patient, explains the "why"
- **Marcus**: Your colleague - peer-level, shares practical tips and his own learning journey

**Narrative Arc:**
- **Opening**: Problem or challenge introduced
- **Development**: Characters guide you through learning concepts
- **Resolution**: You solve the problem, ship the feature
- **Transition**: Setup for next module's challenge

**Voice Guidelines:**
- **Second person**: Always "you", never "we" or "I"
- **Active voice**: "Sarah explains" not "It is explained by Sarah"
- **Conversational**: Natural dialogue, contractions okay
- **Technical accuracy**: Correct terminology, precise explanations
- **No fluff**: Every sentence teaches or advances the story

**Code Example Domains:**
- Vary across modules to show breadth
- **Common domains**: E-commerce (products, cart), banking (transactions), media (streaming, playlists), travel (bookings, flights), productivity (todos, notes)
- **Realistic**: Use believable feature names, realistic data structures

#### Debug This Code Format
Most junior-level modules (01-junior/) include a "Debug This Code" section with 3 debugging challenges:
- **Coverage**: 8 out of 10 modules (01-06, 08-09) - Testing and Accessibility modules excluded as they focus on debugging/testing themselves
- **Position**: Placed immediately before "Practice Exercises" section
- **Structure**: Each challenge includes:
  - Buggy code snippet (realistic, production-like scenarios)
  - "How many bugs can you find?" prompt
  - Nested `<details>` tags for hints
  - Nested `<details>` tags for solution with explanations
  - Multiple solution approaches where applicable
  - "Why it matters" explanations connecting bugs to real-world impact
- **Purpose**: Reinforce learning through active debugging, build pattern recognition, simulate code review scenarios

### Levels Overview
- **Junior (13 modules)**: Foundation - JSX, hooks, state, forms, testing, debugging, git, accessibility, routing, tooling, responsive layouts
- **Mid-Level (17 modules)**: Proficiency - Advanced patterns, performance, state at scale, TypeScript, Next.js, E2E, motion, deployment, state tool selection, a11y tooling, router data APIs, PWA/offline
- **Senior (21 modules)**: Expertise - Architecture, system design, security, design systems, mentoring, ADRs, budgets, production triage, modern React APIs, web workers, privacy logging, SRI
- **Lead (12 modules)**: Leadership - Technical direction, team processes, stakeholder management, conflict resolution

### Cross-Cutting Themes
1. **Real-world scenarios**: Each module uses practical, job-like situations
2. **Feynman technique**: Concepts explained through analogy and mental models
3. **Progressive complexity**: Each level builds on previous knowledge
4. **Varied domains**: Different modules use different product domains (e-commerce, travel, banking, etc.) to demonstrate breadth

## Common Editing Tasks

### Updating a Module
When modifying module content:
1. Preserve the storytelling format with character dialogue
2. Keep the Mental Models table at the top
3. Maintain chapter structure with clear headings
4. Keep "Debug This Code" section before "Practice Exercises" (junior modules only)
5. Ensure navigation links remain intact
6. Update the module's README if key topics change

### Adding New Content
When adding chapters or sections:
1. Use real-world examples tied to the story frame
2. Include code examples with both ❌ BAD and ✅ GOOD patterns
3. Add debug challenges if introducing common pitfalls (junior modules)
4. Add practice exercises at the end if introducing new skills
5. Consider if it needs a Mental Models entry
6. Check if the level README needs updating

### Adding Debug Challenges
When creating new "Debug This Code" challenges:
1. Base bugs on real-world mistakes developers commonly make
2. Include 2-4 bugs per challenge (not too obvious, not too obscure)
3. Provide progressive hints (conceptual → specific)
4. Show multiple valid solutions when applicable
5. Explain the "why" - connect bugs to real production impact
6. Use realistic variable names and scenarios from the module's domain
7. Ensure buggy code is syntactically valid but logically flawed

### Version/API Updates
Many modules reference specific library versions or APIs. When updating:
- Check `docs/react-audit-report.md` for known version drift issues
- Update deprecated APIs (e.g., React Router v5 → v6, MSW v1 → v2)
- Add notes about React version compatibility where relevant (e.g., "React 19 note")
- Consider adding version markers if introducing cutting-edge features

### Navigation Links
All modules should have navigation links at the bottom:
```markdown
---

**Navigation**: [← Previous Module](./XX-name.md) | [Next Module →](./YY-name.md)
```
First module of each level only has "Next →", last module only has "← Previous"

## Known Issues

Refer to [docs/react-audit-report.md](docs/react-audit-report.md) for:
- Code syntax errors in examples
- Deprecated API usage
- Version drift in library examples
- Coverage gaps and improvement opportunities

Priority issues include:
1. Invalid JS syntax in composition examples
2. Jotai example referencing undefined atoms
3. React Router using v5 syntax instead of v6
4. MSW examples using deprecated request API

## Content Quality Standards

### Code Examples
- Must be syntactically correct and runnable
- Show both anti-patterns (❌) and best practices (✅)
- Include comments explaining "why" not just "what"
- Use realistic variable names and scenarios

### Writing Style
- Active voice, conversational tone
- Second person ("you") for the reader
- Character names: You (reader), Sarah (tech lead), Marcus (colleague)
- Technical accuracy over simplification
- No emojis except in specific contexts (accessibility icons, status indicators)

### Accessibility
- All code examples should follow a11y best practices
- Reference WCAG 2.1 AA standards when relevant
- Include semantic HTML in examples
- Avoid color-only indicators in diagrams or tables

## Special Considerations

### Story Continuity
Different modules use different product domains (e-commerce, travel booking, banking, etc.). This is intentional - it demonstrates the breadth of real-world scenarios rather than building a single product across all modules.

### React Version Compatibility
- Assume React 18+ unless specified
- Note React 19 features explicitly (e.g., functional Error Boundaries)
- Don't remove older patterns that are still widely used (e.g., class-based Error Boundaries)

### Leadership Content
Lead-level modules (04-lead/) focus on soft skills and team management:
- Use different narrative style (less code, more scenarios)
- Include templates and frameworks where helpful
- Reference real organizational challenges
- Balance idealism with pragmatism
