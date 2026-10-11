# Clickjacking Defense / Iframe Protection

> Clickjacking (UI redressing) tricks users into clicking hidden page elements by overlaying transparent iframes. Defense is a "headers problem" solved by three layered mechanisms.
>
> Sources: [OWASP Clickjacking Defense Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking_Defense_Cheat_Sheet.html) | [AppSec Brief Implementation Guide](https://appsecbrief.com/articles/clickjacking-x-frame-options-csp-frame-ancestors-guide)

---

## The Attack

An attacker embeds your site in a transparent iframe on their own page, overlays fake UI on top, and tricks the victim into clicking what looks like a benign button — actually clicking the confirmation button on your site (fund transfer, OAuth grant, account deletion).

```html
<!-- attacker page -->
<style>
  iframe { opacity: 0; position: absolute; z-index: 2; }
  .fake-button { position: absolute; z-index: 1; }
</style>
<iframe src="https://victim-bank.com/transfer?amount=1000&to=attacker"></iframe>
<div class="fake-button">Click to claim your prize!</div>
```

Targeted actions: payment confirmations, OAuth permission grants, social media follow/sharing, account settings changes, one-click destructive actions.

---

## Three Defense Mechanisms (Use Together — Defense in Depth)

### Defense 1: `X-Frame-Options` HTTP Header

Tells browsers whether your page may be framed. Set on **all HTML responses** via middleware/server config.

| Value | Meaning | Recommendation |
|-------|---------|----------------|
| `DENY` | Cannot be framed by any origin | **Recommended** for most apps |
| `SAMEORIGIN` | Only same-origin framing allowed | When you have legitimate same-origin iframe use |
| `ALLOW-FROM <uri>` | **Deprecated** — not supported in Chrome/Firefox/Safari | Avoid |

> **Critical mistake:** `X-Frame-Options` in a `<meta>` tag does **NOT** work — must be an HTTP response header.

**Implementation:**
```javascript
// Express.js — using Helmet
const helmet = require('helmet');
app.use(helmet.frameguard({ action: 'deny' }));
```
```python
# Django — built in
MIDDLEWARE = ['django.middleware.clickjacking.XFrameOptionsMiddleware']
X_FRAME_OPTIONS = 'DENY'
```
```python
# Flask
@app.after_request
def add_security_headers(response):
    response.headers['X-Frame-Options'] = 'DENY'
    return response
```
```java
// Spring Security
http.headers(headers -> headers.frameOptions(frameOptions -> frameOptions.deny()));
```

---

### Defense 2: CSP `frame-ancestors` Directive (Modern Successor)

More flexible than `X-Frame-Options`. Specify which origins may embed your page:

```http
# Block all framing
Content-Security-Policy: frame-ancestors 'none';

# Only same-origin
Content-Security-Policy: frame-ancestors 'self';

# Specific trusted partners
Content-Security-Policy: frame-ancestors 'self' https://dashboard.example.com;
```

**Node.js (Express + Helmet):**
```javascript
app.use(helmet.contentSecurityPolicy({
  directives: { frameAncestors: ["'none'"] }
}));
```

**Nginx:**
```nginx
add_header X-Frame-Options "DENY" always;
add_header Content-Security-Policy "frame-ancestors 'none';" always;
```

> When both headers are present, `frame-ancestors` takes precedence in modern browsers. Set **both** for legacy-browser compatibility.

---

### Defense 3: `SameSite` Cookies

`SameSite` doesn't prevent framing, but it **breaks the auth state** clickjacking relies on — session cookies aren't sent in cross-site iframe requests.

```http
Set-Cookie: sessionid=abc123; SameSite=Strict; Secure; HttpOnly
```

| Value | Behavior |
|-------|----------|
| `Strict` | Never sent in cross-site requests. Most secure. Use when cross-site referral isn't needed. |
| `Lax` | Sent only for top-level navigation GETs. Good default for most session cookies. |
| `None` | Must pair with `Secure`. Required for cross-site cookies (third-party widgets). |

---

## DoubleClickjacking Defense

New variant: tricks user into clicking in a **separate window** during a double-click. Frame restrictions (`frame-ancestors`, `X-Frame-Options`) don't apply since the target isn't framed.

**Mitigations for high-risk actions (consent screens, payments, account security):**
- Require server-side transaction authorization (review details + confirm)
- Disable sensitive controls until prior interaction is detected
- Test with mouse, keyboard, touch, and assistive tech
- Keep standard framing defenses as the base layer

---

## Deprecated / Insecure Approaches (Do NOT Use)

- **JavaScript frame-busting** (`if (top !== self) top.location = self.location`) — easily bypassed via `sandbox` attribute or domain restriction
- **`ALLOW-FROM`** — not supported in modern browsers; fails open silently
- **`<meta http-equiv="X-Frame-Options">`** — doesn't work; must be HTTP header

---

## Verification

```bash
curl -sI https://yourapp.example.com | grep -i -E "x-frame|content-security"
# Expected:
# x-frame-options: DENY
# content-security-policy: frame-ancestors 'none'; ...
```

Run [Mozilla Observatory](https://observatory.mozilla.org/) or PageSpeed scan to confirm before deployment.
