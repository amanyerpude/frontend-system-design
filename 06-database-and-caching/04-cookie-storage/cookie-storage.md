# Cookie Storage

## I. Overview

Cookies are small pieces of data stored in the browser and **automatically sent with every HTTP request** to the same origin. They are the only client-side storage mechanism that is automatically transmitted to the server, making them essential for authentication, session management, and tracking.

## II. Core Characteristics

| Characteristic | Behavior |
|----------------|----------|
| **Persistence** | Depends on `Expires`/`Max-Age` attribute (session or persistent) |
| **Scope** | Per domain (set via `Domain` attribute) |
| **Capacity** | ~4KB per cookie, ~50 cookies per domain |
| **API type** | Synchronous (JavaScript) + HTTP headers (server) |
| **Data type** | Strings only |
| **Sent with requests** | **Yes** — automatically with every HTTP request to matching domain |
| **Accessibility** | JavaScript (unless `HttpOnly`) + server |

## III. Cookie Attributes

```http
Set-Cookie: name=value; Domain=example.com; Path=/; Expires=Wed, 21 Oct 2026 07:28:00 GMT; Max-Age=3600; Secure; HttpOnly; SameSite=Strict
```

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `Domain` | Which domain the cookie is valid for | `example.com` or `.example.com` (includes subdomains) |
| `Path` | Which paths the cookie is sent to | `/` or `/api` |
| `Expires` | Absolute expiration date | `Wed, 21 Oct 2026 07:28:00 GMT` |
| `Max-Age` | Seconds until expiration | `3600` (1 hour) |
| `Secure` | Only send over HTTPS | `Secure` |
| `HttpOnly` | Not accessible via JavaScript | `HttpOnly` |
| `SameSite` | CSRF protection | `Strict`, `Lax`, or `None` |

## IV. Cookie Types by Purpose

### Session Cookies

No `Expires` or `Max-Age` attribute — deleted when the browser closes.

```http
Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict
```

### Persistent Cookies

Have `Expires` or `Max-Age` — survive browser restarts.

```http
Set-Cookie: preferences=theme%3Ddark; Max-Age=31536000; SameSite=Lax
```

### HttpOnly Cookies

Not accessible via JavaScript — immune to XSS theft.

```http
Set-Cookie: authToken=xyz789; HttpOnly; Secure; SameSite=Strict
```

```typescript
// ❌ Cannot access HttpOnly cookies
document.cookie  // Returns cookies WITHOUT HttpOnly flag
// authToken is NOT visible here
```

### Secure Cookies

Only sent over HTTPS — never transmitted over HTTP.

```http
Set-Cookie: sessionId=abc123; Secure
```

## V. SameSite Attribute (CSRF Protection)

`SameSite` controls when cookies are sent with cross-origin requests:

| Value | Behavior | Use Case |
|-------|----------|----------|
| `Strict` | Cookie not sent with any cross-origin request | Highest security; breaks external links |
| `Lax` | Cookie sent with top-level navigation (links), not cross-origin requests | Good default for most applications |
| `None` | Cookie always sent with cross-origin requests | Cross-origin APIs (requires `Secure`) |

```http
# Best practice: Lax for most cookies
Set-Cookie: sessionId=abc123; SameSite=Lax; Secure; HttpOnly

# Only use None for cross-origin API needs
Set-Cookie: apiToken=xyz789; SameSite=None; Secure; HttpOnly
```

## VI. JavaScript API

```typescript
// Read all cookies (semicolon-separated)
const allCookies = document.cookie
// "theme=dark; userId=42; sessionId=abc123"

// Set a cookie (no way to set attributes from JS — use server for that)
document.cookie = 'theme=dark; path=/; max-age=31536000'

// Delete a cookie (set expiration in the past)
document.cookie = 'theme=; path=/; max-age=0'

// Parse cookies into an object
function parseCookies(cookieString: string): Record<string, string> {
  return Object.fromEntries(
    cookieString.split('; ').map(cookie => {
      const [key, ...value] = cookie.split('=')
      return [key, decodeURIComponent(value.join('='))]
    })
  )
}

const cookies = parseCookies(document.cookie)
console.log(cookies.theme)  // 'dark'
```

## VII. Server-Side Cookie Management

Setting cookies via HTTP response headers is preferred over JavaScript because it allows setting `HttpOnly`, `Secure`, and `SameSite` attributes:

```typescript
// Express.js
app.post('/login', (req, res) => {
  const token = generateJWT(req.body.userId)

  // Set HttpOnly cookie — JavaScript cannot access it
  res.cookie('authToken', token, {
    httpOnly: true,
    secure: true,
    sameSite: 'lax',
    maxAge: 24 * 60 * 60 * 1000, // 24 hours
    path: '/',
  })

  res.json({ success: true })
})

// Read cookie on subsequent requests
app.get('/api/profile', (req, res) => {
  const token = req.cookies.authToken
  const user = verifyJWT(token)
  res.json(user)
})

// Clear cookie on logout
app.post('/logout', (req, res) => {
  res.clearCookie('authToken', { path: '/' })
  res.json({ success: true })
})
```

## VIII. Cookies vs. Other Storage

| Aspect | Cookies | Local Storage | Session Storage |
|--------|---------|--------------|-----------------|
| **Sent with requests** | Yes (automatically) | No | No |
| **Capacity** | ~4KB | ~5–10MB | ~5–10MB |
| **Expiration** | Configurable | Never | Tab close |
| **JS access** | Yes (unless HttpOnly) | Yes | Yes |
| **Server access** | Yes | No | No |
| **XSS vulnerability** | HttpOnly cookies: no | Yes | Yes |
| **CSRF vulnerability** | Yes (mitigated by SameSite) | No | No |
| **Scope** | Per domain | Per origin (all tabs) | Per tab |

## IX. Security Best Practices

### For Authentication Tokens

```http
# ✅ BEST — HttpOnly + Secure + SameSite=Lax
Set-Cookie: authToken=jwt_here; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=86400
```

**Why this is secure:**
- `HttpOnly` — JavaScript cannot read it → XSS-proof
- `Secure` — Only sent over HTTPS → MITM-proof
- `SameSite=Lax` — Not sent with cross-origin POST requests → CSRF-proof
- `Max-Age` — Automatic expiration limits exposure window

### Common Mistakes

| Mistake | Risk | Fix |
|---------|------|-----|
| Storing JWT in localStorage | XSS theft | Use HttpOnly cookie |
| Missing `SameSite` | CSRF attack | Set `SameSite=Lax` or `Strict` |
| Missing `Secure` | Cookie sent over HTTP | Always set `Secure` in production |
| Too many cookies | Each request carries all cookies (bandwidth waste) | Limit to essential cookies |
| Storing sensitive data in non-HttpOnly cookie | XSS theft | Only put sensitive data in HttpOnly cookies |

## X. Performance Considerations

- **4KB limit per cookie** — don't store large data
- **Sent with every request** — each cookie adds overhead to every HTTP request
- **Cookie overflow** — browsers limit ~50 cookies per domain; exceeding drops cookies
- **Use `Path` attribute wisely** — cookies sent to paths that need them only
- **Consider `__Host-` prefix** — `__Host-authToken` ensures cookie is set by HTTPS, has no Domain attribute, and path is `/`

## XI. Decision Framework

```
Need data available to the server?
├── Auth token → HttpOnly cookie (Secure, SameSite=Lax)
├── CSRF token → Non-HttpOnly cookie (readable by JS for form submission)
├── Session ID → HttpOnly cookie (Secure, SameSite=Strict)
├── User preferences (server needs them) → Regular cookie (Secure, SameSite=Lax)
└── Tracking/analytics → Third-party cookie (note: being phased out)
```

```
Need data only on client?
├── Small, persistent → Local Storage
├── Session-scoped → Session Storage
└── Large/structured → IndexedDB
```

## References

- [MDN: HTTP cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [OWASP: Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP: CSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [web.dev: Cookies](https://web.dev/learn/pwa/storage/)
