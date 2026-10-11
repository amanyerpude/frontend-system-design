# Client-Side Security Hardening

> The browser is an untrusted runtime under constant attacker pressure. Client-side security isn't just about XSS — it's about trust boundaries, secure contexts, safe DOM sinks, and limiting what scripts (including your own and third-party) can do once they reach the browser.
>
> Sources: [URIports — Trusted Types, Defense in Depth](https://www.uriports.com/blog/csp-trusted-types/) | [MDN — Trusted Types API](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API) | [MDN — Secure Contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Secure_Contexts)

---

## The Browser Trust Model

The browser is **both a user interface and an untrusted execution environment**. You don't control:
- The user's browser
- Browser extensions
- Network conditions
- Injected scripts (XSS, compromised dependencies, CDNs)

**Key principle:** Treat everything in the browser as potentially attacker-accessible. Keep secrets out of the browser entirely — tokens, keys, and sensitive business logic belong on the server.

---

## Layer 1: Secure Contexts

HTTPS defines a **secure context**. Many powerful APIs are available **only** over HTTPS:
- `fetch`, `XMLHttpRequest` to HTTPS origins
- Service workers
- Payment Request API
- Web Authentication (WebAuthn)
- Geolocation, camera, microphone
- `window.crypto.subtle` (crypto operations)

Browsers show a **not-secure** warning for HTTP pages attempting these APIs. Always serve over HTTPS.

---

## Layer 2: Content Security Policy (CSP)

CSP restricts what a page can load and execute. A strict, nonce-based CSP is the most impactful XSS mitigation:

```http
Content-Security-Policy: default-src 'self';
  script-src 'nonce-{random}' 'strict-dynamic' https:;
  object-src 'none';
  base-uri 'self';
  frame-ancestors 'none';
  require-trusted-types-for 'script'
```

### Report-Only First
Deploy in report-only mode first to catch legitimate inline scripts:
```http
Content-Security-Policy-Report-Only: <your policy>; report-to default
Reporting-Endpoints: default="https://yourdomain.com/reports"
```

---

## Layer 3: Trusted Types (DOM-Based XSS Elimination)

DOM-based XSS lives in "injection sinks" — APIs that interpret strings as HTML/JS:
```javascript
element.innerHTML = data;        // sink — data parsed as HTML
eval(data);                      // sink — data executed as JS
scriptElement.src = url;         // sink — data is a script URL
```

Trusted Types **forces** these sinks to reject raw strings — they only accept typed objects (`TrustedHTML`, `TrustedScript`, `TrustedScriptURL`).

### Enable Trusted Types
```http
Content-Security-Policy: require-trusted-types-for 'script';
  trusted-types sanitize;  # allowlist policy names
```

### Define a Policy
```javascript
const policy = trustedTypes.createPolicy('sanitize', {
  createHTML: (input) => DOMPurify.sanitize(input),
  createScript: (input) => { throw new Error('Scripts not allowed'); },
  createScriptURL: (input) => {
    // Allow only known-safe script origins
    const url = new URL(input, location.origin);
    if (!['cdn.example.com', location.host].includes(url.host)) {
      throw new Error('Not allowed');
    }
    return input;
  }
});
```

### Use the Policy
```javascript
// BEFORE (unsafe — throws with Trusted Types enforced):
element.innerHTML = userInput;

// AFTER (safe — must go through policy):
element.innerHTML = policy.createHTML(userInput);
```

### The `default` Policy (Migration Aid)
```javascript
trustedTypes.createPolicy('default', {
  createHTML: (input) => {
    console.warn('Default policy hit — refactor this code');
    return DOMPurify.sanitize(input);
  }
});
```

The browser routes any raw string hitting a sink through the `default` policy. Use it to find all sinks that still need refactoring.

---

## Layer 4: Safe DOM Sinks

**Audit every line** that touches these dangerous APIs. The goal is to eliminate sinks entirely wherever possible.

### Safe Sinks (OK) — treat data as text
```javascript
element.textContent = data;           // Plain text, never parsed as HTML
element.insertAdjacentText("beforeend", data);
formfield.value = data;
document.createTextNode(data);
element.setAttribute("title", data);  // Fixed attribute name
```

### Dangerous Sinks (Avoid / Wrap)
```javascript
element.innerHTML = data;               // Use Trusted Types + DOMPurify
document.write(data);                   // Avoid entirely
eval(data);                             // Never
setInterval(data);                      // Never use string form
element.setAttribute("onclick", data);  // Never
```

### Framework Escape Hatches (Audit These)
| Framework | Safe Default | Unsafe Escape Hatch |
|-----------|-------------|-------------------|
| React | `{text}` | `dangerouslySetInnerHTML` |
| Vue | `{{ text }}` | `v-html` |
| Angular | `{{ text }}` | `bypassSecurityTrustHtml()` |
| Lit | `${text}` | `unsafeHTML()` |

Every use of an escape hatch must have sanitized input.

---

## Layer 5: Third-Party Script Isolation

Every external script runs with the **same privileges** as your own code.

### Sandbox Third-Party Widgets
```html
<iframe
  src="https://widget.com/embed"
  sandbox="allow-scripts allow-same-origin"
  allow="camera 'none'; microphone 'none'; payment 'none'">
</iframe>
```

### Isolate with COOP/COEP/CORP
- `Cross-Origin-Opener-Policy: same-origin` — isolate browsing context
- `Cross-Origin-Embedder-Policy: require-corp` — block cross-origin resources
- `Cross-Origin-Resource-Policy: same-site` — control resource loading

These enable powerful features (SharedArrayBuffer, cross-origin isolation) while hardening the sandbox.

---

## Layer 6: Security Headers Baseline

Apply at the CDN/server level, not per-route:

| Header | Value | Purpose |
|--------|-------|---------|
| `Content-Security-Policy` | Strict nonce-based | Primary XSS defense |
| `X-Frame-Options` | DENY | Legacy clickjacking defense (use `frame-ancestors` in CSP) |
| `X-Content-Type-Options` | nosniff | Block MIME sniffing |
| `Referrer-Policy` | strict-origin-when-cross-origin | Limit referrer leakage |
| `Strict-Transport-Security` | `max-age=63072000; includeSubDomains` | Force HTTPS |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` | Disable powerful features |
| `Cross-Origin-Opener-Policy` | same-origin | Context isolation |

---

## Browser Security Boundary Checklist

1. ✅ **HTTPS everywhere** — secure context for all powerful APIs
2. ✅ **Strict CSP** — nonce-based, `strict-dynamic`, report-only first
3. ✅ **Trusted Types enforced** — eliminate DOM XSS sinks
4. ✅ **SRI on all external scripts** — defense against CDN tampering
5. ✅ **SameSite cookies** — `Lax` or `Strict`
6. ✅ **HttpOnly session cookies** — never accessible from JS
7. ✅ **Permissions-Policy** — disable unused features (camera, mic, geolocation)
8. ✅ **X-Frame-Options / frame-ancestors** — prevent clickjacking
9. ✅ **Third-party scripts sandboxed** — iframed with restrictive `sandbox`
10. ✅ **No secrets in client code** — API keys, tokens, business logic stay server-side

---

## References
- [MDN — Trusted Types API](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API)
- [MDN — Secure Contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Secure_Contexts)
- [web.dev — Prevent DOM-based XSS with Trusted Types](https://web.dev/articles/trusted-types)
- [DOMPurify](https://github.com/cure53/DOMPurify)
- [Reporting API](https://developer.mozilla.org/en-US/docs/Web/API/Reporting_API)
