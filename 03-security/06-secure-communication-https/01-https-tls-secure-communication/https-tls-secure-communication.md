# HTTPS, TLS, and Secure Communication

> HTTPS is HTTP over TLS — it protects transport (confidentiality + integrity + identity), not application correctness. Treated as a trust-and-transport system.
>
> Sources: [MDN TLS Configuration Guide](https://developer.mozilla.org/en-US/docs/Web/Security/Practical_implementation_guides/TLS) | [FrontendInterviews.dev HTTPS/TLS Guide](https://frontendinterviews.dev/web-fundamentals/https-tls)

---

## Why TLS Matters

If data is sent over the web unencrypted, a **man-in-the-middle (MiTM)** can intercept, read, and modify it. All inbound/outbound communication must use HTTPS (TLS).

TLS provides three assurances:
| Question | TLS Mechanism | Why It Matters |
|----------|--------------|----------------|
| Can observers read bytes? | Encryption | Protects credentials, cookies, tokens, form data, API payloads |
| Can an attacker change bytes silently? | Authenticated encryption / integrity checks | Prevents undetected tampering |
| Is this the intended server? | Certificate chain + hostname validation | Prevents active impersonation by MiTM |

---

## The TLS Handshake (Simplified)

```
ClientHello   →   supported TLS versions, cipher suites, random values, SNI hostname
ServerHello + certificate   →   selected parameters, server certificate chain
Certificate validation   →   trusted issuer, valid dates, hostname match, policy checks
Key agreement   →   shared session keys
Encrypted HTTP   →   application data protected by TLS
```

The certificate is a **signed statement binding a hostname** (e.g., `app.example.com`) **to a public key**, not just a public key. The browser accepts it only if:
- The chain leads to a **trusted Certificate Authority**
- The certificate is within its **validity period**
- The **hostname matches**

> **Common interview mistake:** Saying "HTTPS prevents MITM because it encrypts." The stronger answer: TLS resists active MITM when certificate validation succeeds and users/tooling do not bypass trust failures.

---

## Browser Guarantees Beyond Encryption

### Secure Contexts
Many browser features are available only over HTTPS (powerful APIs that shouldn't expose data over plaintext).

### Mixed Content
A secure page is weakened by insecure subresources. Browsers **block active content** (scripts) loaded over HTTP:
```html
<!-- BAD -->
<script src="http://cdn.example.com/app.js"></script>
```
Verify all resources load over HTTPS before deployment.

### Cookies
The `Secure` cookie attribute ensures cookies are sent only over HTTPS. Pair with `HttpOnly`, `SameSite`, and proper scope.

### What HTTPS Does NOT Solve
- **XSS** — malicious JS still runs inside the secure page
- **CSRF** — abuse of automatically attached credentials if defenses are weak
- **Authorization bugs** — data leaks over an encrypted channel
- **Origin compromise** — a hacked CDN, dependency, or build pipeline can still serve malicious code

---

## HSTS (HTTP Strict Transport Security)

The `Strict-Transport-Security` header tells browsers to use HTTPS for your host for a specified time, even if the user types `http://`.

```http
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

### Why HSTS Matters: The First-Hop Downgrade Problem
A user types `example.com` (no scheme). If the browser tries HTTP first, an attacker can interfere before the redirect to HTTPS. HSTS prevents this after the browser has learned the policy.

### HSTS Options
| Directive | Purpose |
|-----------|---------|
| `max-age=31536000` (1 year) or `max-age=63072000` (2 years) | Minimum 6 months; longer is recommended |
| `includeSubDomains` | Upgrades all subdomain requests to HTTPS — **careful**, can break non-HTTPS subdomains |
| `preload` | Submits to the HSTS preload list; browsers upgrade before first contact |

### Tradeoff
HSTS is powerful and **unforgiving**. Once `includeSubDomains` or `preload` is set, misconfigured subdomains take your site offline for the duration. Only enable when HTTPS coverage, redirects, and subdomain ownership are mature.

### Implementation (HSTS + CDN redirect + upgrade-insecure-requests)
```nginx
# 1. Redirect all HTTP to HTTPS (same host)
server {
  listen 80;
  return 301 https://$host$request_uri;
}

# 2. Set HSTS header on HTTPS
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
```
```http
# 3. Upgrade insecure requests via CSP
Content-Security-Policy: upgrade-insecure-requests;
```

---

## TLS Configuration Best Practices

- Use the [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/) with Mozilla's [TLS guidelines](https://wiki.mozilla.org/Security/Server_Side_TLS)
- Enforce **TLS 1.2+** (disable TLS 1.0 and 1.1)
- Prefer **ECDHE** key exchange (perfect forward secrecy)
- Disable weak cipher suites (no RC4, no 3DES, no EXPORT ciphers)
- Set up **certificate renewal monitoring** — expiry/wrong SANs/missing intermediates cause outages

---

## Performance and Operations

| Concern | Model |
|---------|-------|
| Handshake latency | Most visible on cold connections and high-latency networks |
| Connection reuse | HTTP/2, HTTP/3, and session resumption reduce repeated setup cost |
| CDN termination | TLS often terminates at the edge, then follows the provider's origin policy |
| Certificate lifecycle | Expiry, wrong SANs, missing intermediates, failed renewal = complete outages |

---

## Senior Interview Answer Pattern

| Interview Prompt | Strong Response |
|------------------|-----------------|
| Is HTTPS enough for secure auth? | No. It protects transport; session design, cookie attributes, CSRF defense, XSS prevention, and authorization still matter. |
| Why do certificate warnings matter? | Bypassing them defeats the identity check that prevents impersonation. |
| Why can mixed content be dangerous? | Reintroduces insecure fetches into a secure page, especially for active content (scripts). |
| Should every site use HSTS preload? | Only when HTTPS coverage and subdomain ownership are mature; preload is sticky. |
