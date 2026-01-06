# Subresource Integrity and Third-Party Scripts

> **Last reviewed**: 2026-01-06


## Week Twenty-One: The Compromised CDN

An analytics vendor gets compromised. The script on your site changes overnight. Customers report strange redirects and support is flooded.

"Third-party scripts are guests in your house," Sarah says. "You must lock the doors."

Marcus adds, "Trust is earned, but verification is your job."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Third-party script** | A guest - helpful, but still untrusted |
| **SRI hash** | A tamper seal - verify content before use |
| **CSP** | A guest list - only allow known sources |
| **Sandboxed iframe** | A meeting room - isolated from the main house |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Senior Module 07 (Supply-Chain Security) - Familiarity with dependency and vendor risk.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Inventory and assess third-party scripts
- [ ] Add Subresource Integrity (SRI) to external assets
- [ ] Configure `crossorigin` and caching safely
- [ ] Use CSP to restrict script sources
- [ ] Define a workflow for updating SRI hashes
- [ ] Isolate untrusted scripts when needed

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Apply SRI and CSP changes over 4-6 weeks

---

## Chapter 1: Inventory Third-Party Scripts

You cannot protect what you do not track.

Sarah says, "If a script has no owner, remove it."

Create a simple table:

- **Owner** (team or vendor)
- **Purpose** (analytics, payments, chat)
- **Impact** (what breaks if removed)
- **Risk** (data access, DOM access, user impact)

"If a script has no owner, remove it," Sarah says.

---

## Chapter 2: Add SRI Hashes

SRI lets the browser verify the exact content of a script.

Marcus says, "If the file changes, the browser should refuse to run it."

```html
<script
  src="https://cdn.example.com/analytics.js"
  integrity="sha384-Base64HashHere"
  crossorigin="anonymous"
></script>
```

"If the file changes, the browser refuses to run it," Marcus says.

---

## Chapter 3: CSP as a Guardrail

Use Content Security Policy to limit where scripts can load from.

Sarah says, "CSP is the seatbelt. SRI is the airbag."

```http
Content-Security-Policy: script-src 'self' https://cdn.example.com; object-src 'none'
```

"CSP is the seatbelt. SRI is the airbag," Sarah says.

---

## Chapter 4: Load Only What You Need

Load third-party scripts intentionally.

Marcus says, "Tag managers are powerful, but they bypass review."

- Prefer `defer` or `async`
- Gate marketing scripts behind consent
- Avoid dynamic script injection from unknown sources

"Tag managers are powerful. They also bypass your review process," Marcus says.

---

## Chapter 5: Update Workflow

SRI hashes must change when the script changes.

Sarah says, "Make hash updates part of release hygiene."

- Track script versions in code review
- Update hash when vendor releases a new build
- Use a staging check to verify hashes

"Make hash updates part of release hygiene," Sarah says.

---

## Chapter 6: Isolate Untrusted Scripts

If a script needs access but you do not trust it, sandbox it.

Marcus says, "Isolation turns breaches into contained incidents."

- Use sandboxed iframes for ads or widgets
- Communicate via `postMessage`
- Avoid granting top-level DOM access

"Isolation turns a breach into a contained issue," Marcus says.

---

## Common Mistakes

1. **Using SRI with dynamic URLs** - hashes break when content changes.
2. **Skipping `crossorigin`** - browsers may refuse the integrity check.
3. **Overly permissive CSP** - `*` and `unsafe-inline` defeat the purpose.
4. **No update plan** - hashes drift and scripts stop working.

Example:

```html
<!-- ❌ missing integrity and CSP allows any script -->
<script src="https://cdn.example.com/analytics.js"></script>

<!-- ✅ locked to exact content and source -->
<script
  src="https://cdn.example.com/analytics.js"
  integrity="sha384-Base64HashHere"
  crossorigin="anonymous"
></script>
```

---

## Practice Exercises

1. Add SRI to a CDN-hosted script in a sample page.
2. Draft a CSP header that allows only approved analytics and payment scripts.
3. Propose a review checklist for adding new third-party scripts.

### Solutions

<details>
<summary>Exercise 1: SRI Tag</summary>

```html
<script
  src="https://cdn.example.com/widget.js"
  integrity="sha384-Base64HashHere"
  crossorigin="anonymous"
></script>
```

**Key points:**
- Use sha384 or stronger.
- Update hashes on version change.
- Keep script URLs stable.

</details>

<details>
<summary>Exercise 2: CSP Draft</summary>

```http
Content-Security-Policy: script-src 'self' https://cdn.example.com https://payments.example.com; object-src 'none'
```

**Key points:**
- Start strict and add only required sources.
- Avoid `unsafe-inline` when possible.
- Test in report-only mode first.

</details>

<details>
<summary>Exercise 3: Script Review Checklist</summary>

```txt
- Business owner and purpose
- Data accessed and sent
- CSP entry and SRI hash
- Consent requirements
- Rollback plan
```

**Key points:**
- Tie every script to an owner.
- Document data access and risk.
- Make removal easy.

</details>

---

## What You Learned

This module covered:

- **Script inventory**: Know what runs on your pages
- **SRI**: Verify exact script contents
- **CSP**: Restrict script sources
- **Update workflow**: Keep hashes current
- **Isolation**: Contain risky scripts

**Key takeaway**: Third-party scripts are part of your security perimeter.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add SRI to vendor scripts in production
- Tighten CSP headers without breaking critical flows
- Remove unused marketing tags
- Build a review checklist for new third-party scripts
- Contain risky widgets in iframes

---

## Further Reading

- [MDN: Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)
- [MDN: Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [W3C: Subresource Integrity](https://www.w3.org/TR/SRI/)

---

**Navigation**: [← Previous Module](./20-privacy-and-pii-logging.md)
