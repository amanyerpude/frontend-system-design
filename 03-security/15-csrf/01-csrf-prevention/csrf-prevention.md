# Cross-Site Request Forgery (CSRF) Prevention

> CSRF tricks an authenticated user's browser into performing an unwanted action on a trusted site. Since browsers auto-include cookies on cross-origin requests, an attacker can forge actions (fund transfers, password changes, etc.) if no CSRF defense exists.
>
> Source: [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

---

## How CSRF Works

1. Victim is logged into `bank.com` (session cookie stored)
2. Victim visits `evil.com` (while still authenticated to the bank)
3. `evil.com` contains a form/script that triggers a POST to `bank.com/transfer`
4. Browser **automatically includes** the session cookie
5. The transfer executes — victim never clicked/approved anything

The attacker can't read the response (Same-Origin Policy blocks it), but the **action still executes**.

---

## Defense Strategies (Pick Based on App Type)

### Modern Apps: Fetch Metadata Headers
The `Sec-Fetch-Site` header tells the server whether a request is `same-origin`, `same-site`, `cross-site`, or `none`. Block cross-site state-changing requests:

```javascript
const SAFE_METHODS = new Set(['GET', 'HEAD', 'OPTIONS']);
const site = req.get('Sec-Fetch-Site');

if (site === 'cross-site' && !SAFE_METHODS.has(req.method)) {
  return false; // Block cross-site POST/PUT/DELETE
}
```

- ✅ **~98% browser coverage** (Chrome, Edge, Firefox, Safari 16.4+)
- ✅ No client-side changes required
- ❗ Requires **fallback** for browsers that don't send the header

### Stateful Apps: Synchronizer Token Pattern
1. Server generates a **random, cryptographically-secure CSRF token**
2. Stores it server-side (in the user's session)
3. Sends it to the client (hidden form field or `<meta>` tag)
4. Client sends it back as a **custom header** on all state-changing requests
5. Server compares submitted token against session-stored token

```html
<form action="/transfer" method="post">
  <input type="hidden" name="CSRFToken"
         value="OWY4NmQwODE4ODRjN2Q2NTlhMmZl...">
  <!-- form fields -->
  <button type="submit">Transfer</button>
</form>
```

```javascript
// Client: read token, attach as custom header
const token = document.querySelector('meta[name="csrf-token"]').getAttribute('content');
fetch('/transfer', {
  method: 'POST',
  headers: { 'X-CSRF-Token': token },
  body: new FormData(form)
});
```

> ✅ **Preferred** — use the header approach over hidden form fields; CORS preflight blocks cross-origin custom headers.

### Stateless Apps: Signed Double-Submit Cookie (Recommended)
Stateless alternative that doesn't require server-side token storage:

```javascript
// Token = HMAC(SHA256, secret, sessionID + randomValue) + "." + randomValue
// Send as HttpOnly? No — cookie must be readable by client JS to echo back as header.
// Cookie name: csrf_token (no HttpOnly)
// Auth cookie: __Host-sessionid (HttpOnly, Secure, SameSite=Strict)
```

Validation:
1. Split token into `hmac` and `randomValue`
2. Recompute expected HMAC with current session + random value
3. **Constant-time compare** the HMACs

> ❗ **Avoid the naive double-submit** (random value in cookie equals the param) — vulnerable to cookie injection on uncontrolled subdomains.

### AJAX/API Apps: Custom Request Header
Since browsers enforce CORS preflight for custom headers, adding a custom header (like `X-CSRF-Token`) means cross-origin requests must first get server permission via CORS:

```javascript
// Client
fetch('/api/update', {
  method: 'POST',
  headers: { 'X-CSRF-Token': CSRF_TOKEN }
});
```

```javascript
// Server: reject if header missing
if (!req.get('X-CSRF-Token')) return res.sendStatus(403);
```

---

## Defense in Depth: SameSite Cookies

`SameSite` controls whether cookies are sent on cross-site requests:

| Value | Behavior |
|-------|----------|
| `Strict` | Never sent on cross-site requests. Breaks "arrive from external link" flows. Best for banks. |
| `Lax` | Sent on top-level GET navigation only. Good default for most session cookies (Chrome/Firefox default since 2020). |
| `None` | Must pair with `Secure`. Required for legitimate cross-site cookies. |

```http
Set-Cookie: sessionid=abc123; SameSite=Strict; Secure; HttpOnly; __Host-
```

### Limitations of SameSite (Don't Rely Solely)
- `Lax` only blocks unsafe methods (POST/PUT) — **GET requests still pass!** Never use GET for state-changing operations.
- Scoped to registrable domain, not origin — `app.example.com` cookie is sent from `evil.example.com`
- Doesn't protect against **client-side CSRF** (where attacker-controlled input causes same-origin JS to send the request)
- Not a substitute for tokens — treat SameSite as an additional layer

---

## The Defense Checklist

For every state-changing endpoint:

1. ✅ **Not using GET** for state-changing actions (GET must be safe/idempotent)
2. ✅ **CSRF token** validated on the server (synchronizer pattern or signed double-submit)
3. ✅ **SameSite** set to `Lax` or `Strict` on session cookies
4. ✅ **`__Host-` prefix** on token cookies (prevents subdomain overwrite)
5. ✅ **Custom header** on AJAX requests (CORS preflight blocks forged headers)
6. ✅ **Origin/Referer verification** as fallback (verify `Origin` header matches target)
7. ✅ **Fetch Metadata** (`Sec-Fetch-Site`) where supported

---

## Client-Side CSRF (Modern Attack Variant)

In SPAs, an attacker can manipulate URL parameters/hash fragments that the client-side JS uses to construct requests. The CSRF token and cookies are included automatically — defense is harder.

```javascript
// VULNERABLE: uses URL fragment for endpoint
const endpoint = window.location.hash.slice(1).split(';')[1];
fetch(endpoint, { method: 'POST', headers: { 'X-CSRF-Token': token }});

// Attacker trick: victim visits yoursite.com/#post;https://yoursite.com/delete-account
// The JS reads the fragment, makes a POST with the user's token
```

**Mitigations:**
- **Never use URL input** to determine request endpoints
- **Validate** endpoint/method against an allowlist
- **Store request data** as predefined safe combinations, not URL-driven
- **Input validation** on all attacker-controllable request parameters

---

## Login CSRF

CSRF on the login form (victim not authenticated yet). Attacker logs victim INTO the attacker's account, then victim enters credit card info that the attacker sees.

**Defense:** Include a CSRF token (pre-session token) on login forms. Generate a pre-session and validate it.

---

## SameSite = Insufficient Alone?

SameSite may be sufficient only if **all** of these hold:
- ✅ No shared registrable domain with untrusted subdomains
- ✅ No GET endpoints perform state changes
- ✅ Session cookie = `SameSite=Strict` (or `Lax` + `__Host-`)
- ✅ Origin/Referer verification on state-changing endpoints
- ✅ You accept excluding users on non-SameSite-enforcing browsers

**Most apps need token-based CSRF protection. SameSite is the extra layer.**

---

## References
- [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [PortSwigger: CSRF](https://portswigger.net/web-security/csrf)
- [RFC 6265bis: SameSite Cookie Attribute](https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis)
- [Fetch Metadata Specification (W3C)](https://www.w3.org/TR/fetch-metadata/)
