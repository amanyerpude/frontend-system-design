# Minification vs Compression

## Overview

**Minification** and **compression** are two distinct techniques for reducing the size of web assets. They operate at different stages of the delivery pipeline and serve different purposes. Understanding both — and using both — is essential for web performance.

## Minification

Minification is a **build-time process** that removes unnecessary characters from source code without changing its functionality. It operates on text files (JavaScript, CSS, HTML, JSON, SVG).

### What Minification Removes

| Technique | Example | Size Reduction |
|-----------|---------|----------------|
| **Whitespace** | `const x = 1;` → `const x=1;` | ~5–15% |
| **Comments** | `// comment` → removed | ~5–10% |
| **Newlines** | Multi-line code → single line | ~5–10% |
| **Shortened variable names** | `functionName` → `a` (in terser) | ~10–20% |
| **Removed quotes** | `"key"` → `key` (CSS) | ~1–3% |
| **Shortened numbers** | `0.500` → `.5` | ~1% |

### Minification in Practice

```javascript
// Before minification
function calculateTotal(items) {
  // Sum up all item prices
  const total = items.reduce(function (accumulator, item) {
    return accumulator + item.price;
  }, 0);

  return total;  // Return the final total
}

// After minification
function calculateTotal(n){return n.reduce((n,r)=>n+r.price,0)}
```

### Minification Tools

| Asset | Tool | Bundler Integration |
|-------|------|-------------------|
| **JavaScript** | Terser, esbuild, SWC | webpack (TerserPlugin), Vite (built-in) |
| **CSS** | cssnano, Lightning CSS | PostCSS (cssnano), Vite (built-in) |
| **HTML** | html-minifier-terser, htmlnano | Build plugins |
| **JSON** | jsonminify | Manual or build step |
| **SVG** | SVGO | Build plugins |

### Build Configuration

```typescript
// vite.config.ts — minification is enabled by default in production
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    minify: 'terser',  // or 'esbuild' (faster, default)
    terserOptions: {
      compress: {
        drop_console: true,    // Remove console.log in production
        drop_debugger: true,
      },
      mangle: true,           // Shorten variable names
    },
  },
})
```

### What Minification Does NOT Do

- **Does not reduce transfer size over the wire** — the file is still text, not binary
- **Does not improve network transfer** — the server still sends the same text encoding
- **Does not handle binary assets** — images, fonts, videos are not minified

## Compression

Compression is a **transfer-time process** that encodes data using fewer bits. It happens between the server and the browser during HTTP transfer.

### Gzip

The most widely supported compression algorithm for web assets.

```nginx
# Nginx configuration
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
gzip_comp_level 6;  # 1-9, higher = smaller but slower
```

### Brotli

Google's compression algorithm — typically 15–25% better than Gzip for text assets.

```nginx
# Nginx with Brotli
brotli on;
brotli_types text/plain text/css application/json application/javascript text/xml application/xml;
brotli_comp_level 6;
```

```nginx
# Fallback to Gzip for browsers without Brotli support
brotli_static on;
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
```

### Compression Effectiveness

| Asset | Original | Gzip | Brotli | Reduction |
|-------|----------|------|--------|-----------|
| JavaScript (100KB) | 100KB | ~30KB | ~25KB | 70–75% |
| CSS (50KB) | 50KB | ~10KB | ~8KB | 80–84% |
| HTML (20KB) | 20KB | ~5KB | ~4KB | 75–80% |
| JSON (10KB) | 10KB | ~3KB | ~2.5KB | 70–75% |
| Image (PNG/JPEG) | — | No effect | No effect | 0% |
| Already-minified JS | 100KB | ~35KB | ~30KB | 65–70% |

**Key insight:** Compressing already-minified JavaScript yields less benefit because minification removed the patterns (whitespace, comments) that compression targets most effectively.

### Server Configuration

```nginx
# Express.js
import compression from 'express'
app.use(compression({
  threshold: 1024,      // Only compress responses > 1KB
  level: 6,             // Compression level (1-9)
  filter: (req, res) => {
    return !req.headers['x-no-compression']
  },
}))
```

## Minification + Compression Together

The two techniques are **complementary** — they target different parts of the pipeline:

```
Source Code
    │
    ▼
┌─────────────────────┐
│   Minification      │  Build time: removes unnecessary characters
│   (Terser, cssnano) │  Reduces file size by ~20-40%
└─────────────────────┘
    │
    ▼
Minified File (e.g., 300KB JS)
    │
    ▼
┌─────────────────────┐
│   Compression       │  Transfer time: encodes with fewer bits
│   (Gzip, Brotli)    │  Reduces transfer size by ~70-80%
└─────────────────────┘
    │
    ▼
Compressed Transfer (e.g., ~90KB)
    │
    ▼
Browser decompresses → Parses minified JS → Executes
```

### Combined Effect

```
Original JS:          500KB
After minification:   300KB  (-40%)
After compression:     90KB  (-80% from minified)
Total reduction:       82%   (500KB → 90KB)
```

## What NOT to Compress

| Asset | Why |
|-------|-----|
| **JPEG/PNG/GIF** | Already compressed; compressing again wastes CPU for no benefit |
| **MP4/WebM** | Already compressed video formats |
| **WOFF2 fonts** | Already compressed font format |
| **Pre-compressed assets** | Compressing twice is wasteful |

```nginx
# Only compress text-based types
gzip_types text/plain text/css application/json application/javascript
           text/xml application/xml image/svg+xml;
# Do NOT add: image/jpeg image/png video/mp4 font/woff2
```

## Minification vs Compression: Key Differences

| Aspect | Minification | Compression |
|--------|-------------|-------------|
| **When** | Build time | Transfer time |
| **How** | Removes unnecessary characters | Encodes with fewer bits |
| **Who** | Build tools (Terser, cssnano) | Server/browser (Gzip, Brotli) |
| **Reduction** | ~20–40% | ~70–80% (on top of minification) |
| **Applies to** | JS, CSS, HTML, JSON, SVG | Any text-based HTTP response |
| **Binary assets** | No | No (JPEG, PNG, video) |
| **Reversible** | No | Yes (browser decompresses) |
| **Affects file size** | Yes (smaller file on disk) | No (same file, smaller transfer) |
| **Affects parsing** | Yes (faster parsing of minified code) | No (browser decompresses first) |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| **Only minifying, not compressing** | Missing ~70–80% transfer reduction | Enable Gzip/Brotli on server |
| **Only compressing, not minifying** | Compressing verbose code is less efficient | Minify before compressing |
| **Compressing binary assets** | Wastes CPU, no benefit | Exclude images, video, fonts |
| **Not compressing in CI** | Local testing shows different sizes than production | Enable compression in test environment |
| **Using Gzip when Brotli is available** | Missing ~15–25% improvement | Use Brotli with Gzip fallback |

## References

- [web.dev: Minification](https://web.dev/learn/performance/serialization)
- [MDN: HTTP Compression](https://developer.mozilla.org/en-US/docs/Web/HTTP/Compression)
- [Google: Brotli](https://github.com/google/brotli)
- [Terser](https://github.com/terser/terser)
