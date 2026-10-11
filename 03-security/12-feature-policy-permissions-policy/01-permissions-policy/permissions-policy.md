# Permissions-Policy (formerly Feature-Policy)

> Permissions-Policy is an HTTP response header (or `<iframe allow>` attribute) that controls which origins can use which **browser features** (camera, microphone, geolocation, etc.). This limits the blast radius of an XSS — even if an attacker injects script, they can't access device APIs you've disabled.
>
> Source: [MDN — Permissions-Policy Header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Permissions-Policy) | [OWASP HTTP Headers Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html)

---

## Why Permissions-Policy Matters

If an XSS attacker gets script execution on your page, they can potentially:
- Turn on your **camera** or **microphone**
- Access your **geolocation**
- Use **fullscreen** to fake a browser UI
- **Capture** your screen
- Access **USB/Bluetooth** devices

Permissions-Policy lets you **disable** these features at the browser level — an XSS payload can't re-enable a feature that's been denied by policy.

> Formerly called **Feature-Policy** (renamed in 2021). Old browsers used `Feature-Policy`; modern use `Permissions-Policy`.

---

## Syntax

### HTTP Header
```http
# Disable a feature for all origins
Permissions-Policy: camera=(), microphone=(), geolocation=()

# Allow only self
Permissions-Policy: camera=(self), geolocation=(self)

# Allow specific trusted origins
Permissions-Policy: camera=(self "https://trusted.example.com")

# Multiple features (comma-separated)
Permissions-Policy: camera=(), microphone=(), geolocation=self
```

### `<iframe allow>` Attribute (HTML)
```html
<iframe
  src="https://example.com"
  allow="camera 'self'; geolocation 'none'; microphone 'src'">
</iframe>
```

---

## Allowlist Values

| Value | Meaning |
|-------|---------|
| `*` | Feature allowed everywhere (default for most) |
| `()` | Feature **disabled** everywhere — highest security |
| `self` | Allowed for the current document + same-origin iframes |
| `"https://origin"` | Allowed for this specific origin only |
| `src` | Allowed if iframe's `src` is same-origin (iframe attribute only) |

> `*` and `()` must be used alone. `self` and origins can be combined.

---

## Common Directives (Browser Features to Control)

| Directive | Feature | Why Disable |
|-----------|---------|-------------|
| `camera` | Video input | XSS could spy on users |
| `microphone` | Audio input | Sound recording |
| `geolocation` | Geo API | Location tracking |
| `payment` | Payment Request API | Financial transactions |
| `usb` | WebUSB | Access hardware |
| `bluetooth` | Web Bluetooth | Connect to devices |
| `fullscreen` | Fullscreen API | UI spoofing |
| `display-capture` | Screen sharing | Record screen |
| `accelerometer` | Motion sensors | Device fingerprinting |
| `gyroscope` | Gyroscope | Device fingerprinting |
| `magnetometer` | Compass | Device fingerprinting |
| `battery` | Battery Status API | Fingerprinting |
| `ambient-light-sensor` | Light sensor | Fingerprinting |
| `picture-in-picture` | Picture-in-picture | UI manipulation |
| `speaker-selection` | Audio output selection | Eavesdropping |
| `storage-access` | Storage Access API | Third-party cookie access |
| `local-fonts` | Font access | Fingerprinting |
| `vr`/`xr-spatial-tracking` | VR/XR | Immersive access |
| `autoplay` | Media autoplay | Annoyance, can mask attacks |

---

## Key Features

### Disabling Powerful Features
```http
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
```
This **completely blocks** these APIs across the document and all iframes.

### Per-Iframe Granularity
You can set a broad policy in the header and then narrow it per iframe:

```http
# Global: allow camera for self + ad network
Permissions-Policy: camera=(self "https://trusted-ad-network.com")
```
```html
<!-- This ad iframe can't use camera despite global allow -->
<iframe src="https://trusted-ad-network.com" allow="camera 'none'"></iframe>
<!-- This partner iframe can -->
<iframe src="https://trusted-ad-network.com" allow="camera"></iframe>
```

### Cross-Origin Isolation
The `cross-origin-isolated` directive controls whether `SharedArrayBuffer`, `performance.measureUserAgentSpecificMemory()`, and high-resolution timers are available — these are needed for Spectre-style attacks.

```http
Permissions-Policy: cross-origin-isolated=(self)
```

### Local Network Access
Prevent scripts from making requests to local/loopback addresses:
```http
Permissions-Policy: local-network=(), loopback-network=()
```
This mitigates **client-side SSRF** (where JS in the browser fetches `http://127.0.0.1:*` to hit internal services).

---

## Reporting Violations

You can report policy violations to a server endpoint using the **Reporting API**:

```http
Reporting-Endpoints: default="https://example.com/reports"
Permissions-Policy: camera=();report-to=default
```

When a blocked feature is requested, the browser sends a violation report.

---

## Browser Support

Permissions-Policy is supported in **Chrome/Edge 85+, Firefox 96+, Safari 16+** (partial). Check the [MDN compatibility table](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy#browser_compatibility) before relying on directives in production — support varies by feature.

---

## Testing

```bash
curl -sI https://yourapp.example.com | grep -i "permissions-policy"

# Test that a blocked feature throws in the browser console
navigator.mediaDevices.getUserMedia({video:true})  // Should reject if camera=()
```

Browser DevTools → **Application** tab → **Permissions** panel also shows active policies.

---

## Security Baseline

```http
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(),
  usb=(), bluetooth=(), display-capture=(), 
  accelerometer=(), gyroscope=(), magnetometer=(),
  picture-in-picture=(), fullscreen=(self),
  local-network=(), loopback-network=()
```

**Principle:** Start by disabling everything you don't use, then enable per-feature as needed. Don't assume "we don't use that" — a compromised third-party script or XSS payload absolutely will.
