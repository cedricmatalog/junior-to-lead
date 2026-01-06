# Tooling Fundamentals

> **Last reviewed**: 2026-01-06


## Week Nine: The Setup That Never Ends

The routing work is done, but the onboarding doc is thin. A new teammate asks how to run it locally.

You realize the project has grown, but the tooling story hasn't.

"Tools are the invisible scaffolding," Sarah says. "When they're solid, teams move fast. When they're shaky, everything slows down."

Marcus nods. "Let's make the scaffolding visible."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Dev server** | A kitchen line - fast feedback while you cook |
| **Build** | Packaging line - everything prepared for shipping |
| **Environment variables** | Locked cabinets - safe access to secrets |
| **Linting/formatting** | Guardrails - keep code consistent and safe |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Junior Module 07 (Styling Approaches) - Comfort building and running React components locally.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Understand `package.json` scripts and how they drive workflows
- [ ] Set up a Vite project and run the dev server
- [ ] Configure environment variables safely
- [ ] Add ESLint and Prettier for consistent code quality
- [ ] Build a production bundle and preview it locally
- [ ] Diagnose common tooling errors (ports, dependencies, builds)

---

## Time Estimate

- **Reading**: 50-70 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Apply tooling habits over 2-3 weeks

---

## Chapter 1: The Script That Runs It All

Sarah points at `package.json`.

"Every workflow is a script. Learn the scripts and you control the project."

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint .",
    "format": "prettier --write ."
  }
}
```

"If someone asks how to run the app, you answer with `npm run dev`."

---

## Chapter 2: The Vite Dev Server

You spin up the project.

```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

"Vite gives fast HMR," Marcus says. "That means edits show up instantly."

---

## Chapter 3: Environment Variables

Your teammate almost commits an API key.

"Stop. Put it in `.env.local`," Sarah says.

```bash
# .env.local
VITE_API_URL=https://api.example.com
VITE_STRIPE_KEY=pk_test_123
```

```jsx
const apiUrl = import.meta.env.VITE_API_URL;
```

"Only variables prefixed with `VITE_` are exposed to the client."

---

## Chapter 4: Linting and Formatting

Inconsistent code slows reviews. Linters fix that.

```bash
npm install -D eslint prettier eslint-config-prettier eslint-plugin-react eslint-plugin-react-hooks
```

```json
// .eslintrc.json
{
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "prettier"
  ],
  "env": { "browser": true, "es2022": true },
  "settings": { "react": { "version": "detect" } }
}
```

"Lint catches bugs. Prettier stops style debates."

---

## Chapter 5: Build and Preview

"Dev is not prod," Marcus warns. "Always build before release."

```bash
npm run build
npm run preview
```

"Preview shows the production bundle locally, so you catch issues early."

---

## Chapter 6: Common Tooling Errors

You hit three classic issues:

1. **Port already in use**
2. **Module not found**
3. **Build fails only in CI**

"Use the error message like a map," Sarah says. "It always points somewhere."

---

## Common Mistakes

1. **Running without scripts** - Always use `npm run` so the team is consistent.
2. **Committing secrets** - Use `.env.local` and `.gitignore`.
3. **Skipping builds** - A dev server can hide production errors.
4. **Disabling lint rules** - Fix the root cause instead of silencing warnings.


Example:
```jsx
// ❌ Directly committing secrets
const stripeKey = 'sk_test_123';

// ✅ Use env variables
const stripeKey = import.meta.env.VITE_STRIPE_KEY;
```

---

## Debug This Code

Before moving to the exercises, test your debugging skills. Each snippet has bugs - can you spot and fix them?

<details>
<summary>Challenge 1: Script That Fails</summary>

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint"
  }
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

Lint needs a target to run on.

</details>

<details>
<summary>Solution</summary>

**Bug**: `eslint` has no path specified.

**Fixed code:**

```json
{
  "scripts": {
    "lint": "eslint ."
  }
}
```

**Why it matters**: Without a target, linting won't check any files.

</details>
</details>

<details>
<summary>Challenge 2: Missing Vite Prefix</summary>

```bash
# .env.local
API_URL=https://api.example.com
```

```jsx
const apiUrl = import.meta.env.API_URL;
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

Vite only exposes variables with a specific prefix.

</details>

<details>
<summary>Solution</summary>

**Bug**: Missing `VITE_` prefix.

**Fixed code:**

```bash
# .env.local
VITE_API_URL=https://api.example.com
```

```jsx
const apiUrl = import.meta.env.VITE_API_URL;
```

**Why it matters**: Without the prefix, the variable is undefined in the browser.

</details>
</details>

<details>
<summary>Challenge 3: Format Script That Does Nothing</summary>

```json
{
  "scripts": {
    "format": "prettier"
  }
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

Prettier needs paths to format.

</details>

<details>
<summary>Solution</summary>

**Bug**: No file path or glob provided.

**Fixed code:**

```json
{
  "scripts": {
    "format": "prettier --write ."
  }
}
```

**Why it matters**: Without a target, formatting won't run.

</details>
</details>

---

## Practice Exercises

1. Add `lint` and `format` scripts to a new project
2. Create a `.env.local` and read it in a component
3. Build and preview a production bundle locally

### Solutions

<details>
<summary>Exercise 1: Lint + Format Scripts</summary>

```json
{
  "scripts": {
    "lint": "eslint .",
    "format": "prettier --write ."
  }
}
```

**Key points:**
- Use `.` so tools scan the repo.
- Keep scripts consistent across projects.
- Add scripts to onboarding docs.

</details>

<details>
<summary>Exercise 2: Read Env Variables</summary>

```bash
# .env.local
VITE_API_URL=https://api.example.com
```

```jsx
function Status() {
  return <p>API: {import.meta.env.VITE_API_URL}</p>;
}
```

**Key points:**
- Prefix variables with `VITE_` for browser exposure.
- Store secrets in `.env.local` and keep it out of git.
- Avoid hardcoding URLs in components.

</details>

<details>
<summary>Exercise 3: Build + Preview</summary>

```bash
npm run build
npm run preview
```

**Key points:**
- Preview uses the real production build.
- Catch issues that don't appear in dev.
- Always test the build before release.

</details>

---

## What You Learned

This module covered:

- **Scripts**: `package.json` is the command center
- **Dev server**: Vite delivers fast feedback loops
- **Env vars**: Safe configuration via `.env.local`
- **Lint/format**: Consistent code and fewer review debates
- **Builds**: Always test production bundles
- **Tooling errors**: Diagnose ports, deps, and CI failures

**Key takeaway**: Reliable tooling makes every developer faster.

---

## Real-World Application

This week at work, you might use these concepts to:

- Onboard a teammate with clear `npm run` commands
- Add a `.env.local` for a new API integration
- Fix a build that only fails in CI
- Introduce ESLint and Prettier to a messy repo
- Validate a production build before a release

---

## Further Reading

- [Vite Docs](https://vitejs.dev/guide/)
- [ESLint Getting Started](https://eslint.org/docs/latest/use/getting-started)
- [Prettier Documentation](https://prettier.io/docs/en/index.html)

---

**Navigation**: [← Previous Module](./08-responsive-layouts.md) | [Next Module →](./10-testing-fundamentals.md)
