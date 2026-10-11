# Input Validation & Sanitization

> Input validation checks whether data meets requirements before use. It reduces malformed data, invalid operations, and resource exhaustion. Validation is distinct from output encoding (the latter prevents XSS in the rendered output).
>
> Source: [OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)

---

## Core Principle: Allowlist, Don't Denylist

> **Define what's allowed and reject everything else. Do not try to recognize every malicious string.**

Blocking apostrophes, for example, rejects legitimate names (O'Malley) without making a database query safe. For free-form text, allow broad Unicode within length limits — you don't need to restrict users to letters and digits.

OWASP Proactive Control C3 distinguishes validation from additional defenses needed downstream:
- **Validate** all input at the boundary
- Use **parameterized queries** for SQL
- Use **context-aware output encoding** for XSS

---

## Input Validation Strategies: Syntax vs Semantics

Validate **both**:
- **Syntax** — expected type and format (e.g., a date looks like a date)
- **Semantics** — whether the value makes sense for the operation (e.g., end date is after start date)

Define rules per field type:

| Input | Rules to Enforce |
|-------|------------------|
| **Fixed choices** (dropdowns) | Exact membership in allowed set — including POSTed values |
| **Numbers and dates** | Expected type, accepted format, min & max values |
| **Strings** | Length limits + required characters/structure |
| **Objects** | Allowed fields, required fields, null handling |
| **Arrays** | Min/max item counts, validate every item (including nested) |

---

## Parse Safely, Then Validate

1. **Apply request size limits** before buffering/parsing input
2. **Configure parser limits** (max nesting depth, max JSON depth/numbers per RFC 8259)
3. Use a **maintained parser** for the expected format; handle parsing failures
4. Validate resulting values **before** business processing/storage
5. **Decode per protocol** before checking rules; validate the representation actually used (avoid double-decoding downstream — a CWE-20 risk)

> Converting text to an integer establishes a **type**, not an acceptable **quantity**. `"99999999999999999999"` parses fine but may not be a valid ID.

---

## Regular Expressions (Use With Care)

Good for simple, structured fields:
- Use `fullmatch()` (Python) — match the **entire** value, not a substring
- Check character class and newline behavior across languages
- **Bound input length** before matching
- Avoid patterns with **excessive backtracking** (ReDoS risk); use non-backtracking engines or match timeouts
- Treat a timeout as validation failure

---

## Validating Free-form Unicode Text

- Agree on a **character encoding** (UTF-8) across all components; reject malformed input
- Preserve legitimate punctuation and scripts (names, comments in different languages)
- If using Unicode normalization for consistent comparison, define and apply the **same policy** before validation, storage, and comparison
- ⚠️ Normalization is **not** sanitization and does not replace output encoding

---

## Validating Rich User Content

When accepting user-authored HTML (WYSIWYG editors, Markdown), use a **maintained HTML sanitization library** like DOMPurify. Regular expressions and input validation cannot replace this:

```javascript
let clean = DOMPurify.sanitize(dirty, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
  ALLOWED_ATTR: ['href'],
  ALLOWED_URI_REGEXP: /^(?:https?|mailto):/i,
  FORBID_TAGS: ['style', 'script', 'iframe'],
  FORBID_ATTR: ['onerror', 'onload', 'style']
});
```

---

## File Upload Validation

Treat **filename** and **content type** as untrusted metadata:
- Validate filenames **after** protocol decoding (an allowed extension doesn't mean safe content)
- Follow the OWASP [File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) for content checks, size limits, storage, and safe serving

---

## Client-Side vs Server-Side Validation

| Layer | Purpose |
|-------|---------|
| **Client-side** | Immediate UX feedback — **bypassable**, never trust |
| **Server-side** | Security boundary — **must** validate here |

Always validate on the server even when the browser checks the same fields. Client-side validation is for convenience, not security.

---

## Common Pitfalls

1. **Client-only checks** — bypassable; validate on server always
2. **Trusting internal sources** — validate data from internal APIs, queues, and stored records when crossing trust boundaries (an internal transport ≠ the value meets rules)
3. **Denylisting/sanitizing** — remove suspicious characters changes meaning and doesn't make queries/encoding safe. Use field-specific allowlist rules.
4. **Stopping at parsing** — test rejected ranges, missing fields, invalid nested items, oversized arrays — not just malformed syntax
5. **Trusting upload metadata** — check filename isn't enough; check the file content
6. **Logging rejected input verbatim** — record failure + metadata without secrets/full bodies. Escape retained values for the log format to prevent log injection

---

## Allowlist vs Denylist: The Difference

| Approach | How It Works | Problem |
|----------|-------------|---------|
| **Denylist** | Recognize and block known-bad characters (e.g., block `<`, `>`, `'`) | Infinite attack surface — encodings, parser quirks, Unicode tricks, and `onerror=` always bypass it |
| **Allowlist** | Define exactly what characters/formats are accepted; reject the rest | Finite: enumerate what's safe, block everything else |

> ✅ **Use allowlists. Always.**

---

## Email Address Validation

Use a maintained email validation library compatible with your mail system's address format. Format validation does **not** prove mailbox access. See the OWASP [Email Validation & Verification Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Email_Validation_and_Verification_Cheat_Sheet.html).

---

## References
- [OWASP Proactive Control C3: Validate All Input & Handle Exceptions](https://top10proactive.owasp.org/archive/2024/the-top-10/c3-validate-input-and-handle-exceptions/)
- [CWE-20: Improper Input Validation](https://cwe.mitre.org/data/definitions/20.html)
- [OWASP DevGuide: Validate Inputs](https://devguide.owasp.org/en/04-design/02-web-app-checklist/05-validate-inputs/)
- [Unicode Standard Annex #15: Normalization Forms](https://www.unicode.org/reports/tr15/)
