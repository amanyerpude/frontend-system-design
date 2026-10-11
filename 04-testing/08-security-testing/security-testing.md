# Security Testing

## What Is Security Testing?

Security testing is the process of evaluating an application to identify vulnerabilities, weaknesses, and misconfigurations that could be exploited by attackers. Unlike functional testing (which asks "does it work?"), security testing asks **"can this be broken, and what happens when it is?"**

Security testing should be **continuous** — integrated throughout the development lifecycle, not a one-time audit before release. The cost of fixing a security bug grows exponentially as it moves downstream: a flaw caught during design costs a fraction of one caught in production.

## The OWASP Framework

The **Open Web Application Security Project (OWASP)** provides the foundational frameworks for web application security testing.

### OWASP Top 10 (2025)

The most critical web application security risks:

| # | Risk | Category | Frontend Impact |
|---|------|----------|-----------------|
| 1 | **Broken Access Control** | Authorization | Frontend routes leaking unauthorized data |
| 2 | **Cryptographic Failures** | Data Protection | HTTP instead of HTTPS, sensitive data in URLs |
| 3 | **Injection** | Input Handling | XSS, SQL injection via unsanitized input |
| 4 | **Insecure Design** | Architecture | Missing security controls at design time |
| 5 | **Security Misconfiguration** | Infrastructure | Default credentials, verbose errors, CORS misconfigs |
| 6 | **Vulnerable Components** | Dependencies | Outdated npm packages with known CVEs |
| 7 | **Authentication Failures** | Identity | Weak session management, credential stuffing |
| 8 | **Data Integrity Failures** | Data | Unsigned updates, CI/CD pipeline compromises |
| 9 | **Logging Failures** | Monitoring | Insufficient security event logging |
| 10 | **SSRF** | Server-Side | Server making requests to attacker-controlled URLs |

### OWASP Web Security Testing Guide (WSTG)

The WSTG is the comprehensive methodology for testing web application security. It covers:

```
WSTG Categories:
├── Information Gathering        (fingerprinting, technology detection)
├── Configuration & Deploy       (TLS, HTTP headers, CORS)
├── Identity Management          (registration, login, account recovery)
├── Authorization                (directory traversal, privilege escalation)
├── Authentication               (credential attacks, session fixation)
├── Session Management           (session hijacking, CSRF)
├── Input Validation             (XSS, SQLi, command injection)
├── Error Handling               (information disclosure via errors)
├── Cryptography                 (weak algorithms, transport security)
├── Business Logic               (workflow bypass, rate limiting)
├── Client-Side Testing          (DOM XSS, clickjacking, CSRF)
└── API Testing                  (REST/GraphQL specific vulnerabilities)
```

## Testing Approaches

Security testing combines multiple approaches — no single technique is sufficient.

### 1. SAST (Static Application Security Testing)

SAST analyzes source code **without executing it** to find vulnerability patterns. It's the "white-box" approach.

**What SAST detects:**
- Hardcoded credentials and API keys
- Unsanitized user input flows to dangerous sinks
- Use of vulnerable functions (`eval`, `innerHTML`, `dangerouslySetInnerHTML`)
- Missing input validation
- Cryptographic weaknesses

**Frontend SAST tools:**
- **ESLint Security Plugin** — `eslint-plugin-security`, `eslint-plugin-no-unsanitized`
- **Semgrep** — Pattern-based static analysis with custom rules
- **SonarQube/SonarCloud** — Code quality + security rules
- **CodeQL** — GitHub's semantic code analysis engine

```bash
# ESLint security plugin
npm install --save-dev eslint-plugin-security

# .eslintrc
{
  "plugins": ["security"],
  "extends": ["plugin:security/recommended"]
}

# This flags potentially unsafe regex patterns (ReDoS)
# This flags object injection, non-literal require, etc.
```

### 2. DAST (Dynamic Application Security Testing)

DAST tests the **running application** from the outside — simulating an attacker without access to source code. It's the "black-box" approach.

**What DAST detects:**
- XSS (reflected, stored, DOM-based)
- SQL injection
- CSRF vulnerabilities
- Security header misconfigurations
- Authentication bypasses

**DAST tools:**
- **OWASP ZAP** — Free, open-source web vulnerability scanner
- **Burp Suite** — Industry-standard web security testing platform
- **Nikto** — Web server scanner for dangerous files/programs

```bash
# OWASP ZAP quick scan
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py \
  -t https://your-app.example.com \
  -r zap-report.html

# ZAP full active scan (more thorough, but sends attack payloads)
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py \
  -t https://your-app.example.com
```

### 3. SCA (Software Composition Analysis)

SCA analyzes **third-party dependencies** for known vulnerabilities (CVEs).

**Frontend SCA tools:**
- **npm audit** — Built into npm; checks against the npm advisory database
- **Snyk** — Continuous monitoring, auto-fix PRs, license compliance
- **Dependabot** — GitHub-native dependency scanning and alerts
- **OWASP Dependency-Check** — Broad language support

```bash
# npm audit — check for vulnerable dependencies
npm audit

# npm audit fix — automatically fix where possible
npm audit fix

# npm audit fix --force — apply breaking fixes
npm audit fix --force

# Snyk continuous monitoring
npx snyk test
npx snyk monitor
```

### 4. Manual Penetration Testing

Automated tools find known vulnerability patterns but **cannot understand business logic flaws, authorization bypasses, or context-specific risks**. Manual testing by skilled testers finds what tools miss.

**When manual testing is essential:**
- Authentication and authorization workflows
- Business logic (e.g., can a user apply a coupon twice?)
- Multi-step attack chains (combine low-severity findings into high-impact exploits)
- Custom cryptographic implementations
- Rate limiting and abuse prevention

## CI/CD Security Integration

Security testing should be automated in the CI/CD pipeline:

```yaml
# GitHub Actions security pipeline
name: Security

on: [push, pull_request]

jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci

      # ESLint security rules
      - run: npx eslint --plugin security src/

      # Semgrep static analysis
      - uses: returntocorp/semgrep-action@v1
        with:
          config: p/owasp-top-ten

  sca:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci

      # npm audit
      - run: npm audit --audit-level=high

      # Snyk (requires SNYK_TOKEN)
      - uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  dast:
    runs-on: ubuntu-latest
    needs: [sast, sca]
    steps:
      - uses: actions/checkout@v4

      # OWASP ZAP baseline scan against preview deployment
      - uses: zaproxy/action-baseline@v0.10.0
        with:
          target: 'https://preview.your-app.com'
```

## Testing Security Headers

Security headers are a critical part of the security posture. Verify they're present and correctly configured:

```bash
# Check security headers with curl
curl -sI https://example.com | grep -iE \
  'strict-transport|content-security|x-frame|x-content-type|referrer|permissions-policy'

# Expected headers:
# Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
# Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc123'
# X-Frame-Options: DENY
# X-Content-Type-Options: nosniff
# Referrer-Policy: strict-origin-when-cross-origin
# Permissions-Policy: camera=(), microphone=(), geolocation=()
```

```javascript
// Express.js security headers middleware
import helmet from 'helmet'
import cors from 'cors'

app.use(helmet())
app.use(cors({
  origin: 'https://your-app.com',
  credentials: true,
}))
```

## Frontend-Specific Security Tests

### XSS Testing

```javascript
// Test: verify user input is properly sanitized
it('should not execute script tags in user comments', async () => {
  const maliciousInput = '<script>alert("xss")</script>'

  await page.fill('[data-testid="comment-input"]', maliciousInput)
  await page.click('[data-testid="submit-comment"]')

  // Verify no dialog was shown (script didn't execute)
  const dialogShown = await page.evaluate(() => {
    return new Promise((resolve) => {
      window.alert = () => resolve(true)
      setTimeout(() => resolve(false), 100)
    })
  })

  expect(dialogShown).toBe(false)
  // Verify text is displayed as text, not executed
  await expect(page.locator('.comment-text'))
    .toHaveText(maliciousInput)
})
```

### CSRF Testing

```javascript
// Test: verify CSRF token is validated
it('should reject requests without CSRF token', async () => {
  const response = await fetch('/api/transfer', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ amount: 1000, to: 'attacker-account' }),
  })

  expect(response.status).toBe(403) // CSRF token missing
})
```

### CORS Misconfiguration Testing

```bash
# Check if origin reflection is happening
curl -sI -H "Origin: https://evil.com" https://api.your-app.com/users
# ❌ Bad: Access-Control-Allow-Origin: https://evil.com (reflected)
# ✅ Good: No Access-Control-Allow-Origin header (origin not whitelisted)
```

## Security Testing Maturity Model

| Level | Approach | Coverage |
|-------|----------|----------|
| **1 — Ad hoc** | Manual checks, occasional audits | Inconsistent |
| **2 — Automated basics** | npm audit + SAST in CI | Dependencies, known patterns |
| **3 — Comprehensive** | SAST + DAST + SCA + manual review | Broad coverage, known + unknown |
| **4 — Continuous** | All of the above + threat modeling + bug bounties | Continuous, proactive |
| **5 — Integrated** | Security as code, shift-left, DevSecOps culture | Embedded in every process |

## Key Principles

1. **No silver bullet** — Automated tools are necessary but insufficient; layer manual review and threat modeling
2. **Continuous testing** — Integrate security checks into every build, not just pre-release
3. **Risk-based prioritization** — A single critical authorization bypass outweighs dozens of low-severity XSS findings
4. **Defense in depth** — Test every layer: network, application, data, user
5. **Assume breach** — Design and test for what happens when a control fails

## References

- [OWASP Web Security Testing Guide](https://wstg.owasp.org/)
- [OWASP Top 10 2025](https://owasp.org/Top10/)
- [OWASP ZAP](https://www.zaproxy.org/)
- [Semgrep Rules Registry](https://semgrep.dev/docs/rules)
- [Snyk Open Source](https://snyk.io/product/open-source-security/)
- [Helmet.js (Express Security Headers)](https://helmetjs.github.io/)
