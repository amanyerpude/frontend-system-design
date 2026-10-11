# SPA vs MPA

## Overview

**Single Page Applications (SPAs)** and **Multi-Page Applications (MPAs)** are the two fundamental architectural patterns for web applications. The choice directly impacts loading performance, interaction speed, SEO, and development complexity.

## Single Page Application (SPA)

A SPA loads the entire application shell on the first request, then handles all subsequent navigation client-side via JavaScript routing. No full page reloads after initial load.

```
Initial Load:
┌─────────────────────────────────────────────────────┐
│  Server sends: HTML shell + JavaScript bundle       │
│  Browser: Parses HTML, loads JS, renders app        │
│  First load is slower (heavy JS bundle)             │
└─────────────────────────────────────────────────────┘

Navigation:
┌─────────────────────────────────────────────────────┐
│  User clicks "About"                                │
│  JavaScript: Intercepts click → updates URL         │
│  React/Vue: Renders new content (no full reload)    │
│  Instant navigation after initial load              │
└─────────────────────────────────────────────────────┘
```

### Examples

- Gmail, Google Maps, Twitter/X, Facebook
- Most modern web applications (React, Vue, Angular)

### Advantages

| Advantage | Description |
|-----------|-------------|
| **Fast navigation** | No full page reload after initial load |
| **Rich interactions** | Smooth transitions, animations, state persistence |
| **Server load** | Less server load — only serves API data after initial load |
| **Offline capability** | Service workers can enable offline mode |
| **Shared state** | State persists across navigations |

### Disadvantages

| Disadvantage | Description |
|-------------|-------------|
| **Slow initial load** | Large JavaScript bundle must download and execute |
| **SEO challenges** | Search engines historically had trouble with JS-rendered content |
| **Memory usage** | Application stays in memory; leaks are a risk |
| **Complexity** | Client-side routing, state management, code splitting |
| **Deep linking** | Requires extra configuration for URL routing |

## Multi-Page Application (MPA)

An MPA is the traditional architecture: each page is a separate HTML document served by the server. Navigation triggers a full page reload.

```
Initial Load:
┌─────────────────────────────────────────────────────┐
│  Server sends: HTML page for Home                   │
│  Browser: Parses HTML, loads CSS/JS, renders page   │
│  First load is faster (less JS)                     │
└─────────────────────────────────────────────────────┘

Navigation:
┌─────────────────────────────────────────────────────┐
│  User clicks "About"                                │
│  Browser: Sends new request to server               │
│  Server: Returns new HTML page                      │
│  Browser: Full reload → new page rendered            │
│  Slower navigation (full reload each time)           │
└─────────────────────────────────────────────────────┘
```

### Examples

- Traditional server-rendered apps (PHP, Django, Rails)
- E-commerce sites (many product pages)
- Content-heavy sites (news, blogs)

### Advantages

| Advantage | Description |
|-----------|-------------|
| **Fast initial load** | Smaller initial payload; server renders HTML |
| **SEO-friendly** | Server-rendered HTML is immediately indexable |
| **Simpler architecture** | Standard HTTP, no client-side routing |
| **Better for content** | Each page can be independently optimized |
| **Lower memory** | Pages are replaced, not accumulated |

### Disadvantages

| Disadvantage | Description |
|-------------|-------------|
| **Slow navigation** | Full page reload on every navigation |
| **Server load** | Server must render every page |
| **No state persistence** | State is lost on navigation (unless in cookies/storage) |
| **Less rich interactions** | Harder to build smooth, app-like experiences |
| **Duplicate code** | JS/CSS must be loaded on every page |

## Comparison

| Aspect | SPA | MPA |
|--------|-----|-----|
| **Initial load** | Slower (heavy JS bundle) | Faster (server-rendered HTML) |
| **Navigation speed** | Faster (no reload) | Slower (full reload) |
| **SEO** | Requires SSR/prerendering | Naturally SEO-friendly |
| **User experience** | App-like, smooth | Traditional, page-based |
| **Server load** | Lower after initial load | Higher (renders every page) |
| **Memory usage** | Higher (app stays in memory) | Lower (pages replaced) |
| **Offline capability** | Possible (service workers) | Difficult |
| **State management** | Client-side | Server-side |
| **Complexity** | Higher (routing, state, bundling) | Lower (standard HTTP) |
| **Code splitting** | Essential (lazy loading routes) | Per-page scripts |

## Performance Comparison

### Initial Load

```
SPA First Load:
TTFB: ────────────────┐ (server responds with shell)
DOM: ─────────────────┤ (minimal HTML)
JS Download: ──────────────────┤ (heavy bundle)
JS Execute: ──────────────────────────┤ (framework init)
App Render: ─────────────────────────────────┤ (first meaningful paint)

MPA First Load:
TTFB: ────────────────┐ (server renders HTML)
DOM: ─────────────────┤ (full HTML)
CSS Download: ─────────────────┤ (stylesheets)
Paint: ────────────────────────┤ (first meaningful paint)
```

**Key insight:** MPAs typically have faster initial loads because the server renders HTML directly. SPAs must download and execute JavaScript before anything renders.

### Navigation

```
SPA Navigation (after initial load):
Click → JS Router → Fetch API Data → Render → Done (fast)

MPA Navigation:
Click → Request → Server Renders → HTML → CSS → JS → Paint → Done (slower)
```

## Hybrid Approaches

Modern frameworks blur the line between SPA and MPA:

### Server-Side Rendering (SSR)

Framework renders the initial page on the server (like MPA), then hydrates into a SPA:

```
1. Server sends fully rendered HTML (like MPA)
2. Browser displays content immediately (fast first paint)
3. JavaScript loads and "hydrates" the page (attaches event handlers)
4. Subsequent navigation is client-side (SPA-like)
```

**Used by:** Next.js, Nuxt, Remix

### Static Site Generation (SSG)

Pre-renders all pages at build time:

```
Build time: Render all pages to static HTML
Serve: Static files (fastest possible load)
Client-side: Hydration for interactivity
```

**Used by:** Gatsby, Astro, Next.js (static pages)

### Islands Architecture

Only interactive components are hydrated; the rest remains static HTML:

```
Server renders full page as static HTML
Only interactive components load JavaScript
Everything else is plain HTML (fastest, most SEO-friendly)
```

**Used by:** Astro, Fresh (Deno)

## Decision Framework

| Choose SPA if | Choose MPA if |
|---------------|---------------|
| Application has rich interactions (dashboards, tools) | Content-heavy (news, blogs, documentation) |
| Users stay for long sessions | Navigation-heavy with short visits |
| SEO is not primary concern (or using SSR) | SEO is critical and SSR is not an option |
| Real-time features needed | Server rendering is simpler to maintain |
| Offline mode is required | Team is smaller, simpler stack preferred |
| Large team with frontend expertise | Content team needs to edit pages directly |

## Key Takeaways

1. **SPA navigation is fast; MPA initial load is fast** — the trade-off is where performance comes from
2. **SSR/SSG bridges the gap** — get MPA-like initial load with SPA-like navigation
3. **SEO favors server-rendered content** — but modern search engines can index JavaScript content (with caveats)
4. **Islands architecture is the future** — minimal JavaScript for maximum performance
5. **The choice depends on use case** — not every app needs SPA-level interactivity

## References

- [MDN: Single-page application](https://developer.mozilla.org/en-US/docs/Glossary/SPA)
- [Next.js: Rendering](https://nextjs.org/docs/basic-features/pages)
- [Astro: Islands Architecture](https://docs.astro.build/en/concepts/islands/)
- [web.dev: Rendering patterns](https://web.dev/rendering-on-the-web/)
