# Git Workflows

> **Last reviewed**: 2026-01-06


## Week Thirteen: Your First Real Feature

Nine weeks in. You've built components, fixed bugs, shipped features. But there's something you've been dreading.

"Time for your first solo feature," Sarah says. "The checkout payment integration. You'll own it start to finish—including the Git workflow."

Until now, you've been pushing to a branch someone else set up. Now you need to manage branches, handle conflicts, and get code reviewed.

"Git isn't just version control," Marcus adds. "It's how teams work together without stepping on each other's toes."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Branches** | Parallel universes - experiment without affecting the main timeline |
| **Commits** | Save checkpoints - go back if things go wrong |
| **Pull Requests** | Asking for feedback - review before publishing |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Junior Module 11 (Debugging Techniques) - Understanding how to investigate and fix issues before committing code.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Create and manage feature branches following naming conventions
- [ ] Write meaningful commit messages using Conventional Commits format
- [ ] Resolve merge conflicts confidently using rebase or merge strategies
- [ ] Create pull requests with clear descriptions and proper testing documentation
- [ ] Use environment variables to keep secrets out of version control
- [ ] Apply proper Git workflows for features, hotfixes, and collaboration

---

## Time Estimate

- **Reading**: 60-75 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice Git workflows daily over 4-6 weeks until it becomes second nature

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

"These tell you nothing," he says. "In six months, you won't remember what 'thing' was."

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

"Now you can read the history and understand exactly what changed."

---

## Chapter 3: The Conflict

Day three. You're about to push, but Marcus messages: "Checkout changes just landed. You might want to rebase."

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

Sarah says, "A good PR description saves review time."

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

Marcus says, "Sync before you merge so nothing surprises you."

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

"How can you avoid conflicts in the first place?" you ask.

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

### Your Git Toolkit

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

### Automating Quality

"One more thing," Marcus adds. "Git hooks."

```bash
# One-time setup
npx husky init

# .husky/pre-commit
npm run lint
npm run test:unit
```

```bash
# .husky/commit-msg
npx --no -- commitlint --edit "$1"
```

"Now you literally can't commit if tests fail or linting errors exist."

---

### Secrets and Configuration

Before you push, Marcus catches something in your code.

"Wait—is that an API key hardcoded in there?"

You look. It is. The Stripe test key, right in the component.

"That can never go to GitHub," he says. "Here's how environment variables work."

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


Example:
```jsx
// ❌ Hardcoded secrets in code
const apiKey = 'sk_test_123';

// ✅ Read secrets from env vars
const apiKey = import.meta.env.VITE_API_KEY;
```

---

## Debug This Code

Before moving to the exercises, test your debugging skills. Each Git scenario has issues - can you spot and fix them?

<details>
<summary>Challenge 1: Broken Git History</summary>

```bash
# Developer's workflow
git checkout main
git checkout -b feature/new-button

# Make changes to Button.jsx
git add .
git commit -m "changes"

# Make more changes
git add .
git commit -m "more changes"

# Make final changes
git add .
git commit -m "fixed it"

git push origin feature/new-button
```

**How many issues can you find?** (Answer: 3 issues)

<details>
<summary>Hint</summary>

Think about commit messages, pulling latest changes, and how this history will look to reviewers.

</details>

<details>
<summary>Solution</summary>

**Issue 1**: Didn't pull latest main before creating branch - might be working on outdated code.

**Issue 2**: Vague commit messages ("changes", "more changes", "fixed it") don't follow Conventional Commits and provide no context.

**Issue 3**: Multiple tiny commits for what should be one logical change - makes history noisy.

**Fixed workflow:**

```bash
# 1. Start from fresh main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/add-primary-button

# 3. Make all your changes, then commit with meaningful message
git add .
git commit -m "feat: add primary button component with hover states

- Created Button component with primary variant
- Added hover and active states
- Included accessibility attributes (aria-label, role)
- Updated Storybook documentation"

# 4. If you made multiple WIP commits, squash them before pushing
git rebase -i HEAD~3  # Squash last 3 commits into one

# 5. Push to remote
git push -u origin feature/add-primary-button
```

**Why it matters**: Clean commit history makes code review easier, helps with debugging (git blame), and creates a professional project history.

</details>

</details>

<details>
<summary>Challenge 2: Merge Conflict Panic</summary>

```bash
# You're working on feature/update-navbar
git checkout feature/update-navbar

# Meanwhile, someone merged changes to Header.jsx on main
# You try to merge main into your branch
git merge main

# Git says: CONFLICT in src/components/Header.jsx
# You see this in the file:
<<<<<<< HEAD
<nav className="navbar">
  <Logo />
  <UserMenu />
</nav>
=======
<nav className="header">
  <Logo />
  <SearchBar />
  <UserMenu />
</nav>
>>>>>>> main

# Panicked developer does:
git merge --abort
# Then tries to push anyway
git push origin feature/update-navbar --force
```

**How many issues can you find?** (Answer: 2 issues)

<details>
<summary>Hint</summary>

Aborting the merge doesn't solve the problem. What's the correct way to handle conflicts? Also, what's wrong with the force push?

</details>

<details>
<summary>Solution</summary>

**Issue 1**: Aborting the merge instead of resolving the conflict - the problem still exists and will block PR merging.

**Issue 2**: Force pushing without resolving conflicts can overwrite others' work and doesn't fix the underlying issue.

**Fixed workflow:**

```bash
# 1. Try to merge (or preferably rebase) main
git checkout feature/update-navbar
git rebase main

# 2. When conflict appears, don't panic! Open the file
# CONFLICT in src/components/Header.jsx

# 3. Manually resolve by choosing what to keep
# Remove conflict markers and combine both changes:
<nav className="header">
  <Logo />
  <SearchBar />
  <UserMenu />
</nav>

# 4. Stage the resolved file
git add src/components/Header.jsx

# 5. Continue the rebase
git rebase --continue

# 6. Push with force-with-lease (safer than --force)
git push origin feature/update-navbar --force-with-lease
```

**Alternative - abort and ask for help:**

```bash
# If you're truly stuck, abort and ask
git rebase --abort
# Then communicate with your team:
# "Hey, I have a conflict in Header.jsx between my navbar changes
# and the recent header refactor. Can someone help me resolve it?"
```

**Why it matters**: Conflicts are normal in team development. Learning to resolve them properly is essential. Force-with-lease is safer than force because it checks if remote has changes you don't have.

</details>

</details>

<details>
<summary>Challenge 3: Commit Message Chaos</summary>

Look at these commit messages from a PR:

```bash
git log --oneline
a1b2c3d fixed bug
b2c3d4e updates
c3d4e5f wip
d4e5f6g more fixes
e5f6g7h final version
f6g7h8i actually final
g7h8i9j ok this is really final now
```

**How many issues can you find?** (Answer: Multiple issues)

<details>
<summary>Hint</summary>

Do these messages follow any convention? Can you tell what changed? Would these be helpful in 6 months?

</details>

<details>
<summary>Solution</summary>

**Issues**:
- No type prefix (feat, fix, chore, etc.)
- No descriptive message about what changed
- Multiple "final" commits suggest poor planning
- "wip" commits should be squashed before PR
- No context for why changes were made

**Fixed commit history (after interactive rebase):**

```bash
# Squash all those commits into logical units:
git rebase -i HEAD~7  # Interactive rebase last 7 commits

# Result should be clear, meaningful commits:
git log --oneline
a1b2c3d feat: add user authentication with JWT tokens
b2c3d4e fix: resolve memory leak in useEffect cleanup
c3d4e5f docs: update API documentation for auth endpoints
```

**Good commit message template:**

```bash
<type>: <short summary>

<optional body explaining why this change was needed>

<optional footer with breaking changes or issue references>
```

**Examples:**

```bash
# Good commit messages:
git commit -m "feat: add dark mode toggle to user preferences

Implements dark mode using CSS variables and localStorage
persistence. Updates all components to respect theme context."

git commit -m "fix: prevent form submission on Enter key in search input

Addresses issue where hitting Enter in search bar would submit
the entire form instead of triggering search.

Closes #123"

git commit -m "refactor: extract validation logic into custom hook

Moves form validation from component into useFormValidation hook
for better reusability across login and signup forms."
```

**Why it matters**: Six months from now when tracking down a bug, meaningful commit messages in `git log` or `git blame` are invaluable. They explain not just what changed, but why.

</details>

</details>

---

## Practice Exercises

1. Practice the feature branch workflow on a sample repo
2. Write 5 commits following Conventional Commits
3. Create a merge conflict and resolve it

### Solutions

<details>
<summary>Exercise 1: Feature Branch Workflow</summary>

**Step-by-step workflow:**

```bash
# 1. Start from fresh main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/add-user-authentication

# 3. Make changes and commit regularly
# (Create LoginForm.jsx)
git add src/components/LoginForm.jsx
git commit -m "feat(auth): add login form component with email and password fields"

# (Add validation)
git add src/components/LoginForm.jsx
git commit -m "feat(auth): add form validation for login fields"

# (Add API integration)
git add src/services/authService.js src/components/LoginForm.jsx
git commit -m "feat(auth): integrate login form with authentication API"

# (Add tests)
git add src/components/LoginForm.test.jsx
git commit -m "test(auth): add tests for login form validation and submission"

# 4. Push branch to remote
git push -u origin feature/add-user-authentication

# 5. Create pull request on GitHub
# (Use GitHub UI to create PR)

# 6. Address review feedback
git add src/components/LoginForm.jsx
git commit -m "refactor(auth): extract validation logic to separate utility"

git push origin feature/add-user-authentication

# 7. Before merging, sync with latest main
git checkout main
git pull origin main
git checkout feature/add-user-authentication
git rebase main

# Resolve any conflicts if they appear
# Then force push (since rebase rewrites history)
git push --force-with-lease origin feature/add-user-authentication

# 8. Merge via GitHub PR (squash and merge)

# 9. Clean up locally
git checkout main
git pull origin main
git branch -d feature/add-user-authentication
```

**Key points:**
- Always start from updated main branch
- Make small, focused commits
- Push early and often
- Sync with main before merging to catch conflicts early
- Use --force-with-lease (not --force) for safety
- Clean up merged branches

</details>

<details>
<summary>Exercise 2: Conventional Commit Messages</summary>

```bash
# Example 1: New feature
git commit -m "feat(cart): add ability to apply discount codes at checkout"

# Example 2: Bug fix
git commit -m "fix(cart): correct total calculation when removing items with quantity > 1"

# Example 3: Documentation
git commit -m "docs(readme): add setup instructions for local development"

# Example 4: Refactoring
git commit -m "refactor(api): extract fetch logic into reusable service layer"

# Example 5: Test addition
git commit -m "test(cart): add integration tests for checkout flow"

# Example 6: Feature with breaking change
git commit -m "feat(auth)!: migrate to OAuth 2.0

BREAKING CHANGE: Password-based authentication is no longer supported.
Users must authenticate using OAuth providers (Google, GitHub, etc.)"

# Example 7: Multiple related changes
git commit -m "feat(profile): add user profile editing

- Add ProfileForm component with validation
- Implement avatar upload with preview
- Add API endpoint for profile updates
- Include accessibility labels for screen readers"

# Example 8: Bug fix with issue reference
git commit -m "fix(navigation): prevent menu from closing on mobile when clicking submenu

Fixes #234"
```

**Format breakdown:**
```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

**Common types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code formatting (not CSS)
- `refactor`: Code restructuring
- `test`: Adding/updating tests
- `chore`: Build tools, dependencies

**Key points:**
- Use imperative mood ("add" not "added")
- Keep subject line under 72 characters
- Reference issues in footer
- Explain "why" in body, "what" is in the code
- Use BREAKING CHANGE footer for breaking changes

</details>

<details>
<summary>Exercise 3: Creating and Resolving Merge Conflicts</summary>

**Setup scenario:**

```bash
# 1. Create a test repository
mkdir git-conflict-practice
cd git-conflict-practice
git init
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit"

# 2. Create main branch content
cat > src/config.js << EOF
export const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3
};
EOF

git add src/config.js
git commit -m "Add initial configuration"

# 3. Create branch A
git checkout -b feature/update-timeout
# Modify config.js
cat > src/config.js << EOF
export const config = {
  apiUrl: 'https://api.example.com',
  timeout: 10000,  // Changed from 5000
  retries: 3
};
EOF
git add src/config.js
git commit -m "Increase API timeout to 10 seconds"

# 4. Go back to main and create branch B
git checkout main
git checkout -b feature/add-debug-mode
# Modify config.js (same lines!)
cat > src/config.js << EOF
export const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3,
  debug: true  // Added new line
};
EOF
git add src/config.js
git commit -m "Add debug mode configuration"

# 5. Merge branch A into main
git checkout main
git merge feature/update-timeout
# This works fine

# 6. Try to merge branch B - CONFLICT!
git merge feature/add-debug-mode
# Auto-merging src/config.js
# CONFLICT (content): Merge conflict in src/config.js
# Automatic merge failed; fix conflicts and then commit the result.
```

**Resolving the conflict:**

```bash
# View the conflict
cat src/config.js
# Shows:
# export const config = {
#   apiUrl: 'https://api.example.com',
# <<<<<<< HEAD
#   timeout: 10000,
#   retries: 3
# =======
#   timeout: 5000,
#   retries: 3,
#   debug: true
# >>>>>>> feature/add-debug-mode
# };

# Open in editor and resolve manually
# Combined version:
export const config = {
  apiUrl: 'https://api.example.com',
  timeout: 10000,  // Keep the updated timeout
  retries: 3,
  debug: true      // Keep the new debug option
};

# Stage the resolved file
git add src/config.js

# Check status
git status
# On branch main
# All conflicts fixed but you are still merging.

# Complete the merge
git commit
# Git opens editor with merge message, save and close

# Push to remote
git push origin main
```

**Alternative: Rebase approach (cleaner history):**

```bash
# Starting from the conflict scenario above
git checkout feature/add-debug-mode
git rebase main

# CONFLICT appears
# Same process to resolve in src/config.js

# After resolving:
git add src/config.js
git rebase --continue

# If more conflicts appear, repeat the process
# When done:
git checkout main
git merge feature/add-debug-mode
# This will be a fast-forward merge, no conflict
```

**Handling complex conflicts:**

```bash
# Use a merge tool
git mergetool

# Or check both versions side by side
git diff HEAD
git diff feature/add-debug-mode

# See the common ancestor version
git show :1:src/config.js  # Common ancestor
git show :2:src/config.js  # Current branch (HEAD)
git show :3:src/config.js  # Incoming branch

# If you want to take one side entirely
git restore --theirs src/config.js  # Take their version
# or
git restore --ours src/config.js    # Keep our version
git add src/config.js
git commit
```

**Abort if needed:**

```bash
# If you want to start over
git merge --abort
# or during rebase
git rebase --abort
```

**Key points:**
- Conflicts happen when same lines are modified in different branches
- Markers: <<<<<<< HEAD, =======, >>>>>>> branch-name
- Resolve by editing the file to the desired final state
- Remove conflict markers before committing
- Test the code after resolving to ensure it works
- Use --yours or --theirs to accept one side entirely
- git mergetool provides visual diff tools
- Communicate with teammates when resolving conflicts in shared code

</details>

---

## What You Learned

This module covered:

- **Feature Branches**: Create isolated branches from main with descriptive names (feature/, fix/, refactor/)
- **Commit Messages**: Use Conventional Commits (feat, fix, docs, refactor) for clear history
- **Conflict Resolution**: Resolve merge conflicts by understanding both versions and combining changes
- **Pull Requests**: Document changes with context, testing steps, and screenshots for reviewers
- **Environment Variables**: Store secrets in .env files using framework-specific prefixes (VITE_, REACT_APP_, etc.)
- **Hotfix Workflow**: Fast-track critical fixes with expedited review and deployment

**Key takeaway**: Git workflows enable team collaboration without stepping on each other's toes - commit often, communicate clearly, and keep secrets safe.

---

## Real-World Application

This week at work, you might use these concepts to:

- Create a feature branch for a new dashboard widget and submit a clean PR
- Resolve conflicts when two developers modify the same component
- Write commit messages that teammates can understand six months later
- Set up environment variables for API keys and feature flags
- Handle a production hotfix without disrupting ongoing feature work

---

## Further Reading

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Branching - Basic Branching and Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)

---

**Navigation**: [← Previous Module](./12-accessibility-fundamentals.md)
