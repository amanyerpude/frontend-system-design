# How to Design a Frontend Security Architecture

> Step-by-step guide on architecting a frontend application that defends against XSS, CSRF, clickjacking, and supply chain attacks through systematic security controls.
>
> Source: [Namaste Frontend System Design](https://namastedev.com/guides/namaste-frontend-system-design/how-to-design-a-frontend-security-architecture)

---

## 1. Implement a Strong Content Security Policy

A Content Security Policy is an HTTP header that tells browsers which sources of content are permitted to load on the page.

- Start with a restrictive `default-src 'self'` policy and selectively add permitted sources.
- Use **nonces** for necessary inline scripts rather than `unsafe-inline`.
- Implement CSP as an **HTTP header**, not a `<meta>` tag, so it applies to the page itself.
- A strict CSP blocks inline scripts, `eval` usage, and loading from unauthorized domains — eliminating the vast majority of XSS attack vectors.

---

## 2. Sanitize All User-Generated Content Before Rendering

- Never render user-supplied HTML directly using `dangerouslySetInnerHTML` in React or `innerHTML` in vanilla JS.
- If rich-text rendering is required, sanitize the HTML through a library like **DOMPurify** before rendering.
- DOMPurify strips all dangerous tags and attributes while preserving safe formatting.
- For plain text, use React's default text rendering (auto-escapes all HTML characters).
- Review every single `dangerouslySetInnerHTML` usage — each is a potential XSS injection point.

---

## 3. Implement CSRF Protection for State-Changing Requests

- Read a random token from a cookie and include it as a custom request header in every state-changing API call (POST/PUT/DELETE).
- Cross-origin requests cannot read cookies, so an attacker cannot include the valid token.
- Configure your API client interceptor to automatically attach this header to all non-GET requests.
- Validate the token on the server for every state-changing endpoint.

---

## 4. Configure Subresource Integrity for Third-Party Scripts

- When loading JS/CSS from a third-party CDN, add an `integrity` attribute with a cryptographic hash of the expected file content.
- The browser verifies the downloaded file matches the hash before executing it.
- Any file that does not match is rejected.
- Generate hashes at build time for all external resources and include them in the HTML.

---

## 5. Prevent Clickjacking with Frame Restrictions

- Use the `X-Frame-Options` HTTP header set to `SAMEORIGIN` (allows framing only from the same domain).
- Use the CSP `frame-ancestors` directive for more granular control.
- Verify both headers are set on **every page response** at the infrastructure level — any unprotected page is a potential attack vector.

---

## 6. Audit and Minimize Third-Party Dependencies

- Run `npm audit` in the CI pipeline and fail the build on high-severity vulnerabilities.
- Use Dependabot or Renovate to automate dependency update pull requests.
- Periodically audit direct dependencies and remove anything not needed.
- Lock dependency versions in `package-lock.json` or `yarn.lock` and commit the lockfile.
- Consider using a private npm registry that scans packages before installation.

---

## 7. Implement Secure Cookie Configuration

Cookies storing sensitive values (like refresh tokens) must use the correct security attributes:

| Attribute    | Purpose |
|--------------|---------|
| `HttpOnly`   | Blocks JavaScript access — eliminates XSS-based cookie theft |
| `Secure`     | Ensures transmission only over HTTPS |
| `SameSite`   | Set to `Strict` or `Lax` — prevents cookies from being sent with cross-origin requests (CSRF protection) |
| `Domain`/`Path` | Restricts cookie scope to only the parts of the app that need it |

---

## 8. Establish a Security Headers Baseline

Configure a baseline set of headers on every HTTP response through CDN or web server config (not application code):

- **Content-Security-Policy**
- **X-Frame-Options**
- **X-Content-Type-Options: nosniff** (prevents MIME-type sniffing)
- **Referrer-Policy: strict-origin-when-cross-origin**
- **Permissions-Policy** (disable unused browser features like camera/microphone)

Test the configuration using [securityheaders.com](https://securityheaders.com) after each deployment.
