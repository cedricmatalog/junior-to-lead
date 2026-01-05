# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **React Developer Curriculum** - a comprehensive learning resource that guides developers from junior to lead-level positions. It contains **42 educational modules** organized into 4 progressive levels (Junior → Mid-Level → Senior → Lead), covering both technical React skills and leadership competencies.

**This is a documentation/curriculum repository, not a code project.** There is no build system, test suite, or application code. All content is markdown-based learning materials.

## Repository Structure

```
junior-to-lead/
├── react/
│   ├── 01-junior/        # 10 modules: React fundamentals through accessibility
│   ├── 02-mid-level/     # 10 modules: Advanced patterns, performance, TypeScript
│   ├── 03-senior/        # 11 modules: Architecture, system design, mentoring
│   ├── 04-lead/          # 12 modules: Technical leadership, team management
│   └── README.md         # Main curriculum overview with navigation
├── docs/
│   └── react-audit-report.md  # Known issues and improvement tracking
└── AGENTS.md             # Generic repository guidelines (legacy)
```

## Content Architecture

### Module Format
Each module follows a consistent storytelling structure:
- **Story frame**: Week-by-week narrative with characters (Sarah the tech lead, Marcus the colleague)
- **Mental Models table**: Quick conceptual reference at the start
- **Chapter-based structure**: Breaking down topics into digestible sections
- **Practice exercises**: Hands-on tasks for skill application
- **Further Reading**: External references
- **Navigation links**: Previous/Next module navigation at the bottom

### Levels Overview
- **Junior (10 modules)**: Foundation - JSX, hooks, state, forms, testing, debugging, git, accessibility
- **Mid-Level (10 modules)**: Proficiency - Advanced patterns, performance, state at scale, TypeScript, Next.js
- **Senior (11 modules)**: Expertise - Architecture, system design, security, design systems, mentoring
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
4. Ensure navigation links remain intact
5. Update the module's README if key topics change

### Adding New Content
When adding chapters or sections:
1. Use real-world examples tied to the story frame
2. Include code examples with both ❌ BAD and ✅ GOOD patterns
3. Add practice exercises at the end if introducing new skills
4. Consider if it needs a Mental Models entry
5. Check if the level README needs updating

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
