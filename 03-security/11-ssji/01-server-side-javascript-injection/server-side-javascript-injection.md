# Server-Side JavaScript Injection (SSJI)

> SSJI is a vulnerability where user-provided input is executed as **JavaScript code on the server**. Unlike XSS (which runs in the victim's browser), SSJI runs on the server — allowing file system access, environment variable theft, shell command execution, and full host compromise. It's the JavaScript equivalent of SSTI (Server-Side Template Injection).
>
> Sources: [DevNotes — SSJI](https://notes.heysuraj.dev/docs/system-design/security/server-side-javascript-injection-ssji) | [PortSwigger — Server-side JS code injection](https://portswigger.net/kb/issues/00100d00_server-side-javascript-code-injection) | [Secforce — Server Side JavaScript Injection](https://www.secforce.com/blog/server-side-javascript-injection/)

---

## What Is SSJI

**Server-Side JavaScript Injection (SSJI)** occurs when an application takes user-controlled input and passes it directly to a JavaScript interpreter on the server — e.g., `eval()`, `Function()`, or a JS sandbox like Node's `vm` module.

| | Scope | Impact |
|---|-------|--------|
| **XSS** | Runs in victim's browser | Session theft, defacement, phishing |
| **SSJI** | Runs on the server | File system access, env var theft, shell exec, **full server compromise** |

---

## How SSJI Happens

### Pattern 1: `eval()` with User Input
```javascript
const userInput = req.query.math;  // Attacker sends: "; process.exit() //"
const result = eval(`(${userInput})`);  // BOOM — arbitrary code execution
```

### Pattern 2: `Function()` Constructor
```javascript
const fn = new Function(userInput);
fn();
```

### Pattern 3: Node.js `vm` Module (False Sense of Safety)
```javascript
const vm = require('vm');
vm.runInNewContext(userInput, {});  // vm sandboxes have a long history of escape bugs
```

### Pattern 4: Template Injection in JS Engines
```javascript
// If user input is interpolated into a template string
// and the engine allows arbitrary JS evaluation...
const template = `Hello ${user.name}`;
```

---

## Common Attack Scenarios

### Reading Environment Variables
```javascript
// Attacker sends: process.env
eval("process.env")
// → { AWS_SECRET_KEY: "...", DB_PASSWORD: "...", ... }
```

### File System Access
```javascript
// Attacker sends: require('fs').readFileSync('/etc/passwd', 'utf8')
require('fs').readFileSync('/app/.env');  // reads secrets file
```

### Shell Command Execution
```javascript
// Attacker sends: require('child_process').execSync('id').toString()
// → uid=0(root) gid=0(root)
```

### Prototype Pollution (Chained Exploit)
```javascript
// Attacker sends: "".__proto__.admin = true
// Pollutes Object.prototype, escalating privileges
```

---

## Real-World Example

A reporting engine that lets users write custom formulas:
```javascript
app.post('/calculate', (req, res) => {
  const { formula } = req.body;
  const result = eval(formula);  // "process.mainModule.require('fs')..."
  res.json({ result });
});
```

An attacker submits:
```
process.mainModule.require('fs').readFileSync('/etc/passwd', 'utf8')
```
→ Full server compromise via `/etc/passwd` access.

---

## Defense Strategies

### 1. **Never use `eval()`, `Function()`, or `vm` with untrusted input**
This is the #1 rule. These APIs almost never need to touch user input.

### 2. Use Safe Expression Libraries
For user-authored expressions (calculators, rules engines), use a **purpose-built, sandboxed evaluator**:

```javascript
// SAFE: expr-eval — sandboxed, no fs/process access
const { Parser } = require('expr-eval');
const parser = new Parser();
const expr = parser.parse('a + b * 2');
const result = expr.evaluate({ a: 5, b: 10 });
```

### 3. Input Validation (Allowlist)
```javascript
// Only allow alphanumeric + operators + parentheses
if (!/^[0-9a-zA-Z+\-*/().\s]+$/.test(userInput)) {
  throw new Error("Invalid characters");
}
```
Even allowlisting isn't foolproof — prefer a real parser.

### 4. AST-Level Validation
Parse the JS to an AST and walk the tree, rejecting any node types that could be dangerous (function calls, member access to `require`, `process`, etc.):

```javascript
const acorn = require('acorn');
const ast = acorn.parse(input, { ecmaVersion: 2020 });
// Walk the AST — reject CallExpression, MemberExpression, etc.
```

### 5. Run in an Isolated Sandbox
If you absolutely must execute untrusted JS:
- Use a **container** (Docker) with **no network access**, **read-only filesystem**
- Drop all Linux capabilities
- Set memory/CPU limits
- Even then, VM/sandbox escapes are discovered regularly — treat as defense-in-depth, not security

---

## Red Flags (Audit These Patterns)

| Pattern | Risk |
|--------|------|
| `eval(userInput)` | Critical — direct code execution |
| `new Function(userInput)` | Critical |
| `vm.runInNewContext(userInput)` | Critical — sandbox often escapeable |
| `setTimeout(userString)` | Critical — string executes as JS |
| `setInterval(userString)` | Critical |
| User input in template literals that get evaluated | High |
| Underscore.js / Lodash `.template(userInput)` | High — template injection |
| EJS, Pug, Handlebars accepting user templates | High — SSTI/SSJI |

---

## Detection

- **Static analysis:** Semgrep rules for `eval()`, `Function()`, `vm.*`
- **Manual audit:** Search codebase for `eval(`, `new Function(`, `vm.`, `setTimeout(` with strings
- **Code review:** Every code path that builds and executes JS from user input

---

## Interview Framing

SSJI maps to **OWASP Top 10 A03:2021 — Injection**. In interviews:
- Explain why `eval()` is dangerous (arbitrary code execution in privileged context)
- Show the difference between server-side (SSJI) and client-side (XSS) injection
- Describe safe alternatives (sandboxed evaluators, AST validation)
- Acknowledge that sandboxing is never a complete defense

---

## References
- [CWE-94: Improper Control of Generation of Code (Code Injection)](https://cwe.mitre.org/data/definitions/94.html)
- [PortSwigger: Server-side JavaScript code injection](https://portswigger.net/kb/issues/00100d00_server-side-javascript-code-injection)
- [Node.js vm module — security considerations](https://nodejs.org/api/vm.html#vm_vm)
- [expr-eval — SAFE sandboxed math parser](https://github.com/silentmatt/expr-eval)
