# Subresource Integrity (SRI)

> SRI lets browsers verify that resources fetched from a CDN are delivered without tampering, using a cryptographic hash. Protects against supply-chain attacks where a compromised CDN serves malicious code.
>
> Sources: [MDN SRI Implementation Guide](https://developer.mozilla.org/en-US/docs/Web/Security/Practical_implementation_guides/SRI) | [OWASP SRI Control](https://community.owasp.org/controls/SubresourceIntegrity) | [subresource-integrity.com](https://subresource-integrity.com/core-sri-fundamentals-browser-security-boundaries/)

---

## The Problem SRI Solves

If an attacker compromises a CDN hosting a JS library your site loads, they can inject malicious code that:
- Alters download links
- Defaces the site
- Steals credentials
- Causes denial of service

Your site trusts the CDN because `<script src="...">` looks legitimate — but the **content has been swapped**.

---

## How SRI Works

SRI binds a **cryptographic digest** to every external resource tag. The browser:
1. Fetches the resource
2. Computes the hash
3. **Matches:** executes the resource
4. **Mismatch:** refuses to execute

This is **fail-closed** — the page breaks rather than serving malicious code.

---

## Required Attributes

| Attribute | Purpose |
|-----------|---------|
| `integrity` | Contains `sha384-{base64hash}` or `sha512-{base64hash}` |
| `crossorigin` | Must be set (`anonymous`) for cross-origin resources, even if no CORS is needed |

```html
<script
  src="https://code.jquery.com/jquery-2.1.4.min.js"
  integrity="sha384-R4/ztc4ZlRqWjqIuvf6RX5yb/v90qNGx6fS48N0tRxiGkqveZETq72KgDVJCp2TC"
  crossorigin="anonymous">
</script>
```

> SRI only works when resources load over HTTPS — prevents a network attacker from stripping the `integrity` attribute.

---

## How to Generate Integrity Hashes

### Option 1: SRI Hash Generator (Easiest)
Use [srihash.org](https://srihash.org/) — generates the script tags with integrity attributes automatically, warns if CDN lacks CORS support.

### Option 2: Command Line
```bash
# For a file you control
openssl dgst -sha384 -binary library.js | openssl base64 -A
# Or via curl + pipe:
curl -s https://code.jquery.com/jquery-3.7.1.min.js | \
  openssl dgst -sha384 -binary | openssl base64 -A
```

The result is your `integrity` hash: `sha384-{output}`

### Option 3: Webpack Plugin (Automated)
```javascript
// webpack.config.js
const SriPlugin = require('webpack-subresource-integrity');
module.exports = {
  output: {
    crossOriginLoading: 'anonymous'
  },
  plugins: [
    new SriPlugin({
      hashFuncNames: ['sha384', 'sha512'],
      enabled: process.env.NODE_ENV === 'production'
    })
  ]
};
```

---

## SRI Best Practices

1. **Use SRI for all externally-hosted scripts** (CDN libraries, fonts, icons)
2. **Use `sha384` or `sha512`** — not `sha256` (too weak)
3. **Pin to specific versions** — never use a "latest" URL with SRI
4. **Include `crossorigin="anonymous"`** on every SRI-protected resource
5. **Update hashes on legitimate changes** — when the CDN updates, the hash breaks and you must regenerate

> If you pin to a version and never change it, the hash stays valid indefinitely. If you use `latest`, you'll need automation to regenerate hashes on every update.

---

## What SRI Cannot Protect Against

❌ **Your page choosing to load from a CDN** — if the HTML is tampered (attacker controls your server), they can change both `src` and `integrity` together

❌ **Dynamically-generated scripts** — if the script content changes per request, SRI's static hash won't match

❌ **Compromised dependencies you already vetted** (first-time install risk) — SRI detects tampering of a *known* version, not a *new* malicious version that passes the hash

SRI is **defense-in-depth** alongside CSP, not a standalone solution.

---

## The Polyfill.io Lesson (Real Incident)

In September 2024, `polyfill.io` (100K+ sites) was compromised. The attacker:
1. Waited for the domain to be sold to an unknown buyer
2. Modified `polyfill.js` on the CDN to steal user data and crypto wallet keys
3. Served the malicious code to **all sites** loading from `polyfill.io`

**If those sites had SRI:** the hash would have mismatched on the altered file → browsers refused to execute → attack blocked. The page might break, but users weren't compromised.

This is SRI's **core value**: fail-closed instead of silent compromise.

---

## SRI + CSP: Defense in Depth

| Control | What It Protects | Limitation |
|---------|-----------------|------------|
| **SRI** | Detects CDN/file tampering before script runs | Only works on pinned external scripts |
| **CSP (strict, nonce-based)** | Prevents execution of injected scripts | Bypassed by script gadgets on allowlisted domains |

**Together:** SRI stops the CDN compromise from running. CSP (with `strict-dynamic` + nonces) stops any injected script even if SRI is bypassed. Use both.

---

## SRI in Frameworks

| Framework | How |
|-----------|-----|
| **Webpack** | `webpack-subresource-integrity` plugin (auto-generates hashes at build) |
| **Next.js** | Built-in SRI via `experimental.sri` flag |
| **React/Vite** | `vite-plugin-html` with SRI plugin |
| **Vanilla HTML** | Manual `integrity` + `crossorigin` attributes |
