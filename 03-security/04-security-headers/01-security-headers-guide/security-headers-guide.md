# HTTP Security Response Headers

> Proper HTTP response headers prevent XSS, clickjacking, information disclosure, and more. A baseline set should be applied at the CDN/server level so they're never missed on individual routes.
>
> Source: [OWASP HTTP Headers Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html)

---

## Essential Security Headers

### `X-Frame-Options` — Clickjacking Defense
> **Superseded by CSP `frame-ancestors`** in browsers that support it, but still useful for legacy compatibility.

- Indicates whether a page can be rendered in a `<frame>`, `<iframe>`, `<embed>`, or `<object>`
- Only useful when the response has interactive content (links, buttons) — useless on redirects or JSON API responses

```
X-Frame-Options: DENY
```
**Recommendation:** Use CSP `frame-ancestors` if possible. Otherwise, set `X-Frame-Options: DENY` (do not allow framing).

---

### `X-XSS-Protection` — **DEPRECATED / HARMFUL**
> ⚠️ **Do NOT set this header. Set it to `0` to disable it.**

IE/Chrome/Safari's XSS auditor filter can actually **create XSS vulnerabilities** in safe websites.

```
X-XSS-Protection: 0
```
**Recommendation:** Use a CSP that bans inline JavaScript instead.

---

### `X-Content-Type-Options: nosniff` — MIME Sniffing Defense
Prevents browsers from **sniffing/gessing MIME types**, which can turn non-executable types into executable ones (MIME Confusion Attacks).

```
X-Content-Type-Options: nosniff
```
**Recommendation:** Set correctly on all responses. Pair with a correct `Content-Type` header (e.g., `text/html; charset=utf-8`). The `charset` is required to prevent XSS in HTML.

---

### `Referrer-Policy` — Referrer Information Control
Controls how much referrer info (`Referer` header) is included with requests.

```
Referrer-Policy: strict-origin-when-cross-origin
```
Modern browsers default to this, but set it explicitly for certainty.
- Same-origin requests include origin + path + query string
- Cross-origin requests include origin only
- HTTPS-to-HTTP requests omit the header entirely

---

### `Content-Security-Policy (CSP)` — Primary XSS Defense
Specifies allowed content origins. Mitigates XSS and data injection attacks.

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-{random}'; object-src 'none';
```
**Recommendation:** See the dedicated [Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html) — CSP is complex and context-dependent.

> CSP is only relevant for responses that render scripts/code. A JSON API response gets no benefit.

---

### `Cache-Control` — Sensitive Data Caching
Controls how responses are cached by browsers and intermediate caches.

```
Cache-Control: no-store          # Prohibits storage in compliant caches (sensitive data)
Cache-Control: private          # Caches only in user-specific (non-shared) caches
Cache-Control: no-cache         # Cached but must revalidate first — does NOT prevent caching!
```
**Recommendation:** Use `no-store` for sensitive responses. Use `no-cache` carefully (it allows storage). The `no-store` directive doesn't control the Cache API — explicitly exclude sensitive data there too.

---

### `Set-Cookie` — Cookie Security
Not a security header per se, but cookie attributes are crucial.

**Recommendation:** See the [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html#cookies) for detailed cookie configuration. Key attributes:
- `HttpOnly` — blocks JS access
- `Secure` — HTTPS only
- `SameSite=Strict|Lax` — CSRF defense
- `__Host-` prefix — locks to host

---

### `Strict-Transport-Security (HSTS)` — Force HTTPS
Tells browsers to only use HTTPS, even if the user types `http://`.

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```
**Recommendation:** Set `max-age` ≥ 1 year (31536000). Use `includeSubDomains` only after testing all subdomains. `preload` requires HSTS at [hstspreload.org](https://hstspreload.org/). **Warning:** Misconfiguration locks users out until `max-age` expires.

---

## Cross-Origin Headers (Advanced Hardening)

These protect against Spectre-style attacks that cross the Same-Origin Policy boundary:

### `Cross-Origin-Opener-Policy (COOP)`
Isolates browsing context — top-level document doesn't share context group with cross-origin documents.
```
Cross-Origin-Opener-Policy: same-origin
```

### `Cross-Origin-Embedder-Policy (COEP)`
Prevents loading cross-origin resources without explicit permission (`require-corp`).
```
Cross-Origin-Embedder-Policy: require-corp
```
Requires `<img crossorigin>`, `<script crossorigin>` etc. on the resource side.

### `Cross-Origin-Resource-Policy (CORP)`
Controls which origins may load a resource — browser blocks it before it enters an attacker's process.
```
Cross-Origin-Resource-Policy: same-site
```

---

### `Permissions-Policy` — Browser Feature Control
Controls which origins can use which browser features (camera, microphone, geolocation, etc.). Prevents XSS from enabling device access.

```
Permissions-Policy: geolocation=(), camera=(), microphone=()
```
This example **disables** geolocation, camera, and microphone for all domains. Enable only what you need.

---

## Information-Disclosure Headers (Remove These)

### `Server`
Reveals server software. Remove or set non-informative values.
```
Server: webserver
```

### `X-Powered-By`
Exposes technology stack (e.g., PHP version, Express). Remove entirely.
```
# Remove all X-Powered-By headers
```

### `X-AspNet-Version` / `X-AspNetMvc-Version`
.NET version disclosure. Disable in `web.config` / `Global.asax`.

### `X-Robots-Tag`
Controls crawler indexing behavior:
```
X-Robots-Tag: noindex, nofollow       # Private/sensitive content
X-Robots-Tag: index, follow           # Public content you want indexed
```

### `X-DNS-Prefetch-Control`
Controls DNS prefetching. Set to `off` if you don't control outbound links.
```
X-DNS-Prefetch-Control: off
```

---

## Deprecated Headers (Do NOT Use)

### `Public-Key-Pins (HPKP)` — ❌
Removed from Chromium in 2018, unsupported by modern browsers. Causes outages. Use **Certificate Transparency (CT)** + **CAA DNS records** instead.

---

## Secure File Download Headers
When serving user-provided files, force download instead of inline rendering:
```
Content-Disposition: attachment
Content-Type: application/octet-stream
X-Content-Type-Options: nosniff
```

---

## Server / Framework Implementation

### Node.js (Express) — using Helmet
```javascript
const helmet = require('helmet');
app.use(helmet());                    // Sets most security headers
app.use(helmet.frameguard({ action: 'deny' })); // X-Frame-Options: DENY
```

### Nginx
```nginx
add_header X-Frame-Options "DENY" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;
add_header Content-Security-Policy "default-src 'self'; ..." always;
add_header Permissions-Policy "geolocation=(), camera=()" always;
```

### Apache (.htaccess)
```apache
<IfModule mod_headers.c>
Header unset X-Frame-Options
Header always set X-Frame-Options "DENY"
Header always set X-Content-Type-Options "nosniff"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
</IfModule>
```

---

## Testing Your Headers

- **[Mozilla Observatory](https://observatory.mozilla.org/)** — online scanner for homepage headers
- **[securityheaders.com](https://securityheaders.com)** — rates your header set (A-F grade)
- **SmartScanner** — scans all pages, not just the homepage

Verify with curl:
```bash
curl -sI https://yourapp.example.com | grep -i -E "x-frame|x-content|referrer|strict-transport|content-security|permissions"
```
