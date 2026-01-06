# Privacy and PII Logging

> **Last reviewed**: 2026-01-06


## Week Twenty: The Support Ticket with Secrets

A support ticket arrives with a screenshot of a log line. It includes a full address, phone number, and order notes. The user never agreed to share any of it.

"Logs are part of your product," Sarah says. "Treat them like production data."

Marcus adds, "If you would not print it on a billboard, do not log it."

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **PII** | Toxic waste - handle with care and limit exposure |
| **Data minimization** | Packing light - only take what you need |
| **Redaction** | A black marker - hide sensitive details |
| **Retention** | An expiry date - delete when no longer needed |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Module 06 (Security Practices) - Familiarity with secure data handling.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Classify data and map where it flows
- [ ] Design logging policies that avoid sensitive data
- [ ] Implement client-side redaction and allowlists
- [ ] Configure error reporting to scrub PII
- [ ] Set retention rules and access boundaries
- [ ] Build a privacy-first incident response plan

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Apply privacy discipline over 6-8 weeks

---

## Chapter 1: Classify Data and Flows

Before you log anything, decide what it is.

Create a simple data map:

- **PII**: names, email, phone, address
- **Sensitive**: auth tokens, payment details, health data
- **Operational**: request IDs, feature flags, timing metrics

"If the data is personal, treat it as high risk," Sarah says.

---

## Chapter 2: Log by Allowlist

Avoid logging entire objects. Log only what you approve.

```ts
const allowedKeys = new Set(['event', 'route', 'status', 'requestId']);

function safeLog(payload: Record<string, unknown>) {
  const scrubbed: Record<string, unknown> = {};
  for (const [key, value] of Object.entries(payload)) {
    if (allowedKeys.has(key)) scrubbed[key] = value;
  }
  logger.info(scrubbed);
}
```

"Allowlist beats blacklist," Marcus says. "You will miss fields otherwise."

---

## Chapter 3: Consent and Analytics Boundaries

Analytics should respect consent and collect only what you need.

- Gate analytics behind a consent flag
- Avoid user-entered text and form fields
- Use anonymized identifiers when possible

"If the user opted out, your code should not emit events," Sarah says.

---

## Chapter 4: Scrub Error Reports

Error reporting is valuable, but stack traces can leak data. Scrub before sending.

```ts
const errorClient = createErrorClient({
  beforeSend: event => {
    event.request.url = redactUrl(event.request.url);
    event.request.headers = redactHeaders(event.request.headers);
    event.extra = redactEvent(event.extra);
    return event;
  },
});
```

"Privacy does not mean blind. It means intentional," Marcus says.

---

## Chapter 5: Retention and Access

Keep logs only as long as needed.

- Define retention windows by data class
- Restrict access to logs by role
- Encrypt at rest and in transit
- Document who can export data

"If no one needs it, delete it," Sarah says.

---

## Chapter 6: Privacy Incident Playbook

You will eventually discover a leak. Plan for it now.

- Rotate secrets and revoke tokens
- Remove leaked data from storage
- Notify stakeholders and legal teams
- Add tests to prevent regressions

"Fast response turns a breach into a lesson," Marcus says.

---

## Common Mistakes

1. **Logging entire user objects** - includes hidden PII.
2. **Putting PII in URLs** - query strings leak to logs and analytics.
3. **Skipping consent checks** - analytics fires even after opt-out.
4. **Unlimited retention** - old data becomes liability.

Example:

```ts
// ❌ logging entire payload
logger.info({ event: 'checkout', payload: formValues });

// ✅ log only approved fields
logger.info({ event: 'checkout', plan: formValues.planId, requestId });
```

---

## Practice Exercises

1. Create a data classification table for a checkout flow.
2. Add a `redact` utility that removes emails and phone numbers.
3. Update an error reporter configuration to strip query params.

### Solutions

<details>
<summary>Exercise 1: Data Classification</summary>

```txt
PII: name, email, phone
Sensitive: payment token
Operational: requestId, status
```

**Key points:**
- Classify by risk, not convenience.
- Document where each field flows.
- Prefer the least sensitive category possible.

</details>

<details>
<summary>Exercise 2: Redaction</summary>

```ts
function redact(value: string) {
  return value.replace(/\b\S+@\S+\b/g, '[email]')
    .replace(/\b\d{3}[- ]?\d{3}[- ]?\d{4}\b/g, '[phone]');
}
```

**Key points:**
- Redact before sending data out.
- Use allowlists for known safe fields.
- Add tests for common formats.

</details>

<details>
<summary>Exercise 3: Strip Query Params</summary>

```ts
function redactUrl(url: string) {
  return url.split('?')[0];
}
```

**Key points:**
- Query params often include identifiers.
- Strip them before logging.
- Preserve path for debugging.

</details>

---

## What You Learned

This module covered:

- **Data classification**: Know what you are handling
- **Allowlist logging**: Log only what you approve
- **Consent-aware analytics**: Respect user choices
- **Error scrubbing**: Remove PII from reports
- **Retention rules**: Delete data on schedule

**Key takeaway**: Privacy discipline is a product quality feature, not a legal chore.

---

## Real-World Application

This week at work, you might use these concepts to:

- Remove PII from client logs
- Add consent gating to analytics
- Define retention windows for error reports
- Build a redaction utility for URLs and headers
- Run a privacy review before a launch

---

## Further Reading

- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [NIST Privacy Framework](https://www.nist.gov/privacy-framework)
- [GDPR Article 5 Principles](https://gdpr-info.eu/art-5-gdpr/)

---

**Navigation**: [← Previous Module](./19-web-workers-off-main-thread.md) | [Next Module →](./21-subresource-integrity-third-party-scripts.md)
