# Git Workflows

Collaborate effectively with your team using Git best practices.

## Learning Objectives

By the end of this module, you will:
- Use branching strategies for feature development
- Write meaningful commit messages
- Create quality pull requests
- Resolve merge conflicts confidently

## Branching Strategy

### Feature Branch Workflow

```bash
# Start from updated main
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/user-authentication

# Work on feature, commit regularly
git add .
git commit -m "feat: add login form component"

# Push and create PR
git push -u origin feature/user-authentication
```

**Branch naming conventions:**
- `feature/description` - New features
- `fix/description` - Bug fixes
- `refactor/description` - Code refactoring
- `docs/description` - Documentation updates

### Keeping Branch Updated

```bash
# Option 1: Merge main into your branch
git checkout feature/my-feature
git merge main

# Option 2: Rebase onto main (cleaner history)
git checkout feature/my-feature
git rebase main

# If conflicts, resolve then continue
git add .
git rebase --continue
```

## Commit Messages

Follow Conventional Commits for consistency.

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

**Examples:**

```bash
# Good commits
git commit -m "feat(auth): add password reset functionality"
git commit -m "fix(cart): prevent negative quantities"
git commit -m "refactor(api): extract fetch logic to custom hook"

# Bad commits
git commit -m "fix stuff"
git commit -m "WIP"
git commit -m "addressed PR comments"
```

**Commit often, but make each commit meaningful:**

```bash
# Before committing, review changes
git diff
git diff --staged

# Stage specific files/hunks
git add -p  # Interactive staging
```

## Pull Requests

### Creating Quality PRs

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

### PR Best Practices

1. **Keep PRs small** - 200-400 lines ideal
2. **One concern per PR** - Don't mix features
3. **Self-review first** - Catch obvious issues
4. **Respond to feedback** - Discuss, don't defend
5. **Update with latest main** - Before merging

### Reviewing PRs

```markdown
# Good review comments
"Consider using useMemo here since this calculation is expensive"
"This could throw if user is null - should we add a guard?"

# Constructive, not critical
Instead of: "This is wrong"
Say: "This might cause issues when X happens. What about Y?"
```

## Resolving Conflicts

```bash
# When merge conflict occurs
git status  # See conflicting files

# Open file and resolve
<<<<<<< HEAD
const greeting = "Hello";
=======
const greeting = "Hi there";
>>>>>>> feature/new-greeting

# Choose one, or combine:
const greeting = "Hello there";

# Mark resolved and continue
git add .
git commit -m "resolve merge conflict in greeting"
```

**Conflict prevention:**
- Pull main frequently
- Communicate with team about overlapping work
- Keep PRs small and merge often

## Common Git Commands

```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Discard local changes to a file
git checkout -- filename

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
```

## Git Hooks

Automate checks before commits.

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

## Practice Exercises

1. Practice the feature branch workflow on a sample repo
2. Write 5 commits following Conventional Commits
3. Create a merge conflict and resolve it

## Further Reading

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Branching - Basic Branching and Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
