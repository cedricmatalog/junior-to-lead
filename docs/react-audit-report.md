# React Curriculum Audit Report

Date: 2026-01-06
Scope: /home/cedric/junior-to-lead/react

## Audit Summary
- Scanned 48 Markdown files in `react/` (43 modules + 5 READMEs).
- All 43 modules include Mental Models, Prerequisites, Learning Objectives, Time Estimate, Common Mistakes, Practice Exercises + Solutions, What You Learned, Real-World Application, Further Reading, and Navigation.
- Navigation links are valid for all modules (no missing targets).
- Debug This Code coverage matches the guideline (8 of 10 junior modules).
- Voice and example markers were normalized (second-person voice; ❌/✅ examples in modules with JS/TS snippets).

## Inventory
- Module counts (actual): Junior 10, Mid 10, Senior 11, Lead 12 (total 43).
- `react/README.md` now reflects Junior 10 and total 43, and the quick nav includes Accessibility.
- `CLAUDE.md` reflects 43 modules in the structure section but still says 42 in the purpose blurb.

## Format Drift
The CLAUDE format expects 5-8 chapters per module. These junior modules exceed that range:
- `react/01-junior/05-forms-and-validation.md` (9 chapters)
- `react/01-junior/06-styling-approaches.md` (9 chapters)
- `react/01-junior/07-testing-fundamentals.md` (10 chapters)
- `react/01-junior/08-debugging-techniques.md` (11 chapters)
- `react/01-junior/09-git-workflows.md` (11 chapters)
- `react/01-junior/10-accessibility-fundamentals.md` (9 chapters)

## Content Balance
- Module length is inverted: Junior modules are the longest (avg ~4.2k words) while Lead modules are the shortest (avg ~1.3k words). Consider rebalancing so complexity and depth increase by level.

## Version / API Drift
- `react/01-junior/09-git-workflows.md` uses `git checkout --theirs/--ours`; consider `git restore --theirs/--ours` or clarify that checkout is legacy-but-supported.

## Coverage Gaps / Opportunities
- Add a junior-level routing/navigation module (React Router basics for non-Next.js apps).
- Add capstone projects per level plus a lightweight assessment checklist.
- Add an explicit early tooling module (Vite/webpack basics, linting/formatting).
- Add a “last reviewed” or “version target” marker per module to reduce ecosystem drift.

## Recommended Next Steps
1) Reduce chapter counts for the remaining junior modules to align with the 5-8 chapter guideline.
2) Update `CLAUDE.md` purpose blurb to 43 modules to match the structure section.
3) Decide whether to add capstones + version tags as part of a 2026 refresh.
