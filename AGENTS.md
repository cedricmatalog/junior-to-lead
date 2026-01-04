# Repository Guidelines

## Project Structure & Module Organization
This repository is currently a scaffold with no source directories committed yet. Use the following structure for new work so the project stays consistent as it grows:

- `src/` for application/library code.
- `tests/` for automated tests, mirroring `src/` paths.
- `docs/` for design notes, ADRs, and contributor docs.
- `scripts/` for build/test helpers.
- `assets/` for static files (images, fixtures, etc.).

If you introduce a different layout, document it here and keep top-level directories minimal.

## Build, Test, and Development Commands
No build or test tooling is configured yet. When adding tooling, include a minimal set of commands in this section. Example patterns:

- `npm run dev` — start a local dev server.
- `npm test` — run the test suite.
- `make build` — produce a production build.

Keep commands simple and ensure they run from the repository root.

## Coding Style & Naming Conventions
Follow the existing style in a file. If creating new code, use these defaults:

- Indentation: 2 spaces for JSON/YAML/JS/TS; 4 spaces for Python; tabs only if the language standard requires it.
- Naming: `kebab-case` for file names, `PascalCase` for types/classes, `camelCase` for variables and functions.
- Formatting: add a formatter when you introduce a language (`prettier`, `black`, `gofmt`, etc.) and keep it in version control.

## Testing Guidelines
No test framework is set up yet. If you add one, document it here and follow a clear naming convention:

- JavaScript/TypeScript: `*.test.ts` or `*.spec.ts` in `tests/`.
- Python: `test_*.py` in `tests/`.

Target meaningful coverage for core logic and add regression tests for every bug fix.

## Commit & Pull Request Guidelines
There is no established commit history yet. Use Conventional Commits for new work, e.g.:

- `feat: add initial parser`
- `fix: handle empty input`

PRs should include a short description, testing notes (commands run), and screenshots or logs when behavior changes.

## Security & Configuration Tips
Store secrets in environment variables or local config files ignored by git (e.g., `.env`). Do not commit credentials. Document required env vars in `docs/` or this file as they are introduced.
