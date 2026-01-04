# React Curriculum Audit Report

Date: 2026-01-04
Scope: /home/cedric/junior-to-lead/react

## Audit Summary
- Scanned 47 Markdown files in `react/` (42 modules + 5 READMEs), ~31.7k words.
- All modules include Learning Objectives, Practice Exercises, and Further Reading.
- Module counts match the curriculum overview (Junior 9, Mid 10, Senior 11, Lead 12).
- Internal narrative links appear consistent; external links were not validated offline.

## Inventory
- `react/README.md` provides the overview and quick navigation across all 4 levels.
- Level READMEs include milestones, module tables, and progression guidance:
  - `react/01-junior/README.md`
  - `react/02-mid-level/README.md`
  - `react/03-senior/README.md`
  - `react/04-lead/README.md`
- Average module length increases by level (Junior ~563 words -> Lead ~832 words).

## Strengths
- Clear progression from fundamentals to architecture to leadership.
- Practical, example-driven modules with exercises and references.
- Strong balance of technical and leadership content at senior/lead levels.

## Findings (Fix First)
1) Invalid JS syntax in composition example:
   - `function FancyButton extends Button { ... }` is not valid JS.
   - Location: `react/01-junior/02-component-patterns.md:21`

2) Jotai example references an undefined atom:
   - `celsiusAtom` is not defined in the snippet.
   - Location: `react/02-mid-level/03-state-at-scale.md:155`

3) Type name collision in TypeScript examples:
   - `interface ButtonProps` and `type ButtonProps` defined in the same scope.
   - Location: `react/02-mid-level/08-typescript-mastery.md:221` and `react/02-mid-level/08-typescript-mastery.md:235`

## Findings (Version / API Drift)
1) React Router example uses `<Redirect />` (v5) rather than `<Navigate />` (v6).
   - Location: `react/02-mid-level/04-api-design-integration.md:178`

2) MSW POST handler uses `req.body` rather than `await req.json()`.
   - Location: `react/02-mid-level/07-testing-strategies.md:54`

3) FID is deprecated in favor of INP; `onFID` references may be outdated.
   - Location: `react/03-senior/03-advanced-performance.md:36` and `react/03-senior/03-advanced-performance.md:271`

4) `X-XSS-Protection` header is deprecated in modern browsers.
   - Location: `react/03-senior/04-security-practices.md:245`

5) `git checkout -- filename` is legacy; `git restore` is recommended in modern Git.
   - Location: `react/01-junior/09-git-workflows.md:176`

## Coverage Gaps / Opportunities
- Add a junior-level routing/navigation module (React Router basics) for non-Next.js paths.
- Add capstone projects per level plus a lightweight assessment checklist for measurable progress.
- Add an explicit tooling/module earlier (Vite/webpack basics, linting/formatting).
- Add a "last reviewed" or "version target" marker per module to reduce ecosystem drift.

## Recommended Next Steps
1) Fix the three high-severity code errors listed above.
2) Update version-sensitive examples (Router, MSW, web-vitals, security headers, Git).
3) Decide whether to add capstones + version tags as part of a 2026 refresh.
