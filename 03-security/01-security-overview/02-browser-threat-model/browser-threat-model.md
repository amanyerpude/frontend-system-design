# Frontend Application Security and the Browser Threat Model

> Design frontend security through trust boundaries: XSS, CSRF, CSP, token storage, third-party code, BFF, and supply-chain risk.
>
> Source: [System Design Space](https://system-design.space/en/chapter/frontend-security-browser-threat-model/)

---

## Core Principle: The Browser Is an Untrusted Runtime

You don't control the browser, its extensions, injected scripts, or network conditions. Treat the client as a public runtime under constant attacker pressure. This means:

- Don't store secrets in the browser expecting them to stay safe.
- Treat all client-side input as hostile.
- The server should make the final trust decision, not the frontend.

---

## Trust Boundary Map

```
Browser (untrusted)
  → Application Code (untrusted inputs)
    → Session Storage (defines blast radius)
      → API / BFF (server boundary — verifies authorization)
        → Third-Party Code (untrusted — same privileges as your code)
```

Each boundary must explicitly limit what the client may store, execute, and send onward.

| Boundary       | Responsibility |
|----------------|----------------|
| **Runtime**    | Browser, network, extensions, localStorage are outside your control |
| **Application Code** | Client logic treats inputs as untrusted; secrets never live here |
| **Session Storage**  | Cookies, tokens, and local data define the blast radius during XSS |
| **API / BFF**  | Server verifies action, authorization, and request shape |
| **Third-Party Code** | Widgets, analytics, packages get only the access allowed by policy |

---

## Main Threat Classes

1. **XSS & Script Injection** — user-controlled HTML, unsafe Markdown, third-party widgets, DOM-based sinks turn the UI into a hijack surface.
2. **CSRF, Session Fixation** — cookies, SameSite, CSRF tokens, redirect flows, anti-replay must all be understood in the browser context.
3. **Token Storage** — any long-lived token in browser storage is effectively leaked if one XSS occurs. `localStorage` and embedded config are not vaults.
4. **Supply Chain & Third-Party Scripts** — every external script runs with the same privileges as your own code.

---

## Protection Layers

### CSP and Script-Source Limitation
- CSP doesn't solve every problem, but it sharply reduces the cost of accidental XSS.
- Use a **nonce or hash strategy** and ban inline scripts.
- Control third-party domains explicitly.

### Safer Auth Boundaries
- Prefer `HttpOnly` cookies over tokens in storage.
- Use short-lived credentials and refresh-token rotation.
- A **Backend-For-Frontend (BFF)** exchanges the session cookie for a service token and keeps the refresh token itself.

### Escaping and Typed Rendering Surfaces
- Every path that renders HTML, Markdown, template fragments, or `dangerouslySetInnerHTML` is a boundary that needs explicit sanitization.
- Missed boundaries stay silent until an attacker reaches them.

### Dependency Hygiene and Provenance
- Lockfiles, dependency review, artifact integrity, and minimizing package sprawl matter on the frontend just as much as on the backend.

---

## Threat Modeling: Step by Step

| Element            | Questions |
|--------------------|-----------|
| **Assets**         | What does the user/company lose if a boundary is breached? (session, PII, money, trust) |
| **Entry Points**   | URL params/fragment, content from other users, `postMessage`, browser storage, third-party scripts, clipboard, drag-drop |
| **Attacker Capabilities** | Controls the network (open Wi-Fi), makes victim open a link, places content on your site, compromised a public-registry package |
| **Decision Table** | For each entry-point/asset pair: where does the trust boundary run, who validates the data, and what happens if the check is bypassed? An empty cell = a finding. |

---

## Browser Sessions in Practice

### Cookie Instead of a Token in Storage
A session cookie with `HttpOnly`, `Secure`, and `SameSite=Lax` is unreachable from script. XSS no longer *steals* the session — it can only use it while the page is open, bounding damage by time and context.

### Backend for Frontend (BFF) and Token Exchange
The browser knows only the cookie. The BFF exchanges it for a service token for domain services and keeps the refresh token itself.

### Refresh-Token Rotation Detects Theft
A second use of an old refresh token revokes the entire chain.

### PKCE for Public Clients (No Backend)
The authorization code is bound to a one-time secret, so an intercepted code is useless. Any app without its own server must use PKCE.

### Short Lifetimes and Revocation
Access tokens should live for minutes, not days. Sign-out and password changes revoke sessions on the **server**, not just in browser storage on one device.

---

## CSP in Production

### Strict Policy Instead of a Domain List
- One-time **nonce per response** + `strict-dynamic`: a script with the nonce may load other scripts, no domain allowlist needed.
- Domain lists are bypassed through open redirects and old JSONP endpoints on allowed hosts.

### Report-Only First
- Run the policy in **report-only mode** for a week, collect violations at a reporting endpoint, then switch on enforcement.
- Without this step, forgotten integrations break and the policy gets rolled back.

### Subresource Integrity + Trusted Types
- Hash of the script served from a CDN protects against file substitution.
- **Trusted Types** forbid assigning strings to dangerous DOM sinks without an explicit validation policy — DOM-based XSS disappears as a class.

### Isolating Third-Party Code
- Widgets live in an embedded `<iframe sandbox>` with a `Permissions-Policy` disabling camera, payments, fullscreen.
- Tag managers get the same review as code, because they are code in production.

---

## Security Review Questions

1. Which data do we treat as trusted in the browser, and what is that trust based on?
2. Can arbitrary script execution happen through content paths, Markdown, WYSIWYG, or third-party snippets?
3. Which client-side tokens or session artifacts create too large a blast radius if compromised?
4. How quickly can we disable an unsafe external integration without a full product rollback?

---

## Maturity Signal

A mature frontend product doesn't rely on a heroic reviewer. It rests on safe defaults: controlled rendering surfaces, minimal token exposure, a strict script policy, and the ability to disable an unsafe integration in minutes, not in a release cycle.
