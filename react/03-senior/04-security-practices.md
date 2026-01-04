# Security Practices

Protect your applications and users from common vulnerabilities.

## Learning Objectives

By the end of this module, you will:
- Prevent XSS, CSRF, and injection attacks
- Implement secure authentication flows
- Configure Content Security Policy
- Handle sensitive data safely

## XSS Prevention

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

## CSRF Protection

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

## Content Security Policy

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

## Secure Authentication

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

## Sensitive Data Handling

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

## Security Headers Checklist

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

> Note: `X-XSS-Protection` is deprecated in modern browsers and should not be used. Modern CSP headers provide better protection.

## Dependency Security

```bash
# Audit dependencies
npm audit

# Fix vulnerabilities
npm audit fix

# Use Snyk or Dependabot for continuous monitoring
```

## Practice Exercises

1. Implement CSP for an existing application
2. Set up OAuth with PKCE flow
3. Audit and fix XSS vulnerabilities

## Further Reading

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [MDN Web Security](https://developer.mozilla.org/en-US/docs/Web/Security)
