# Supply-Chain Security

> **Last reviewed**: 2026-01-06


## Week Seven: The Dependency You Didn't Choose

A transitive dependency ships malicious code. Your app isn't targeted, but you're exposed.

"Supply-chain security is about the code you never wrote," Sarah says.

Marcus adds, "If you rely on the ecosystem, you must guard the ecosystem."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Dependency graph** | A family tree - risk travels through branches |
| **Lockfile** | A ledger - exact versions you ship |
| **Integrity** | A seal - proof the package is unchanged |
| **Review** | A checkpoint - verify before you merge |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 06 (Security Practices) - Familiarity with common web security risks.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Use lockfiles to ensure deterministic installs
- [ ] Run dependency audits and interpret results
- [ ] Set policies for dependency updates
- [ ] Use integrity checks and provenance signals
- [ ] Review dependency changes in PRs
- [ ] Respond quickly to dependency incidents

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Build supply-chain habits over 4-6 weeks

---

## Chapter 1: The Dependency Graph

"Your app is the sum of its dependencies," Sarah says.

Use tooling to visualize it:

```bash
npm ls
```

"Know what you ship before users do."

---

## Chapter 2: Lockfiles Are Non-Negotiable

Lockfiles freeze versions.

```text
package-lock.json / pnpm-lock.yaml / yarn.lock
```

"Without a lockfile, you cannot reproduce a build."

---

## Chapter 3: Audits and Alerts

Run audits regularly.

```bash
npm audit
npm audit --omit=dev
```

"Treat high and critical issues as blockers," Marcus says.

---

## Chapter 4: Integrity and Provenance

Check package integrity and provenance signals.

"Integrity hashes and signed packages reduce tampering risk."

---

## Chapter 5: Dependency Review in PRs

Add a policy:

- Review dependency diffs
- Require approval for new packages
- Prefer minimal libraries

---

## Chapter 6: Incident Response

When a vuln hits:

1) Identify exposure
2) Patch or pin
3) Release
4) Communicate

"Speed matters. So does clarity."

---

## Common Mistakes

1. **Ignoring lockfiles** - Installs become non-deterministic.
2. **Blindly running audit fix** - It can break production.
3. **No review for new deps** - Risk sneaks in unnoticed.
4. **Delaying patches** - Exposure time grows.


Example:
```bash
# ❌ Install without lockfile
npm install

# ✅ Commit and enforce lockfile
git add package-lock.json
```

---

## Practice Exercises

1. Run an audit and classify the results by severity
2. Add a dependency review checklist to PRs
3. Write a response plan for a critical dependency CVE

### Solutions

<details>
<summary>Exercise 1: Audit Classification</summary>

```text
Critical: 0
High: 2
Moderate: 5
Low: 12
Action: Fix high first, schedule moderate.
```

**Key points:**
- Triage by severity.
- Fix critical/high quickly.
- Track remaining issues.

</details>

<details>
<summary>Exercise 2: PR Checklist</summary>

```text
Dependency Review:
- Why is this package needed?
- Is it maintained?
- License acceptable?
- Alternatives considered?
```

**Key points:**
- Make reviews explicit.
- Keep scope tight.
- Require approvals.

</details>

<details>
<summary>Exercise 3: Response Plan</summary>

```text
1) Identify impacted versions
2) Patch and test
3) Release hotfix
4) Notify stakeholders
```

**Key points:**
- Move quickly.
- Verify fixes.
- Communicate clearly.

</details>

---

## What You Learned

This module covered:

- **Lockfiles**: Deterministic installs
- **Audits**: Detect vulnerable dependencies
- **Integrity**: Reduce tampering risk
- **Review**: Gate new dependencies
- **Response**: Patch quickly when risks appear

**Key takeaway**: Supply-chain security is risk management for code you didn't write.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add dependency review steps to PRs
- Patch a critical vulnerability quickly
- Enforce lockfile usage in CI
- Reduce unused dependencies in the bundle
- Create a dependency update policy

---

## Further Reading

- [npm audit](https://docs.npmjs.com/cli/v10/commands/npm-audit)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
- [SLSA Supply Chain Levels](https://slsa.dev/)

---

**Navigation**: [← Previous Module](./06-security-practices.md) | [Next Module →](./08-accessibility-a11y.md)
