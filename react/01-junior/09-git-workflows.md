# Git Workflows

## Week Nine: Your First Real Feature

Nine weeks in. You've built components, fixed bugs, shipped features. But there's something you've been dreading.

"Time for your first solo feature," Sarah says. "The checkout payment integration. You'll own it start to finish—including the Git workflow."

Until now, you've been pushing to a branch someone else set up. Now you need to manage branches, handle conflicts, and get code reviewed.

"Git isn't just version control," Marcus adds. "It's how teams work together without stepping on each other's toes."

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Branches** | Parallel universes - experiment without affecting the main timeline |
| **Commits** | Save checkpoints - go back if things go wrong |
| **Pull Requests** | Asking for feedback - review before publishing |

Keep these in mind. They'll click as we build.

---

## Chapter 1: Starting Clean

Day one of the payment feature. Sarah walks you through the proper start.

```bash
# Start from updated main
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/checkout-payment-integration

# Work on feature, commit regularly
git add .
git commit -m "feat(checkout): scaffold payment form component"

# Push and create PR
git push -u origin feature/checkout-payment-integration
```

"Always start from a fresh main," she emphasizes. "If you branch from stale code, you're asking for conflicts."

**Branch naming conventions:**
- `feature/description` - New features
- `fix/description` - Bug fixes
- `refactor/description` - Code refactoring
- `docs/description` - Documentation updates

---

## Chapter 2: Commit Messages That Tell a Story

Marcus reviews your first few commits.

```bash
# Your commits
git commit -m "updated stuff"
git commit -m "more changes"
git commit -m "fixed thing"
```

"These tell me nothing," he says. "In six months, you won't remember what 'thing' was."

He introduces Conventional Commits:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation
- `style` - Formatting (not CSS)
- `refactor` - Code restructuring
- `test` - Adding tests
- `chore` - Build/tooling changes

You redo your commits:

```bash
# Good commits
git commit -m "feat(checkout): add credit card input with validation"
git commit -m "feat(checkout): implement Luhn algorithm for card validation"
git commit -m "feat(checkout): add Stripe integration hook"
git commit -m "fix(checkout): handle Amex 15-digit card numbers"
```

"Now I can read the history and understand exactly what changed."

---

## Chapter 3: The Conflict

Day three. You're about to push, but Marcus messages: "I just merged some checkout changes. You might want to rebase."

You pull main and... conflicts.

```bash
git checkout main
git pull origin main
git checkout feature/checkout-payment-integration
git rebase main

# CONFLICT (content): Merge conflict in src/components/CheckoutPage.jsx
```

You open the file and see the markers:

```jsx
<<<<<<< HEAD
const greeting = "Hello";
=======
const greeting = "Hi there";
>>>>>>> feature/new-greeting
```

"Above the `=======` is what's in main," Sarah explains. "Below is your code. Choose one, combine them, whatever makes sense."

```jsx
// You decide to combine
const greeting = "Hello there";
```

Then continue:

```bash
git add src/components/CheckoutPage.jsx
git rebase --continue

# After rebase, push with --force-with-lease
git push --force-with-lease origin feature/checkout-payment-integration
```

"Why `--force-with-lease` instead of `--force`?" you ask.

"Safety. If someone else pushed to your branch, it won't overwrite their changes."

---

## Chapter 4: The Pull Request

Time to request review. You open GitHub and start typing the PR description.

```markdown
## Description
Brief explanation of what this PR does.

## Changes
- Added login form component
- Integrated with auth API
- Added form validation

## Testing
- [ ] Unit tests pass
- [ ] Tested login flow manually
- [ ] Tested error states

## Screenshots
[If UI changes, include before/after]
```

Sarah reviews your PR description:

```markdown
## Context
We're integrating Stripe for payment processing (JIRA-189).

## Changes
- New PaymentForm component with card validation
- usePaymentProcessor hook for Stripe API
- Updated CheckoutPage to use new payment flow

## Testing
- Tested with Stripe test cards (4242...)
- Verified error states with declined cards
- Checked mobile responsive layout

## Screenshots
[Before/After images]
```

"Perfect. A reviewer can understand the PR without reading every line of code."

**PR best practices:**
1. **Keep PRs small** - 200-400 lines ideal
2. **One concern per PR** - Don't mix features
3. **Self-review first** - Catch obvious issues
4. **Respond to feedback** - Discuss, don't defend
5. **Update with latest main** - Before merging

---

## Chapter 5: Code Review Feedback

The reviews come in. Marcus has suggestions:

```markdown
> Consider using useMemo here since this calculation is expensive

> This could throw if user is null - should we add a guard?
```

"Good review comments explain *why*," Sarah notes. "Not just 'this is wrong.'"

You address the feedback:

```bash
git add src/utils/cardValidation.js src/components/PaymentForm.jsx
git commit -m "refactor(checkout): extract card validation to utility module"

git add src/components/PaymentForm.jsx
git commit -m "feat(checkout): add loading spinner during payment processing"

git add src/components/PaymentForm.jsx
git commit -m "fix(checkout): add aria-live for payment error messages"

git push origin feature/checkout-payment-integration
```

---

## Chapter 6: The Merge

Approved! But before merging, one last sync:

```bash
git checkout main
git pull origin main
git checkout feature/checkout-payment-integration
git rebase main
git push --force-with-lease origin feature/checkout-payment-integration
```

You click "Squash and merge" in GitHub. The commits become one clean entry:

```
feat(checkout): add payment integration (#234)

This PR adds Stripe payment processing to the checkout flow:
- Credit card input with real-time validation
- Support for Visa, Mastercard, Amex
- Loading states and error handling
- Accessible error messages

Closes #189
```

Clean up your local branch:

```bash
git checkout main
git pull origin main
git branch -d feature/checkout-payment-integration
```

---

## Chapter 7: The Hotfix

Friday afternoon. Production alert. The checkout is crashing.

"Hotfix time," Marcus says. "Different workflow."

```bash
# Branch directly from main
git checkout main
git pull origin main
git checkout -b hotfix/checkout-null-crash

# Quick fix
git add src/components/CheckoutPage.jsx
git commit -m "fix(checkout): handle null cart items in payment flow"

# Push immediately
git push -u origin hotfix/checkout-null-crash
```

The PR gets expedited review—someone's always available for hotfixes. After merge:

```bash
git checkout main
git pull origin main
git tag -a v1.2.1 -m "Hotfix: checkout null crash"
git push origin v1.2.1
```

"Hotfixes skip the normal queue," Sarah explains. "But they still need review. A bad fix can make things worse."

---

## Chapter 8: Staying in Sync

"How do I avoid conflicts in the first place?" you ask.

Marcus shares the secrets:

```bash
# Option 1: Merge main into your branch (preserves history)
git checkout feature/my-feature
git merge main

# Option 2: Rebase onto main (cleaner history)
git checkout feature/my-feature
git rebase main
```

"Communicate with your team too."

```markdown
# Slack message when starting overlapping work
"Hey team, I'm working on the checkout payment form today.
I'll be touching CheckoutPage.jsx and adding new files in /components/checkout/.
Let me know if anyone else needs to work in that area."
```

**Conflict prevention:**
- Pull main frequently
- Communicate about overlapping work
- Keep PRs small and merge often

---

## Chapter 9: Your Git Toolkit

Sarah shares the commands you'll use daily:

```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Discard local changes to a file
git restore filename

# Stash changes temporarily
git stash
git stash pop

# See commit history
git log --oneline -10
git log --oneline --graph

# Find when a bug was introduced
git bisect start
git bisect bad  # Current commit is bad
git bisect good abc123  # This commit was good
# Git helps find the breaking commit

# Amend last commit message
git commit --amend -m "new message"

# Cherry-pick a commit
git cherry-pick abc123

# Interactive staging (choose specific changes)
git add -p
```

---

## Chapter 10: Automating Quality

"One more thing," Marcus adds. "Git hooks."

```bash
# .husky/pre-commit
npm run lint
npm run test:unit
```

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  }
}
```

"Now you literally can't commit if tests fail or linting errors exist."

---

## Chapter 11: Secrets and Configuration

Before you push, Marcus catches something in your code.

"Wait—is that an API key hardcoded in there?"

You look. It is. The Stripe test key, right in the component.

"That can never go to GitHub," he says. "Let me show you environment variables."

### The .env File

```bash
# .env.local (never commit this!)
VITE_STRIPE_KEY=pk_test_abc123
VITE_API_URL=http://localhost:3001
VITE_FEATURE_FLAG_CHAT=true
```

```jsx
// Access in your code
const stripeKey = import.meta.env.VITE_STRIPE_KEY;
const apiUrl = import.meta.env.VITE_API_URL;
```

### Framework Differences

"Different frameworks handle this differently," Sarah explains.

```bash
# Vite projects - prefix with VITE_
VITE_API_URL=https://api.example.com

# Create React App - prefix with REACT_APP_
REACT_APP_API_URL=https://api.example.com

# Next.js - prefix with NEXT_PUBLIC_ for client-side
NEXT_PUBLIC_API_URL=https://api.example.com
```

```jsx
// Vite
const url = import.meta.env.VITE_API_URL;

// Create React App
const url = process.env.REACT_APP_API_URL;

// Next.js (client-side)
const url = process.env.NEXT_PUBLIC_API_URL;
```

### The .env.example File

"Create a template for other developers," Marcus adds.

```bash
# .env.example (commit this!)
# Copy to .env.local and fill in values

# Stripe API key (get from dashboard.stripe.com)
VITE_STRIPE_KEY=

# Backend API URL
VITE_API_URL=http://localhost:3001

# Feature flags
VITE_FEATURE_FLAG_CHAT=false
```

### What Goes in .gitignore

```bash
# .gitignore
.env
.env.local
.env.*.local

# Keep the example
!.env.example
```

### Environment-Specific Configs

```bash
# .env.development
VITE_API_URL=http://localhost:3001
VITE_DEBUG=true

# .env.production
VITE_API_URL=https://api.production.com
VITE_DEBUG=false

# .env.test
VITE_API_URL=http://localhost:3001
VITE_DEBUG=false
```

### Type Safety for Environment Variables

```typescript
// src/env.d.ts (Vite)
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_STRIPE_KEY: string;
  readonly VITE_API_URL: string;
  readonly VITE_FEATURE_FLAG_CHAT: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

```typescript
// Validate required variables at startup
function validateEnv() {
  const required = ['VITE_STRIPE_KEY', 'VITE_API_URL'];

  for (const key of required) {
    if (!import.meta.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
}

validateEnv();
```

**The rules:**
1. Never commit secrets to Git
2. Create `.env.example` as documentation
3. Use framework-specific prefixes
4. Validate required variables on startup
5. Different values for different environments

By Friday, your payment feature is live. The commit history is clean. No conflicts. No broken builds. And no secrets in the repo.

"You're not just writing code anymore," Sarah says. "You're collaborating."

---

## Git Workflow Summary

| Stage | Commands |
|-------|----------|
| Start feature | `git checkout main && git pull && git checkout -b feature/name` |
| Save progress | `git add . && git commit -m "feat: description"` |
| Stay updated | `git rebase main` |
| Push branch | `git push -u origin feature/name` |
| After review | `git rebase main && git push --force-with-lease` |
| After merge | `git checkout main && git pull && git branch -d feature/name` |

## Common Mistakes

1. **Committing without pulling** - Always start from fresh main
2. **Vague commit messages** - Your future self will hate you
3. **Giant PRs** - Small PRs get reviewed faster
4. **Ignoring conflicts** - They don't go away on their own

## Practice Exercises

1. Practice the feature branch workflow on a sample repo
2. Write 5 commits following Conventional Commits
3. Create a merge conflict and resolve it

## Further Reading

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Branching - Basic Branching and Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
