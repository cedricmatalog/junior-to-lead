# Hiring and Onboarding

> **Last reviewed**: 2026-01-06


## Week 35: The Team Expansion

Headcount is approved and the team needs to grow fast without lowering the bar. Sarah asks you to design a hiring process that is fair, consistent, and aligned with team needs. Marcus reminds you that onboarding is just as important as hiring -- it determines how quickly new hires become effective. This week focuses on structured interviews, bias reduction, and onboarding plans that work.

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Interview process** | A funnel - consistent steps reduce noise |
| **Rubrics** | A scorecard - compare candidates fairly |
| **Bias reduction** | Guardrails - protect decisions from shortcuts |
| **Onboarding** | A runway - accelerate takeoff |

Keep these in mind. They'll click as we build.

---

## Prerequisites

Module 34 (Team Processes) - Comfort with team rhythms and expectations.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Design structured interview loops and rubrics
- [ ] Reduce bias in evaluation and feedback
- [ ] Run effective hiring debriefs
- [ ] Build 30-60-90 day onboarding plans
- [ ] Create onboarding guides that scale

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice hiring/onboarding leadership over 8-12 weeks

---

## Chapter 1: Designing Interviews

### Interview Structure

```markdown
## Frontend Developer Interview Process

### Stage 1: Recruiter Screen (30 min)
- Role overview
- Experience review
- Logistics (salary, timeline)
- Culture/values fit

### Stage 2: Technical Screen (60 min)
- Live coding: Build a small component
- Focus: Problem-solving, React fundamentals
- Interviewer: Senior engineer

### Stage 3: Onsite/Virtual (4 hours)
1. System Design (60 min)
   - Design a frontend feature
   - Focus: Architecture thinking

2. Coding Deep Dive (90 min)
   - Larger implementation
   - Focus: Code quality, testing

3. Behavioral (45 min)
   - Past experiences
   - Focus: Collaboration, growth

4. Team Meet (45 min)
   - Casual conversation
   - Focus: Culture add, questions

### Stage 4: Hiring Committee
- Review all feedback
- Make decision
- Reference checks
```

### Coding Assessment Design

```markdown
### Assessment: Build a Task Manager Component

### Time: 60 minutes

### Requirements
Build a task list component that:
1. Displays a list of tasks
2. Allows adding new tasks
3. Allows marking tasks complete
4. Filters tasks (all/active/complete)

### Starter Code Provided
- Basic project setup
- API mock with data
- CSS reset

### What We're Looking For
- React patterns and hooks usage
- Component structure
- State management choices
- Code organization
- Handling edge cases

### What We're NOT Looking For
- Perfect CSS
- 100% test coverage
- Knowing our specific stack

### Candidate Instructions
- Ask clarifying questions
- Think out loud
- It's okay to look things up
- Focus on working code first
```

### Evaluation Rubric

```markdown
### Coding Assessment Rubric

### Problem Solving (1-4)
4: Identifies edge cases, elegant solutions
3: Solves problem correctly, reasonable approach
2: Needs hints, gets there eventually
1: Struggles significantly

### React Skills (1-4)
4: Idiomatic React, advanced patterns
3: Solid fundamentals, good practices
2: Basic understanding, some issues
1: Major gaps in understanding

### Code Quality (1-4)
4: Clean, well-organized, good naming
3: Readable, minor improvements possible
2: Works but messy, needs refactoring
1: Hard to follow, poor organization

### Communication (1-4)
4: Clear explanations, good questions
3: Explains thinking, engages well
2: Some communication, needs prompting
1: Difficult to understand approach

### Overall: Sum / 16
14-16: Strong hire
10-13: Hire
7-9:   Borderline, more signal needed
Below 7: No hire
```

## Chapter 2: Reducing Bias

### Structured Interviews

```markdown
### Behavioral Interview Questions

Ask all candidates the same questions:

1. "Tell me about a technical project you're proud of."
   - Follow-up: What was your specific role?
   - Follow-up: What would you do differently?

2. "Describe a time you had to learn something new quickly."
   - Follow-up: How did you approach it?
   - Follow-up: What was the outcome?

3. "Tell me about a disagreement with a teammate."
   - Follow-up: How did you resolve it?
   - Follow-up: What did you learn?

**Score each answer before moving on.**
**Don't compare candidates until all interviews done.**
```

### Avoiding Common Biases

| Bias | Description | Mitigation |
|------|-------------|------------|
| Similarity | Prefer people like us | Diverse panel, structured scoring |
| Halo effect | One trait influences all | Separate criteria, multiple interviewers |
| Anchoring | First impression dominates | Score during interview, not after |
| Confirmation | Seek evidence for initial impression | Devil's advocate in debrief |

## Chapter 3: Making Decisions

### Debrief Structure

```markdown
### Hiring Debrief Agenda

1. **Individual scores shared** (5 min)
   - No discussion yet
   - Just numbers on the board

2. **Concerns first** (10 min)
   - What gave you pause?
   - Specific examples only

3. **Strengths** (10 min)
   - What impressed you?
   - Specific examples only

4. **Discussion** (15 min)
   - Resolve conflicting signals
   - Ask about gaps in assessment

5. **Decision** (5 min)
   - Hire / No hire / Need more info
   - Document reasoning
```

## Chapter 4: Onboarding

### First Day

```markdown
### Day 1 Checklist

### Before They Arrive
- [ ] Laptop configured
- [ ] Accounts created (GitHub, Slack, etc.)
- [ ] Calendar invites sent
- [ ] Buddy assigned
- [ ] Welcome message in #general

### Day 1 Schedule
09:00 - Welcome, laptop setup
10:00 - HR paperwork, benefits
11:00 - Meet the team (casual)
12:00 - Lunch with buddy
13:00 - Development setup walkthrough
15:00 - Codebase tour
16:00 - First small task (update README)
17:00 - End of day check-in
```

### First Week

```markdown
### Week 1 Goals

### Learn
- [ ] Understand product and users
- [ ] Navigate codebase structure
- [ ] Know team members and roles
- [ ] Understand development workflow

### Do
- [ ] Complete dev environment setup
- [ ] Ship one small PR
- [ ] Attend all team ceremonies
- [ ] 1:1 with manager and tech lead

### Connect
- [ ] Coffee chat with 3 team members
- [ ] Ask questions in Slack
- [ ] Join relevant channels
```

### 30-60-90 Day Plan

```markdown
### 30-60-90 Day Onboarding Plan

### First 30 Days: Learn
- Complete all onboarding tasks
- Shadow on 2-3 features
- Ship 3-5 small PRs
- Understand deployment process
- **Checkpoint**: Can explain our architecture

### Days 31-60: Contribute
- Own a medium-sized feature
- Participate in code reviews
- Join on-call rotation (shadow)
- Identify first improvement idea
- **Checkpoint**: Shipping independently

### Days 61-90: Own
- Lead a feature from start to finish
- Mentor newer team member
- Present in team meeting
- Propose and implement improvement
- **Checkpoint**: Fully productive team member
```

### Onboarding Documentation

```markdown
## Chapter 5: New Developer Guide

### Getting Started
1. [Development Setup](./dev-setup.md)
2. [Architecture Overview](./architecture.md)
3. [Team Practices](./practices.md)

### Key Concepts
- [Our Design System](./design-system.md)
- [State Management](./state.md)
- [Testing Strategy](./testing.md)

### Processes
- [How to Deploy](./deployment.md)
- [Code Review Guidelines](./code-review.md)
- [On-Call Guide](./on-call.md)

### Who to Ask
- Technical questions: @tech-lead
- Process questions: @engineering-manager
- Product questions: @product-manager
```

---

## Common Mistakes

1. **Unstructured interviews** - Inconsistent feedback creates noise.
2. **Bias in scoring** - Gut feelings override evidence.
3. **No onboarding owner** - New hires drift without a guide.
4. **Overloading week one** - Too much information, too little retention.

## Practice Exercises

1. Design an interview loop for a senior frontend role
2. Create an evaluation rubric for a coding challenge
3. Build a 30-day onboarding checklist

### Solutions

<details>
<summary>Exercise 1: Interview Loop</summary>

```markdown
1. Recruiter screen (30 min)
2. Technical screen (60 min)
3. System design (60 min)
4. Pairing session (90 min)
5. Values and collaboration (45 min)
```

**Key points:**
- Balance technical depth and collaboration.
- Use the same loop for all candidates.
- Assign clear owners per stage.

</details>

<details>
<summary>Exercise 2: Rubric</summary>

```markdown
Criteria (1-5):
- Problem solving
- Code quality
- React fundamentals
- Communication
```

**Key points:**
- Score against defined criteria.
- Capture evidence with examples.
- Use rubric in debrief to avoid bias.

</details>

<details>
<summary>Exercise 3: 30-Day Checklist</summary>

```markdown
Week 1: Setup, documentation, first small PR
Week 2: Pairing sessions, first feature
Week 3: Own a ticket end-to-end
Week 4: Demo work and review goals
```

**Key points:**
- Focus on quick wins early.
- Provide a clear progression.
- End with a checkpoint and feedback.

</details>

---

## What You Learned

This module covered:

- **Interview design**: Structured loops and scoring
- **Bias reduction**: Fair, consistent evaluations
- **Debriefs**: Clear hiring decisions
- **Onboarding**: Fast ramp with guidance
- **Documentation**: Scalable onboarding materials

**Key takeaway**: Hiring and onboarding are systems, not one-off events.

---

## Real-World Application

This week at work, you might use these concepts to:

- Update your interview loop and rubrics
- Train interviewers on bias mitigation
- Create a 30-60-90 onboarding plan
- Write a new developer guide

---
## Further Reading

- [Work Rules!](https://www.workrules.net/) - Google's hiring insights
- [Who](https://whothebook.com/) - Hiring methodology

---

**Navigation**: [<- Previous Module](./03-team-processes.md) | [Next Module ->](./05-stakeholder-communication.md)
