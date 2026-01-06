# Deployment and CI

> **Last reviewed**: 2026-01-06


## Week Fifteen: The Release That Went Sideways

The team ships a new release on Friday. By Monday morning, users report a blank screen. The build passed locally, but production behaved differently.

"Deployment is part of the product," Sarah says. "If the release fails, the feature fails."

Marcus adds, "A reliable pipeline turns releases into routine instead of rituals."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **CI pipeline** | Assembly line - each stage checks quality |
| **Artifact** | Sealed package - the exact build you ship |
| **Environment** | Different stages - dev, staging, production |
| **Rollback** | Rewind button - restore the last known good version |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 11 (Code Quality) - Familiarity with linting and testing tools.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Understand the difference between dev, staging, and production
- [ ] Build and store artifacts for consistent releases
- [ ] Create a CI pipeline for linting, tests, and builds
- [ ] Manage environment variables safely in deployments
- [ ] Use release strategies like canary and feature flags
- [ ] Plan rollbacks and verify health after release

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 4-5 hours
- **Mastery**: Practice release workflows over 4-6 weeks

---

## Chapter 1: Environments Are Different

The app behaves differently in production. Why?

"Different environments have different data, configs, and constraints," Sarah says.

```text
Dev: fast, local, mock data
Staging: production-like, shared testing
Prod: real users, real stakes
```

---

## Chapter 2: Build Artifacts

"Build once, deploy many," Marcus says.

```bash
npm run build
# dist/ is your artifact
```

"If you build on every server, you risk inconsistent output."

---

## Chapter 3: A Simple CI Pipeline

You wire lint, tests, and build together.

```yaml
name: CI
on: [pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

"If CI fails, the change doesn't ship."

---

## Chapter 4: Environment Variables in Deployments

Local `.env.local` doesn't exist in production.

"Use platform env settings," Sarah says.

```text
VITE_API_URL=https://api.example.com
VITE_FEATURE_FLAG_NEW_CHECKOUT=true
```

"Never hardcode production keys in the repo."

---

## Chapter 5: Release Strategies

You can reduce risk with gradual rollouts.

- **Canary**: Release to a small percentage of users
- **Feature flags**: Hide new features behind toggles
- **Dark launch**: Ship code without exposing UI

"Small steps make failures smaller," Marcus says.

---

## Chapter 6: Rollbacks and Health Checks

When things go wrong, revert fast.

```text
Rollback steps:
1) Re-deploy last known good artifact
2) Verify health checks
3) Monitor error rates and logs
```

"Speed matters. A rollback is a feature."

---

## Common Mistakes

1. **Building in production** - Builds should be repeatable artifacts.
2. **Skipping CI checks** - Lint and tests are your safety net.
3. **Hardcoding env values** - Use platform secrets.
4. **No rollback plan** - You need a fast escape hatch.


Example:
```bash
# ❌ Build on every server
ssh prod "npm run build"

# ✅ Build once, deploy artifact
npm run build
rsync -r dist/ prod:/var/www/app
```

---

## Practice Exercises

1. Create a CI pipeline that runs lint, test, and build
2. Document deployment env vars in a README section
3. Draft a rollback checklist for a release

### Solutions

<details>
<summary>Exercise 1: CI Pipeline</summary>

```yaml
name: CI
on: [pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

**Key points:**
- Fail fast on lint/test.
- Build after quality checks.
- Keep CI deterministic with `npm ci`.

</details>

<details>
<summary>Exercise 2: Env Var Docs</summary>

```markdown
## Environment Variables

- `VITE_API_URL` - Base API URL for the frontend
- `VITE_FEATURE_FLAG_NEW_CHECKOUT` - Enables new checkout flow
```

**Key points:**
- Document purpose, not values.
- Keep secrets out of docs.
- Ensure staging and prod have matching vars.

</details>

<details>
<summary>Exercise 3: Rollback Checklist</summary>

```text
Rollback Checklist:
1) Re-deploy last known good artifact
2) Confirm health check endpoint returns 200
3) Verify error rate back to baseline
4) Communicate rollback in release channel
```

**Key points:**
- Rollbacks should be quick and scripted.
- Verify the system after rollback.
- Communicate clearly to stakeholders.

</details>

---

## What You Learned

This module covered:

- **Environments**: Dev, staging, and prod differ by design
- **Artifacts**: Build once for consistent deploys
- **CI**: Automate lint, test, and build checks
- **Secrets**: Store env vars safely in platforms
- **Release strategy**: Canary and feature flags reduce risk
- **Rollback**: Fast recovery protects users

**Key takeaway**: Reliable releases are the result of disciplined pipelines.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add a CI job that blocks broken builds
- Document required env vars for new services
- Roll out a feature behind a flag
- Plan a rollback before a risky release
- Validate a staging build before production

---

## Further Reading

- [GitHub Actions Docs](https://docs.github.com/actions)
- [Vite Deployment Guide](https://vitejs.dev/guide/static-deploy.html)
- [Feature Flags Explained](https://martinfowler.com/articles/feature-toggles.html)

---

**Navigation**: [← Previous Module](./14-animations-and-motion.md) | [Next Module →](./16-react-router-data-apis.md)
