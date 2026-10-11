# Dependency & Supply Chain Security

> Over 80% of modern apps are third-party code. Supply chain attacks target these dependencies. SCA (Software Composition Analysis) + SBOMs are now legally required in the EU (CRA, NIS2).
>
> Sources: [The Practical Developer — npm Supply Chain](https://the-practical-developer.online/posts/npm-supply-chain-security-audit-ci/) | [dev.to — SCA Complete Guide 2026](https://dev.to/florianlenz/software-composition-analysis-sca-the-2026-complete-guide-1ob5)

---

## The Threat Landscape

Modern applications are **assembled, not written**. Over 80% of enterprise codebases contain open-source components, and 80% of those contain at least one known vulnerability.

**Recent incidents:**
- **event-stream (2018):** A malicious actor took over the package, harvested private keys, shipped to thousands of downstream consumers unseen for two months.
- **ua-parser-js (2021):** A package with 7M weekly downloads hijacked, stuffed with cryptominers and credential stealers.
- **eslint-scope (2024):** Even linting tools in your pipeline are attack surface.

> **Pinning versions does NOT protect you.** Your lockfile records a tarball hash, but if an attacker publishes a new version matching your semver range, npm silently fetches the new (malicious) payload.

---

## What Is SCA? (Software Composition Analysis)

SCA automatically identifies third-party open-source components in your codebase. It:
- Detects **known vulnerabilities** (cross-referenced against CVE/NVD databases)
- Checks **license compliance**
- Finds **outdated packages**
- Produces a **Software Bill of Materials (SBOM)** — a manifest of all components

| SAST | SCA |
|------|-----|
| Scans your custom, proprietary code | Scans third-party open-source dependencies |
| Detects logic flaws, SQLi, XSS | Detects known CVEs, outdated versions, license risks |
| Output: line-of-code fixes | Output: package upgrades, SBOM |

---

## Legal Drivers (Why This Isn't Optional Anymore)

- **EU Cyber Resilience Act (CRA):** Requires documenting all components via SBOM + active vulnerability monitoring for products sold in the EU.
- **NIS2 Directive:** Forces essential entities to secure supply chains. Banks, hospitals, energy providers now audit your SCA processes.
- **Consequence:** No SBOM or dependency scanning → lose enterprise contracts.

---

## Layer 1: `npm audit` Done Right

`npm audit` compares your tree against the npm Advisory database. The problem isn't the tool — it's running it without a policy, drowning teams in noise.

**Strategy:** `--audit-level=high` + an **allowlist** for annotated false positives.

```json
// .npm-audit-allowlist.json
{
  "ignore": [
    {
      "id": 1096098,
      "reason": "Prototype pollution in dev-only mock library, no production exposure",
      "expires": "2026-09-01"
    }
  ]
}
```

```javascript
// scripts/audit.js
import { readFileSync } from 'node:fs';
import { execSync } from 'node:child_process';
const allowlist = JSON.parse(readFileSync('.npm-audit-allowlist.json', 'utf-8'));
const ignoredIds = new Set(allowlist.ignore.map(i => i.id));
const result = JSON.parse(execSync('npm audit --json', { encoding: 'utf-8' }));
const failures = [];
for (const [pkg, advisories] of Object.entries(result.vulnerabilities || {})) {
  for (const via of advisories.via || []) {
    if (typeof via === 'object' && via.source && !ignoredIds.has(via.source)) {
      failures.push(`${pkg}: ${via.title || 'unknown'} (${via.severity})`);
    }
  }
}
if (failures.length > 0) {
  console.error('Blocking vulnerabilities found:\n' + failures.join('\n'));
  process.exit(1);
}
console.log(`Audit passed. ${ignoredIds.size} findings suppressed.`);
```

Now CI fails only on real High/Critical production threats, not dev-only noise.

---

## Layer 2: Lockfile Integrity Verification

Your `package-lock.json` contains SHA-512 integrity hashes. npm verifies these during install — **but only what's already in the lockfile**.

Enforce lockfile presence + integrity on every install:

```bash
# scripts/verify-lockfile.sh
if [ ! -f "package-lock.json" ]; then
  echo "ERROR: lockfile missing. Run 'npm install --package-lock-only'"
  exit 1
fi
# Verify all integrity fields parse correctly
INVALID=$(node -e "
const lock = require('./package-lock.json');
const issues = [];
for (const [path, meta] of Object.entries(lock.packages || {})) {
  if (meta.resolved && !meta.integrity) issues.push(path);
}
if (issues.length) console.log(issues.join('\n'));
")
if [ -n "$INVALID" ]; then
  echo "ERROR: Packages missing integrity hashes:" && echo "$INVALID" && exit 1
fi
```

Run **before** `npm ci` in CI.

---

## Layer 3: npm Provenance (npm 9+)

npm **provenance** attests that a package was built and published from a specific source repo and CI pipeline.

Check your existing dependencies:
```bash
npm query ":provenance"
```

For your own packages — enable provenance in your publish workflow:
```yaml
# .github/workflows/publish.yml
permissions:
  id-token: write  # Required for provenance
jobs:
  publish:
    steps:
      - run: npm publish --provenance --access public
```

Enforce provenance requirements in `.npmrc`:
```
provenance=true
```

---

## Layer 4: Dependency Drift Detection

Attackers can republish a package with the **same version** but a different tarball. Detect this by caching expected hashes and comparing:

```bash
# scripts/check-dependency-drift.sh
# Commit this hash cache to your repo; review changes in PRs
```

If a vetted package's tarball changes without a version bump → red flag. A reviewer must notice before merging.

---

## Layer 5: Dependency Policy Rules (CI)

Block risky dependency patterns with `dependency-cruiser` or a custom script:

```javascript
// scripts/check-deps.js
const pkg = JSON.parse(readFileSync('package.json', 'utf-8'));
for (const [name, version] of Object.entries(pkg.dependencies)) {
  if (version.startsWith('git+'))        // git: URLs bypass registry integrity
  if (version.startsWith('file:'))       // local paths: not reproducible in CI
  if (version.includes('*'))            // wildcard ranges: unpredictable
}
```

---

## Layer 6: Review Dependency Diffs in PRs

Treat every dependency update like a **code change**. Dependabot/Renovate PRs should be reviewed with the same rigor:

1. **What changed?** Patch version with hundreds of new files = suspicious.
2. **Who published it?** Maintainer list changed = potential account takeover.
3. **What new transitive deps were added?** New sub-dependencies = new attack surface.

Automate the transitive diff:
```bash
# Compare lockfile tree against base branch
git checkout main && npm ci && npm ls --all --json > /tmp/deps-main.json
git checkout PR_BRANCH && npm ci && npm ls --all --json > /tmp/deps-pr.json
# Diff the flattened dependency lists
```

---

## Complete CI Pipeline (GitHub Actions)

```yaml
name: Dependency Security
on: [pull_request, push]
jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'npm' }

      - name: Lockfile integrity check
        run: bash scripts/verify-lockfile.sh

      - name: Dependency policy check
        run: node scripts/check-deps.js

      - name: npm audit with allowlist
        run: node scripts/audit.js

      - name: Dependency drift check
        run: bash scripts/check-dependency-drift.sh

      - name: Transitive dependency diff (PRs only)
        if: github.event_name == 'pull_request'
        run: ...flatten and diff two lockfiles...
```

---

## What This Doesn't Solve

- **Zero-day exploits** in packages already installed (no advisory exists yet)
- **Typosquatting** (e.g., `lodashs` instead of `lodash`) — mitigated partially by dependency review, but the strongest defense is scoped publishing + package-manager warnings
- **Compromised npm registry** or **CA cert compromise** — existential risk with no practical app-level mitigation

---

## Key Takeaway

The cost of setup is **a few scripts + one CI workflow**. The cost of recovering from a compromised dependency is an **incident postmortem + forensic audit + crisis communication**. Set up the pipeline, suppress the noise, review every dependency change like it's code.
