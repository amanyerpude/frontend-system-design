# OWASP Cross-Site Scripting (XSS) Prevention Cheat Sheet

> Official OWASP guidance on preventing XSS through context-aware output encoding, input validation, framework security, and layered defenses.
>
> Source: [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

---

## Introduction

Cross-Site Scripting (XSS) is a misnomer — the term has widened to include injection of basically **any content**, not just scripts. XSS attacks can lead to account impersonation, user behavior observation, external content loading, and stealing sensitive data.

**Key principle:** No single technique solves XSS. The right **combination** of defensive techniques is necessary.

---

## Framework Security

Modern web frameworks (React, Angular, Vue, Lit, etc.) reduce XSS bugs via templating, auto-escaping, and secure defaults. But vulnerabilities still occur in **escape hatches** and insecure usage:

- **`dangerouslySetInnerHTML`** in React (without sanitizing HTML)
- **Angular's** `bypassSecurityTrustAs*` functions
- **Lit's** `unsafeHTML` function
- **Template injection**
- **Out-of-date framework plugins/components**
- **Unvalidated URL values** (`javascript:` URLs in `src`/`href`)

Every framework has gaps — output encoding and HTML sanitization fill them.

---

## XSS Defense Philosophy: Perfect Injection Resistance

For an XSS attack to succeed, an attacker must insert and execute malicious content. Therefore **all variables** must go through validation, then escaping or sanitization. Any variable that skips this process is a weakness.

---

## Output Encoding (Core Defense)

Output encoding converts untrusted input into **safe data** (not runnable code). The key rule: **encode for the specific context** where the data is rendered.

### Output Encoding Contexts

#### 1. HTML Body Context
Insert data between basic HTML tags (`<div>`, `<b>`):
```
<div> $varUnsafe </div>
```
**Defense:** HTML entity encoding.
| Char | Encoded |
|------|---------|
| `&`  | `&amp;` |
| `<`  | `&lt;`  |
| `>`  | `&gt;`  |
| `"`  | `&quot;`|
| `'`  | `&#x27;`|

Use `textContent` to assign text without parsing HTML.

#### 2. HTML Attribute Context
Data inserted into attributes like `title` or `value`:
```
<input value="ENCODED DATA">
```
**Defense:** HTML attribute encoding. Always surround values with double (`"`) or single (`'`) quotes.

> Use `addEventListener()` with trusted handler functions instead of inline event-handler attributes. Inline handler code is decoded by the HTML parser before JavaScript runs — encoding cannot make it safe.

#### 3. JavaScript Context
Data inserted into inline `<script>` blocks:
```
<script>const message = "ENCODED DATA";</script>
```
**Defense:** Use a library encoder designed for both JavaScript string and HTML script contexts. Do not rely on manual quote escaping. Never insert untrusted code, identifiers, or expressions. Do not pass encoded values to `eval()`.

#### 4. CSS Context
Data inserted into inline CSS:
```
<div style="property: UNTRUSTED_DATA;">
```
**Defense:** Only place variables in CSS **property values**. Validate strictly. Use fixed properties (e.g., `element.style.color`) with allowlisted values. Never let untrusted input choose the property or supply a full declaration block.

#### 5. URL Context
Data inserted into URLs:
```
<a href="http://example.com?test=$varUnsafe">link</a>
```
**Defense:** URL encoding (`%HH` format). Fully quote attributes. For JavaScript URL construction, use `encodeURIComponent()` on each untrusted parameter.

---

## Dangerous Contexts (Avoid These)

Output encoding cannot fully protect in these contexts — **never** place untrusted variables here:

- Directly inside `<script>...</script>`
- Inside HTML comments (`<!-- -->`)
- Inside `<style>` blocks (`style` tags / `script` tags)
- In attribute names or tag names
- In `javascript:` URLs

---

## HTML Sanitization

When users need to author **actual HTML** (e.g., rich-text editors, Markdown renderers), output encoding breaks functionality. Use **HTML sanitization** instead:

```javascript
let clean = DOMPurify.sanitize(dirty);
```

Important caveats:
- If you sanitize then **modify** the result, you may void your security efforts.
- Check that downstream libraries don't **mutate** the sanitized string.
- **Regularly patch** DOMPurify or your sanitizer — browsers change and bypasses are discovered.

**OWASP recommends DOMPurify** for HTML sanitization.

---

## Safe Sinks (vs. Dangerous Sinks)

XSS sinks are places where variables are placed into a webpage. Use **safe sinks** that treat data as text:

```javascript
elem.textContent = data;              // Safe
elem.insertAdjacentText("beforeend", data);
formfield.value = data;
document.createTextNode(data);
elem.setAttribute(safeName, data);    // Safe name (not an event handler)
elem.innerHTML = DOMPurify.sanitize(data);
```

**Avoid** dangerous sinks:
- `innerHTML` (without sanitization)
- `document.write()`
- `eval()`, `setInterval()` with string args
- Event-handler attributes (`onclick="..."`)
- `javascript:` URLs

---

## Other Controls (Defense in Depth)

| Control | Purpose |
|---------|---------|
| **Cookie Attributes** | HttpOnly blocks JS access (limits XSS impact) |
| **Content Security Policy (CSP)** | Browser allowlist; defense-in-depth, not primary defense |
| **Trusted Types** | Forces safe DOM sinks — eliminates DOM-based XSS classes |
| **Web Application Firewall (WAF)** | Unreliable; not recommended as primary XSS defense |

---

## XSS Prevention Rules Summary

| Data Type | Context | Sample Defense |
|-----------|---------|----------------|
| String | HTML Body | HTML Entity Encoding (Rule #1) |
| String | Safe HTML Attributes | Quote value + attribute encoding |
| String | GET Parameter | URL Encoding (Rule #5) |
| String | Untrusted URL in href/src | Canonicalize + URL Validation + Allowlist http/https |
| String | CSS Value | Structural validation + CSS hex encoding |
| String | JavaScript Variable | Insert in quoted string + context-aware encoder |
| HTML | HTML Body | HTML validation (JSoup, AntiSamy, HTML Sanitizer) |
| String | DOM XSS | [DOM-based XSS Prevention Cheat Sheet](DOM_based_XSS_Prevention_Cheat_Sheet.html) |

---

## Common Anti-Patterns (Ineffective Approaches)

### Sole Reliance on CSP Headers
CSP is best as defense-in-depth, customized per application. Enterprise-wide one-size-fits-all CSP policies:
- Break legacy apps (business pushes back → waivers → cracks)
- Assume uniform browser support (risky for older browsers)
- Still need proper encoding as the primary defense

### Reliance on HTTP Interceptors/Filters
Interceptors can't know the **context** where data is rendered (HTML vs. JavaScript vs. CSS). They:
- Miss tainted input from headers, cookies, or internal databases
- Fail against DOM-based XSS entirely
- Produce double-encoding or broken rendering
- Can't reason about data from backend APIs or databases

---

## References
- [OWASP Java Encoder: Output Contexts and Boundaries](https://github.com/OWASP/owasp-java-encoder/blob/main/docs/contexts.md)
- [DOMPurify Documentation](https://github.com/cure53/DOMPurify)
- [XSS Filter Evasion Cheat Sheet](XSS_Filter_Evasion_Cheat_Sheet.html)
