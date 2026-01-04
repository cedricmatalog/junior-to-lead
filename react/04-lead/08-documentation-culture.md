# Documentation Culture

Build knowledge systems that scale with your team.

## Learning Objectives

By the end of this module, you will:
- Create documentation that gets used
- Establish README and ADR standards
- Build effective knowledge management
- Encourage a documentation-first culture

## Why Documentation Matters

### The Cost of Missing Documentation

```markdown
## Scenario: Missing Documentation Impact

**Without documentation:**
- New engineer asks 5 questions/day
- Each question takes 15 min from someone
- 5 engineers × 15 min × 5 questions = 6+ hours/day of interruption
- Onboarding takes 3 months

**With documentation:**
- New engineer finds 4 of 5 answers in docs
- 1 question/day, still captured in docs
- Onboarding takes 1 month
- Knowledge persists when people leave
```

## Types of Documentation

### Documentation Map

```markdown
## Documentation Types

### Reference
**Purpose**: Look up specific information
**Examples**: API docs, configuration options
**Updates**: When code changes
**Location**: Near the code

### Tutorial
**Purpose**: Learn how to do something
**Examples**: Getting started, how-to guides
**Updates**: Quarterly review
**Location**: Team wiki

### Explanation
**Purpose**: Understand why/how something works
**Examples**: Architecture docs, ADRs
**Updates**: When design changes
**Location**: docs/ folder

### Runbook
**Purpose**: Handle specific situations
**Examples**: Incident response, deployment
**Updates**: After incidents
**Location**: Runbook system
```

## README Standards

### Project README Template

```markdown
# Project Name

Brief description of what this project does.

## Quick Start

\`\`\`bash
npm install
npm run dev
\`\`\`

## Prerequisites

- Node.js 18+
- Access to [internal service]
- Environment variables (see .env.example)

## Development

### Setup

1. Clone the repository
2. Copy .env.example to .env.local
3. Run `npm install`
4. Run `npm run dev`

### Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server |
| `npm run build` | Build for production |
| `npm run test` | Run tests |
| `npm run lint` | Check code style |

### Architecture

Brief overview of how the code is organized.

See [Architecture Documentation](./docs/architecture.md) for details.

## Deployment

How to deploy this project.

## Contributing

Link to contribution guidelines.

## Team

- **Owner**: @team-name
- **Slack**: #channel-name
- **On-call**: link-to-rotation
```

### Component README Template

```markdown
# ComponentName

Brief description of the component's purpose.

## Usage

\`\`\`tsx
import { ComponentName } from './ComponentName';

<ComponentName
  prop1="value"
  prop2={value}
  onEvent={handleEvent}
/>
\`\`\`

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| prop1 | string | required | Description |
| prop2 | number | 0 | Description |
| onEvent | () => void | - | Description |

## Examples

### Basic Usage
[Example code]

### With Variants
[Example code]

## Notes

Any important implementation details or gotchas.
```

## Architecture Decision Records

### ADR Directory Structure

```
docs/
└── architecture/
    └── decisions/
        ├── README.md           # Index and how to write ADRs
        ├── 0001-use-nextjs.md
        ├── 0002-state-management.md
        └── template.md
```

### ADR Index

```markdown
# Architecture Decision Records

## What is an ADR?
An Architecture Decision Record captures an important architectural
decision made along with its context and consequences.

## When to Write an ADR
- Choosing between technologies
- Major architectural changes
- Decisions with long-term impact
- Decisions that were debated

## How to Write an ADR
1. Copy template.md
2. Number sequentially (0001, 0002, ...)
3. Fill in all sections
4. Get review from stakeholders
5. Merge when accepted

## ADR Index

| # | Title | Status | Date |
|---|-------|--------|------|
| 0001 | [Use Next.js](./0001-use-nextjs.md) | Accepted | 2024-01 |
| 0002 | [State Management](./0002-state-management.md) | Accepted | 2024-01 |
| 0003 | [Testing Strategy](./0003-testing-strategy.md) | Proposed | 2024-02 |
```

## Knowledge Management

### Wiki Structure

```markdown
## Team Wiki Organization

### Getting Started
- New Engineer Onboarding
- Development Setup
- Team Practices

### Technical
- Architecture Overview
- System Design Docs
- API Documentation
- Runbooks

### Process
- How We Work
- Code Review Guidelines
- Deployment Process
- Incident Response

### Reference
- Glossary
- Useful Links
- Contact Directory
```

### Keeping Docs Updated

```markdown
## Documentation Maintenance

### Triggers for Updates
- Code changes that affect documented behavior
- Onboarding questions not answered by docs
- Post-incident action items
- Quarterly review cycle

### Ownership
- Each doc has an owner
- Owner reviews quarterly
- Team can propose changes via PR

### Review Cycle
1. Monthly: Check for outdated content
2. Quarterly: Comprehensive review
3. Annually: Reorganize and archive

### Quality Checks
- Links still work?
- Screenshots current?
- Code examples run?
- Contact info accurate?
```

## Building Documentation Culture

### Making It Easy

```markdown
## Documentation Practices

### In Pull Requests
- README updated if behavior changes
- ADR for architectural decisions
- Inline comments for complex logic

### In Standups
- "I updated the wiki with..."
- "I couldn't find docs for X"

### In Onboarding
- First task: Fix something in docs
- Docs feedback is valuable
- Questions → Documentation gaps

### Recognition
- Celebrate good documentation
- Include in performance reviews
- "Docs Hero" of the month
```

### Documentation as Code

```markdown
## Docs-as-Code Practices

### Treat docs like code:
- Version control (git)
- Review process (PRs)
- Testing (link checks, spell check)
- CI/CD (auto-deploy)

### Example: Docusaurus Setup

\`\`\`yaml
# .github/workflows/docs.yml
name: Deploy Docs
on:
  push:
    branches: [main]
    paths: ['docs/**']

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm run docs:build
      - run: npm run docs:deploy
\`\`\`
```

## Practice Exercises

1. Create a README template for your team
2. Write an ADR for a recent decision
3. Audit existing documentation for gaps

## Further Reading

- [Divio Documentation System](https://documentation.divio.com/)
- [Write the Docs](https://www.writethedocs.org/)
