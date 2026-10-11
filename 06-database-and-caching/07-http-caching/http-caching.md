# HTTP Caching

## I. Overview

HTTP caching is the mechanism by which browsers and intermediate caches (CDNs, proxies) store and reuse HTTP responses to avoid unnecessary network requests. It is the **most impactful performance optimization** for repeat visits — cached resources load instantly without any network round trip.

## II. How HTTP Caching Works

```
User requests resource
        │
        ▼
┌─────────────────┐
│ Browser Cache?  │── Yes → Is it fresh? ── Yes → Serve from cache (instant)
└─────────────────┘             │
        │ No                     │ No
        ▼                       ▼
┌─────────────────┐    Revalidate with server
│  Send Request   │         │
│  to Server      │         ▼
└─────────────────┘    Server responds:
        │              304 Not Modified (use cache)
        │              200 OK (send new version)
        ▼
   Server Response
   (with cache headers)
        │
        ▼
   Store in Browser Cache
```

## III. Cache-Control Headers

`Cache-Control` is the primary mechanism for controlling HTTP caching.

### Directive Reference

| Directive | Meaning | Example |
|-----------|---------|---------|
| `no-cache` | Must revalidate with server before using cached copy | `Cache-Control: no-cache` |
| `no-store` | Never cache the response | `Cache-Control: no-store` |
| `public` | Can be cached by any cache (browser, CDN, proxy) | `Cache-Control: public, max-age=3600` |
| `private` | Can only be cached by the browser (not CDN/proxy) | `Cache-Control: private, max-age=3600` |
| `max-age` | Seconds the response is fresh | `Cache-Control: max-age=86400` |
| `s-maxage` | Seconds the response is fresh (for shared caches/CDNs) | `Cache-Control: s-maxage=3600` |
| `immutable` | Response will never change during max-age | `Cache-Control: public, max-age=31536000, immutable` |
| `must-revalidate` | Must revalidate if stale (no silent serving of stale) | `Cache-Control: max-age=3600, must-revalidate` |
| `stale-while-revalidate` | Serve stale while fetching fresh version | `Cache-Control: max-age=3600, stale-while-revalidate=86400` |

### Common Cache-Control Configurations

```nginx
# Static assets (JS, CSS, images) — long cache
Cache-Control: public, max-age=31536000, immutable

# HTML pages — short cache, must revalidate
Cache-Control: no-cache

# API responses — short cache
Cache-Control: public, max-age=60, s-maxage=300

# Sensitive data — never cache
Cache-Control: no-store

# Stale-while-revalidate — serve stale, update in background
Cache-Control: public, max-age=60, stale-while-revalidate=86400
```

## IV. ETag and Last-Modified (Revalidation)

When a cached response is stale, the browser revalidates with the server using conditional requests:

### ETag (Entity Tag)

A unique identifier for a specific version of a resource:

```http
# Server response
HTTP/1.1 200 OK
ETag: "abc123def456"
Cache-Control: public, max-age=3600

# Browser revalidation request
GET /app.js
If-None-Match: "abc123def456"

# If unchanged
HTTP/1.1 304 Not Modified
# (No body — browser uses cached version)

# If changed
HTTP/1.1 200 OK
ETag: "newversion789"
Content-Length: 45678
```

### Last-Modified

A timestamp of when the resource was last modified:

```http
# Server response
HTTP/1.1 200 OK
Last-Modified: Wed, 21 Oct 2026 07:28:00 GMT
Cache-Control: public, max-age=3600

# Browser revalidation request
GET /app.js
If-Modified-Since: Wed, 21 Oct 2026 07:28:00 GMT

# If unchanged
HTTP/1.1 304 Not Modified
```

### ETag vs. Last-Modified

| Aspect | ETag | Last-Modified |
|--------|------|---------------|
| **Precision** | Content-based (exact) | Timestamp-based (1-second resolution) |
| **Reliability** | High — detects any change | Lower — same-second changes missed |
| **Performance** | Requires server computation | Cheap to generate |
| **Best for** | Dynamic content, precise caching | Static files, simple caching |

## V. Cache-Busting Strategies

When deploying new versions of assets, browsers must fetch the new version, not use the cached one.

### Content Hashing (Best Practice)

```javascript
// webpack / Vite: output with content hash
// dist/assets/main.a1b2c3d4.js
// dist/assets/style.e5f6g7h8.css

// HTML references the hashed filename
<script src="/assets/main.a1b2c3d4.js"></script>
<link href="/assets/style.e5f6g7h8.css" rel="stylesheet">
```

**How it works:**
- Each build produces different hashes → different filenames
- Browser sees a new URL → treats it as a new resource → fetches fresh
- Old cached versions are unused (and expire naturally)

### Query String Versioning

```html
<!-- Version via query string -->
<script src="/app.js?v=1.2.3"></script>
```

**Downside:** Some CDNs ignore query strings. Content hashing is more reliable.

### Cache-Busting Middleware

```typescript
// Express.js — set appropriate headers per file type
app.use((req, res, next) => {
  if (req.path.match(/\.(js|css|png|jpg|jpeg|gif|svg|woff2?)$/)) {
    // Hashed assets: long cache
    res.set('Cache-Control', 'public, max-age=31536000, immutable')
  } else if (req.path.match(/\.html$/)) {
    // HTML: no cache (always check for new version)
    res.set('Cache-Control', 'no-cache')
  }
  next()
})
```

## VI. Cache Hierarchy

```
┌──────────────────────────────────────────────────────────────┐
│                      Cache Hierarchy                         │
│                                                              │
│  Browser Cache (Memory + Disk)                               │
│      │ Cache-Control: private, max-age                       │
│      ▼                                                       │
│  CDN Cache (Edge)                                            │
│      │ Cache-Control: public, s-maxage                       │
│      ▼                                                       │
│  Reverse Proxy Cache (Nginx, Varnish)                        │
│      │ Cache-Control: public, max-age                        │
│      ▼                                                       │
│  Origin Server                                               │
│                                                              │
│  Each layer can serve from cache without hitting origin.     │
└──────────────────────────────────────────────────────────────┘
```

## VII. Common Patterns

### Pattern 1: Long Cache for Hashed Assets

```nginx
# Hashed assets — never change filename
location ~* \.(js|css|woff2?|png|jpg|jpeg|gif|svg)$ {
    add_header Cache-Control "public, max-age=31536000, immutable";
}
```

### Pattern 2: Short Cache for API

```nginx
location /api/ {
    add_header Cache-Control "public, max-age=60, s-maxage=300, stale-while-revalidate=3600";
}
```

### Pattern 3: No Cache for Authentication

```nginx
location /api/auth/ {
    add_header Cache-Control "no-store";
}
```

### Pattern 4: SWR (Stale-While-Revalidate)

```typescript
// Client-side: use stale data while fetching fresh data
async function fetchDataWithSWR(url: string) {
  const cached = localStorage.getItem(`cache:${url}`)

  if (cached) {
    const { data, timestamp } = JSON.parse(cached)
    const age = Date.now() - timestamp

    if (age < STALE_THRESHOLD) {
      return data  // Serve fresh from cache
    }

    if (age < REVALIDATE_THRESHOLD) {
      // Serve stale, fetch fresh in background
      fetchAndCache(url)
      return data  // Return stale immediately
    }
  }

  // No cache or too old — fetch fresh
  return fetchAndCache(url)
}
```

## VIII. Debugging HTTP Caching

### Chrome DevTools

1. Open DevTools → Network tab
2. Check "Disable cache" to force fresh requests
3. Click a request → Headers tab → check `Cache-Control` and `ETag`
4. Look for `(from disk cache)` or `(from memory cache)` in the Size column

### cURL

```bash
# Check cache headers
curl -sI https://example.com/app.js | grep -i cache

# Force cache bypass
curl -sI -H "Cache-Control: no-cache" https://example.com/app.js

# Check if 304 is returned
curl -sI -H "If-None-Match: \"abc123\"" https://example.com/app.js
```

## IX. Key Takeaways

1. **Cache-Control is the primary caching mechanism** — use it on every response
2. **Hash your assets** — content hashing enables aggressive caching without staleness
3. **HTML should have `no-cache`** — it's the entry point that references hashed assets
4. **Use ETag for precise revalidation** — more reliable than Last-Modified
5. **stale-while-revalidate** improves perceived performance — serve stale, update in background
6. **Never cache sensitive data** — use `no-store` for auth endpoints

## References

- [MDN: HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)
- [MDN: Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)
- [web.dev: HTTP caching](https://web.dev/learn/performance/serialization/)
- [MDN: ETag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag)
