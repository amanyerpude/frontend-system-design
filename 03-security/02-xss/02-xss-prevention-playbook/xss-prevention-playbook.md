# XSS Prevention Playbook: Defense in Depth Strategies

> A layered approach to XSS prevention. Every individual defense has documented bypasses — stack mechanisms so a bypass of one is caught by the next.
>
> Source: [SecureCodingHub](https://www.securecodinghub.com/blog/xss-prevention-defense-in-depth-developer-guide)

---

## Why XSS Prevention Requires Layered Defenses

No single defense is sufficient. XSS-resistant applications **stack mechanisms** so that each layer's failure modes rarely align. The eight layers:

1. **Context-Aware Output Encoding**
2. **Allowlist Input Validation**
3. **Content Security Policy (CSP)**
4. **HttpOnly + Secure + SameSite Cookies**
5. **Trusted Types API**
6. **Framework Defaults & Escape Hatch Audits**
7. **Sanitization Libraries**
8. **Subresource Integrity (SRI) for Third-Party Scripts**

---

## Layer 1 — Context-Aware Output Encoding

The foundation: encode untrusted data with the rules for its specific output context:

| Context | Encoding |
|---------|----------|
| HTML body text | HTML entity encoding |
| HTML attribute values | HTML attribute encoding |
| JavaScript string literals | JavaScript encoding |
| URL components | URL encoding |
| CSS values | CSS hex encoding |

**Use a context-aware encoding library** (OWASP Java Encoder, OWASP ESAPI, Go `html/template`) rather than a single global "escape function." A global HTML encoder applied to JavaScript context data is still vulnerable.

> Most apps rely on the framework's auto-encoding (Layer 6). Manual encoding is needed for building HTML strings outside templating, generating JSON without serializers, constructing URLs with user params, and creating emails/PDFs.

---

## Layer 2 — Allowlist Input Validation

Input validation is **defense-in-depth**, not the primary defense — encoding at the output sink is. It narrows attack surface and reduces blast radius.

**Pattern:** Type-coerce-then-validate. Parse request body, coerce each field to expected type (reject mismatches), validate against schema constraints (return 400 on mismatch).

Use schema validation libraries: **Zod, Joi, Yup, ajv** (TS/Node), **Pydantic** (Python), **Bean Validation** (Java), **FluentValidation** (.NET).

> **Never use denylists** ("block these bad characters"). Allowlists enumerate what's permitted and reject everything else. Denylists always have bypasses.

---

## Layer 3 — Content Security Policy (CSP)

CSP is the browser-enforced layer that limits what a page can execute **even if a payload reaches the HTML**. A correctly configured CSP turns script injection into "browser blocked it + logged a violation."

### Nonce-Based CSP (Modern Approach)

Host-based allowlists are **bypassable** (CDNs serve JSONP endpoints, Angular templates). Instead:

- Generate a **cryptographically random nonce per response**
- Include it in the CSP header
- Set the same nonce on every legitimate inline script

```javascript
// Express middleware: per-response nonce + CSP header
const crypto = require('crypto');
app.use((req, res, next) => {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.cspNonce = nonce;
  res.setHeader('Content-Security-Policy', [
    "default-src 'self'",
    `script-src 'nonce-${nonce}' 'strict-dynamic' https:`,
    "object-src 'none'",
    "base-uri 'self'",
    "frame-ancestors 'none'",
    "require-trusted-types-for 'script'"
  ].join('; '));
  next();
});
// Template: <script nonce="<%= cspNonce %>">...</script>
```

### Operational Discipline
1. Deploy in **Report-Only** mode (`Content-Security-Policy-Report-Only`)
2. Collect violation reports (`report-uri` / `report-to`)
3. Fix legitimate violations
4. Switch to **enforcement**

### Remaining Bypasses
- **Script gadgets**: pre-existing JS on the page that takes attacker input and executes it (mitigated by Trusted Types — Layer 5)
- **Dangling-markup attacks**: data exfiltration even when script is blocked (requires careful HTML structure)

---

## Layer 4 — HttpOnly + Secure + SameSite Cookies

The cookie-attribute layer **limits damage** when XSS lands.

- **`HttpOnly`**: blocks JavaScript access entirely. An XSS payload cannot read or exfiltrate the cookie. The session cookie survives XSS that reads everything else on the page.
- **`Secure`**: only sent over HTTPS
- **`SameSite=Lax` or `Strict`**: blocks cross-site cookie inclusion (CSRF protection)
- **`__Host-` prefix**: locks cookie to a single origin's host

> **Anti-pattern:** Storing the session token in localStorage or a non-HttpOnly cookie because "the SPA needs to read the token." This is exploited in ~30 lines of XSS code.

**Correct approach:** Keep the session in an `HttpOnly` cookie and let the browser attach it automatically (with CSRF defenses for state-changing requests).

---

## Layer 5 — Trusted Types API

Trusted Types mandates safe DOM sinks at the API level. Dangerous properties (`innerHTML`, `outerHTML`, `document.write`, `eval`, script `src`) accept **only special typed values** (`TrustedHTML`, `TrustedScript`, `TrustedScriptURL`) when enforced.

Enable via CSP: `require-trusted-types-for 'script'`

```javascript
if (window.trustedTypes && window.trustedTypes.createPolicy) {
  window.trustedTypes.createPolicy('default', {
    createHTML: (input) => DOMPurify.sanitize(input, { RETURN_TRUSTED_TYPE: true }),
    createScript: (input) => { throw new Error('Script string conversion not allowed'); },
    createScriptURL: (input) => {
      const url = new URL(input, location.origin);
      const allowedHosts = ['cdn.example.com', location.host];
      if (!allowedHosts.includes(url.host)) throw new Error('Not allowed');
      return input;
    }
  });
}
// CSP: require-trusted-types-for 'script'; trusted-types default;
```

> **Browser support** (2026): Good in Chromium (Chrome, Edge, Opera), not yet in Firefox/Safari. Treat as additional layer, not a sole defense.

---

## Layer 6 — Framework Defaults & Escape Hatches

Modern frameworks auto-escape by default:

| Framework | Safe Default | Unsafe Escape Hatch |
|-----------|-------------|-------------------|
| React | `{text}` in JSX | `dangerouslySetInnerHTML={{ __html: text }}` |
| Vue | `{{ text }}` | `v-html="text"` |
| Angular | `{{ text }}` | `bypassSecurityTrustHtml(text)` |
| Svelte | `{text}` | `__html` (rare) |

**Rule:** Every appearance of an escape hatch is a code-review trigger. The reviewer asks:
1. Where did the input come from?
2. What sanitization did it receive?
3. Does the use case justify bypassing the framework default?

The most common failure: using the escape hatch for Markdown rendering **without sanitization**.

---

## Layer 7 — Sanitization Libraries

For features rendering HTML with formatting (comments, rich-text editors, Markdown), use a **sanitization library with a strict allowlist**.

### DOMPurify (Browser / Node.js)
```javascript
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
  ALLOWED_ATTR: ['href'],
  ALLOWED_URI_REGEXP: /^(?:https?|mailto):/i,
  FORBID_TAGS: ['style', 'script', 'iframe'],
  FORBID_ATTR: ['onerror', 'onload', 'style']
});
```

### Python — Bleach
```python
import bleach
clean = bleach.clean(user_input,
  tags=['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
  attributes={'a': ['href']},
  protocols=['http', 'https', 'mailto'])
```

### Java — OWASP HTML Sanitizer
```java
PolicyFactory policy = Sanators.FORMATTING.and(Sanitizers.LINKS);
String safeHtml = policy.sanitize(userInput);
```

> **Failure mode:** Sanitization libraries fail when the library's parser doesn't match the browser's parser. **Mutation-XSS** exploits these mismatches. DOMPurify mitigates by running inside the browser DOM using the actual parser. Server-side sanitizers using their own parser are more exposed — keep current.

---

## Layer 8 — Subresource Integrity (SRI) for Third-Party Scripts

SRI protects against XSS coming through **compromised third-party scripts**. The `integrity` attribute contains a cryptographic hash of the expected content.

```html
<script
  src="https://cdn.example.com/widget.js"
  integrity="sha384-{base64hash}"
  crossorigin="anonymous">
</script>
```

The browser verifies the hash before executing. A CDN compromise → hash mismatch → browser blocks.

> **Tradeoff:** Hash must update every time the third-party script legitimately changes. Use SRI for version-pinned CDN libraries; for frequently-changing scripts (tag managers, analytics), rely on strict CSP and a content-review process for new tags.

---

## The XSS Prevention Checklist for Every Feature

Every pull request that adds user-facing rendering, accepts user input, or modifies an existing rendering path passes through:

1. **Where does user-controlled data appear?** List every input source (params, headers, cookies, file uploads, third-party API responses, database fields from other users) and trace each through code paths.
2. **What sinks does each value reach?** Map source-to-sink for every flow. (HTML body, HTML attribute, JavaScript string, URL, CSS, JSON, SQL, shell.)
3. **Framework default or escape hatch?** If escape hatch, input must be sanitized through a configured allowlist library first.
4. **Input validation at boundary?** Schema validation library with an **allowlist** schema. Type coercion before validation.
5. **Strict CSP deployed?** Nonce-based, `strict-dynamic`, no `unsafe-inline`, no `unsafe-eval`, `object-src 'none'`, `base-uri 'self'`, restrictive `frame-ancestors`.
6. **Cookies fully attributed?** HttpOnly, Secure, SameSite=Lax/Strict, `__Host-` prefix where applicable.
7. **Trusted Types enforced?** CSP `require-trusted-types-for 'script'`, named policies for legitimate sink consumers.
8. **Third-party scripts pinned + SRI'd?** Version-pinned URLs, integrity attribute, hash updated through code review.
9. **Tested with XSS payloads in CI?** SAST + DAST + representative payload corpus against staging.
10. **Rollback path?** Feature flag for the new rendering path, CSP violation monitoring, runbook for "we shipped XSS."

---

## Common Anti-Patterns

| Anti-Pattern | Why It Fails |
|-------------|-------------|
| Blacklist filters (`input.replace(/<script>/gi, '')`) | Encodings, case variations, parser quirks, event handlers (`onerror=`), SVG events, `data:` URLs all bypass it |
| Regex sanitization of HTML | HTML is not a regular language; always misses cases |
| Input-time encoding (encode at request parse) | Same string needs different encodings per sink; corrupts storage/search/comparison |
| "We use a WAF" | Bypassed through encoding variations, payload chunking; not effective against DOM-based XSS |
| "Our framework prevents XSS" | Escape hatches exist and get used — becoming the foothold |
| "We sanitize on input" | Sanitization is sink-agnostic; cannot know if data flows to HTML, JS, SQL, or shell |

---

## Closing: The Stack Is the Defense

The eight layers are **not alternatives** — they compose. Each layer has documented failure modes; the program's job is to ensure those failure modes rarely overlap.
