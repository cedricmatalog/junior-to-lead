# Security Practices

> **Last reviewed**: 2026-01-06


## Week Six: The Security Audit

Security just flagged a set of vulnerabilities in the frontend. Sarah needs a remediation plan, and Marcus reminds you that frontend security is not just backend responsibility. This week is about protecting users by default: defending against XSS, enforcing CSP, securing authentication flows, and handling sensitive data safely.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **XSS prevention** | A filter - remove dangerous input before it spreads |
| **CSRF protection** | A wristband - prove who is allowed in |
| **CSP** | A whitelist - only trusted sources allowed |
| **Auth security** | A lockbox - protect credentials and tokens |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Senior Module 04 (Advanced Performance) - Familiarity with runtime environments and deployment.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Prevent XSS and injection vulnerabilities
- [ ] Implement CSRF protection in client flows
- [ ] Configure Content Security Policy safely
- [ ] Secure authentication and token storage
- [ ] Handle sensitive data without leaks
- [ ] Maintain a dependency security checklist

---

## Time Estimate

- **Reading**: 70-90 minutes
- **Exercises**: 3-5 hours
- **Mastery**: Practice secure patterns over 6-8 weeks

---

## Chapter 1: XSS Prevention

You need to protect every surface that touches user-controlled content.

"Assume input is hostile until proven safe," Sarah says.

### React's Built-in Protection

```tsx
// React escapes content by default - safe
const userInput = '<script>alert("xss")</script>';
return <div>{userInput}</div>;
// Renders as text, not executed

// DANGER: dangerouslySetInnerHTML bypasses protection
// Only use with sanitized content
return <div dangerouslySetInnerHTML={{ __html: sanitizedHTML }} />;
```

### Sanitization

```tsx
import DOMPurify from 'dompurify';

function RichContent({ html }: { html: string }) {
  const sanitized = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p'],
    ALLOWED_ATTR: ['href'],
  });

  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}
```

### URL Validation

```tsx
function SafeLink({ href, children }: { href: string; children: React.ReactNode }) {
  // Prevent javascript: URLs
  const isUnsafe = href.toLowerCase().startsWith('javascript:');

  if (isUnsafe) {
    console.warn('Blocked unsafe URL:', href);
    return <span>{children}</span>;
  }

  return <a href={href}>{children}</a>;
}

// Validate URLs
function isValidUrl(url: string): boolean {
  try {
    const parsed = new URL(url);
    return ['http:', 'https:'].includes(parsed.protocol);
  } catch {
    return false;
  }
}
```

## Chapter 2: CSRF Protection

State-changing requests must prove the request came from your app.

"If a cookie is sent, it can be abused," Marcus says.

### Token-Based Protection

```tsx
// Get CSRF token from meta tag or cookie
function getCsrfToken(): string {
  return document.querySelector('meta[name="csrf-token"]')?.getAttribute('content') || '';
}

// Include in API requests
async function secureRequest(url: string, options: RequestInit = {}) {
  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'X-CSRF-Token': getCsrfToken(),
    },
    credentials: 'include', // Send cookies
  });
}
```

### SameSite Cookies

```tsx
// Backend should set cookies with SameSite
// Set-Cookie: session=abc123; SameSite=Strict; Secure; HttpOnly

// For authentication, prefer HttpOnly cookies over localStorage
// JWT in HttpOnly cookie > JWT in localStorage
```

## Chapter 3: Content Security Policy

CSP is your front line against injected scripts.

"CSP is your seatbelt," Sarah says. "Use it."

### Basic CSP

```html
<!-- Meta tag approach -->
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self';
  script-src 'self' 'unsafe-inline' https://cdn.example.com;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  connect-src 'self' https://api.example.com;
  font-src 'self' https://fonts.gstatic.com;
">
```

### Nonce-Based CSP

```tsx
// Server generates unique nonce per request
// Passed to app via header or meta tag

// In component
<script nonce={nonce}>
  // Inline script with nonce is allowed
</script>

// CSP header
// Content-Security-Policy: script-src 'nonce-abc123'
```

### Report-Only Mode

```
Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp-report
```

## Chapter 4: Secure Authentication

Token handling should reduce damage when mistakes happen.

"Treat auth as a flow, not a checkbox," Marcus says.

### OAuth/OIDC Flow

```tsx
// Using PKCE for SPA
async function startAuth() {
  const codeVerifier = generateRandomString(64);
  const codeChallenge = await sha256(codeVerifier);

  sessionStorage.setItem('code_verifier', codeVerifier);

  const authUrl = new URL('https://auth.example.com/authorize');
  authUrl.searchParams.set('client_id', CLIENT_ID);
  authUrl.searchParams.set('redirect_uri', REDIRECT_URI);
  authUrl.searchParams.set('response_type', 'code');
  authUrl.searchParams.set('code_challenge', codeChallenge);
  authUrl.searchParams.set('code_challenge_method', 'S256');

  window.location.href = authUrl.toString();
}

// Callback handler
async function handleCallback(code: string) {
  const codeVerifier = sessionStorage.getItem('code_verifier');

  const response = await fetch('/api/auth/token', {
    method: 'POST',
    body: JSON.stringify({ code, code_verifier: codeVerifier }),
  });

  const { access_token } = await response.json();
  // Store token securely (preferably in HttpOnly cookie via backend)
}
```

### Secure Token Storage

```tsx
// Prefer: Backend stores token in HttpOnly cookie
// OK: sessionStorage (cleared on tab close)
// Avoid: localStorage (persists, XSS vulnerable)

// If using in-memory storage
class TokenStore {
  private token: string | null = null;

  setToken(token: string) {
    this.token = token;
  }

  getToken() {
    return this.token;
  }

  clear() {
    this.token = null;
  }
}
```

## Chapter 5: Sensitive Data Handling

If data should not be exposed, the UI must treat it as radioactive.

"If it can hurt the user, treat it as toxic," Sarah says.

### Input Masking

```tsx
function CreditCardInput() {
  const [value, setValue] = useState('');

  return (
    <input
      type="text"
      inputMode="numeric"
      autoComplete="cc-number"
      value={formatCardNumber(value)}
      onChange={e => setValue(e.target.value.replace(/\D/g, ''))}
    />
  );
}

// Don't log sensitive data
function handlePayment(cardNumber: string) {
  console.log('Processing payment...'); // OK
  console.log('Card:', cardNumber); // NEVER
}
```

### Secure Forms

```html
<!-- Prevent browser caching sensitive data -->
<form autocomplete="off">
  <input
    type="password"
    autocomplete="new-password"
    name="password"
  />
</form>
```

## Chapter 6: Security Headers Checklist

Headers are inexpensive protection that many teams forget.

"Headers are cheap insurance," Marcus says.

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

> Note: `X-XSS-Protection` is deprecated in modern browsers and should not be used. Modern CSP headers provide better protection.

## Chapter 7: Dependency Security

Your dependency tree is part of your attack surface.

"Every dependency is a trust decision," Sarah says.

```bash
# Audit dependencies
npm audit

# Fix vulnerabilities
npm audit fix

# Use Snyk or Dependabot for continuous monitoring
```

---

## Common Mistakes

1. **Trusting user input** - Always sanitize and validate before rendering.
2. **Storing tokens in localStorage** - It increases XSS blast radius.
3. **Missing CSP** - Without a policy, injected scripts are harder to stop.
4. **Ignoring dependency audits** - Vulnerabilities in libraries still affect your app.


Example:
```jsx
// ❌ Render unsanitized HTML
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ Sanitize before rendering
<div dangerouslySetInnerHTML={{ __html: sanitize(userInput) }} />
```

## Practice Exercises

1. Implement CSP for an existing application
2. Set up OAuth with PKCE flow
3. Audit and fix XSS vulnerabilities

### Solutions

<details>
<summary>Exercise 1: CSP Header</summary>

```http
Content-Security-Policy: default-src 'self'; base-uri 'self'; script-src 'self' https://cdn.example.com; object-src 'none'; frame-ancestors 'none'; report-uri /csp-report
```

**Key points:**
- Start with `default-src 'self'`.
- Add only trusted CDNs.
- Disable risky sources like `object-src`.

</details>

<details>
<summary>Exercise 2: OAuth PKCE</summary>

```ts
// Generate PKCE verifier/challenge (S256)
function base64UrlEncode(buffer: ArrayBuffer) {
  return btoa(String.fromCharCode(...new Uint8Array(buffer)))
    .replace(/=/g, '')
    .replace(/\+/g, '-')
    .replace(/\//g, '_');
}

const verifier = crypto.randomUUID().replace(/-/g, '');
const digest = await crypto.subtle.digest('SHA-256', new TextEncoder().encode(verifier));
const challenge = base64UrlEncode(digest);

// Store verifier in memory or session storage
sessionStorage.setItem('pkce_verifier', verifier);

// Send to auth server with challenge
const url = `${AUTH_URL}?response_type=code&code_challenge=${challenge}&code_challenge_method=S256`;
window.location.assign(url);
```

**Key points:**
- PKCE prevents intercepted codes from being reused.
- Verifier stays on the client.
- Server validates challenge before issuing tokens.

</details>

<details>
<summary>Exercise 3: XSS Audit</summary>

```tsx
import DOMPurify from 'dompurify';

function Comment({ html }: { html: string }) {
  const sanitized = DOMPurify.sanitize(html);
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}
```

**Key points:**
- Sanitize before using `dangerouslySetInnerHTML`.
- Restrict allowed tags and attributes as needed.
- Avoid rendering raw HTML when possible.

</details>

---

## What You Learned

This module covered:

- **XSS defense**: Sanitization and safe rendering
- **CSRF protection**: Safeguards for state-changing requests
- **CSP**: Enforcing trusted sources only
- **Auth security**: Safer token handling
- **Sensitive data**: Preventing leaks in UI and logs

**Key takeaway**: Secure defaults prevent small mistakes from becoming incidents.

---

## Real-World Application

This week at work, you might use these concepts to:

- Add CSP headers to production responses
- Audit all uses of `dangerouslySetInnerHTML`
- Move tokens out of localStorage
- Tighten security headers in your CDN
- Set up automated dependency vulnerability alerts

---

## Further Reading

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [MDN Web Security](https://developer.mozilla.org/en-US/docs/Web/Security)

---

**Navigation**: [← Previous Module](./05-performance-budgets.md) | [Next Module →](./07-supply-chain-security.md)
