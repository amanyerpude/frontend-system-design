# Server-Side Request Forgery (SSRF) Prevention

> SSRF abuses an application to make requests to internal/external networks or the server itself. Enabled by mishandling of URLs (image download, Webhook URL, internal service calls).
>
> Source: [OWASP SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)

---

## What Is SSRF

SSRF tricks a server into making requests to:
- **Internal networks** (`10.x`, `172.16-31.x`, `192.168.x`)
- **The server itself** (`localhost`, `127.0.0.1`)
- **Cloud metadata endpoints** (`169.254.169.254`, `metadata.google.internal`, Azure IMDS)

**Entry points:** user-submitted image URLs, Webhook/callback URLs, server-side fetches for previews/scraping/proxying.

> Not limited to HTTP — attackers use `gopher://`, `file://`, `ftp://`, `data://`, `dict://`, `phar://`.

---

## SSRF Attack Flow

```
Attacker → Vulnerable App: Crafted request (e.g., URL = http://169.254.169.254/...)
Vulnerable App → Target: Forwards request using vulnerable URL
Target → Vulnerable App: Response (e.g., cloud credentials)
Vulnerable App → Attacker: Returns response
```

The attacker uses the app **as a proxy** to reach internal systems.

---

## Two Defense Cases

### Case 1: App calls only identified, trusted applications
Use the **allowlist approach** — you know which services you call.

**Application-layer (4 steps):**
1. **Validate input format** with vetted libraries (not hand-rolled regex)
   - Strings: regex for simple formats; library methods for complex ones
   - IPs: use `InetAddressValidator` (Java), `ip-address` (JS), `IPAddr` (Ruby) — these handle hex/octal/DWORD bypasses
   - Domains: use `DomainValidator` (Java), `is-valid-domain` (JS), `validators.domain` (Python)
2. **Allowlist the destination** — cross-check IP/domain against a list of permitted targets
3. **Resolve DNS once, validate the IP, bind the connection** — prevent DNS rebinding
   - Resolve DNS → validate IP → connect to validated IP while preserving hostname for `Host` header, SNI, and TLS cert verification
4. **Block redirects** — disable HTTP redirect following in your HTTP client

**Network-layer (defense in depth):**
- Firewall rules limiting which IPs/ports the app can reach
- Network segregation to block illegitimate outbound calls

```python
# Example: monitoring allowlist domains against internal IP ranges
import ipaddress, dns.resolver
DNS_RESOLVER = dns.resolver.Resolver()
DNS_RESOLVER.nameservers = ["1.1.1.1"]

def verify_dns_records(domain, records):
    for record in records or []:
        value = record.to_text().strip()
        try:
            ip = ipaddress.ip_address(value)
            if not ip.is_global:  # Blocks localhost, private, link-local
                print(f"[!] {domain} resolves to internal IP {value}")
                return True  # Flag it
        except ValueError:
            return True
    return False
```

### Case 2: App can fetch ANY external URL (e.g., Webhooks)
Allowlists aren't possible — destinations are dynamic. Use **blocklist** (last resort) plus a legitimacy token.

**Validation flow:**
1. Validate input is a valid IP/domain
2. **Block private/local ranges:**
   | Category | Ranges to Block |
   |----------|----------------|
   | Localhost | `127.0.0.0/8`, `0.0.0.0/8`, `::1/128` |
   | RFC1918 Private | `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` |
   | Cloud metadata | `169.254.169.254`, `metadata.google.internal` |
   | IPv6 private | `fc00::/7` |
   | IPv6 link-local | `fe80::/10` |
   | Multicast | `224.0.0.0/4`, `ff00::/8` |
3. Use a **legitimacy token**: target generates a random token; callback must pass it (POST-only)
4. **Disable redirects** in the HTTP client
5. **Validate protocol** against allowlist (HTTP/HTTPS only; block `gopher://`, `file://`, etc.)

---

## Cloud Metadata Protection (High Priority)

The most dangerous SSRF target is the **cloud metadata service**:

| Cloud | Metadata Endpoint | Mitigation |
|-------|-------------------|------------|
| AWS | `169.254.169.254` / `metadata.amazonaws.com` | Migrate to **IMDSv2** (requires session token); disable IMDSv1 |
| GCP | `metadata.google.internal` | Restrict with firewall rules |
| Azure | `169.254.169.254` | Restrict via route tables |

---

## Bypass Techniques Defenders Must Know

- **IP encoding:** Hex (`0x7f000001`), octal, DWORD (`2130706433`), mixed encodings
- **DNS rebinding:** Resolve allowlisted domain → validate IP → DNS changes between validation and connection
- **Redirect chains:** `https://evil.com` → 302 → `http://127.0.0.1`
- **URL parser disagreement:** `http://example.com\@evil.com` — WHATWG URL parser reads host as `example.com`; RFC 3986 / urllib parses `evil.com`
- **Decimal/octal encodings in hostnames**

> **Defense:** Reject URLs whose host is not read identically by every parser. Match the host against an allowlist, then **build the request yourself** — don't carry over the user's path/query.

---

## Common Mistakes

1. **Trusting IP allowlists without DNS validation** at connection time
2. **Following redirects** — attacker redirects to internal IP after validation
3. **Allowing `0.0.0.0`** — maps to `127.0.0.1` in many environments
4. **Allowing `[::]`** — IPv6 "any" address, may resolve to localhost
5. **Using `localhost` keyword** — resolves to `127.0.0.1`
6. **Permitting internal subdomains** like `vault.service.consul`
7. **Forgetting IPv6** — blocking `127.0.0.0/8` but allowing `[::1]`

---

## Detection & Testing

- **Semgrep rules:** Search for SSRF patterns with `semgrep -r "r/q/ssrf"`
- **Burp Suite / ZAP** SSRF scans
- **URL validation bypass cheat sheet** (PortSwigger): enumerate edge cases

Verify your app can't reach metadata:
```bash
# Simulate from your app's runtime network
curl http://169.254.169.254/latest/meta-data/  # Should be blocked
curl http://metadata.google.internal/          # Should be blocked
```

---

## References
- [CWE-918: Server-Side Request Forgery](https://cwe.mitre.org/data/definitions/918)
- [PortSwigger URL Validation Bypass Cheat Sheet](https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet)
- [AWS IMDSv2 Guidance](https://aws.amazon.com/blogs/security/imdsv2-security/)
- [OWASP SSRF Bible (PDF)](https://cheatsheetseries.owasp.org/assets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet_SSRF_Bible.pdf)
