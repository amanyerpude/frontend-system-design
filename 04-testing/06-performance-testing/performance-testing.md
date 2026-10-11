# Performance Testing

## Why Performance Testing Matters

Performance directly impacts user behavior, revenue, and search rankings. Studies consistently show:
- **53% of users abandon** a page that takes longer than 3 seconds to load
- **Google uses Core Web Vitals** as a ranking signal in search results
- **Every 100ms of latency** can cost conversion rate for e-commerce sites

Performance testing measures whether the application meets these thresholds — and stays meeting them as the codebase grows. Without continuous testing, performance **decays silently** with every feature addition.

## The Four Core Web Vitals

Core Web Vitals are Google's standardized metrics for measuring user-perceived performance. They are the primary targets for any frontend performance work.

| Metric | Full Name | Measures | Good | Needs Work | Poor |
|--------|-----------|----------|------|------------|------|
| **LCP** | Largest Contentful Paint | When the main content becomes visible (loading) | ≤ 2.5s | 2.5–4.0s | > 4.0s |
| **INP** | Interaction to Next Paint | Responsiveness to user input (interactivity) | ≤ 200ms | 200–500ms | > 500ms |
| **CLS** | Cumulative Layout Shift | Visual stability of the page (layout) | ≤ 0.1 | 0.1–0.25 | > 0.25 |
| **FCP** | First Contentful Paint | When anything first appears on screen | ≤ 1.8s | 1.8–3.0s | > 3.0s |

**Key context:**
- **LCP** replaced FID (First Input Delay) in March 2024 as the interactivity metric because INP better captures the full interaction experience
- **INP** is measured at the 98th percentile of interactions (worst-case, not average)
- **CLS** only counts unexpected layout shifts; shifts caused by user interaction are excluded

### What Each Metric Actually Measures

```
Timeline:
───────────────────────────────────────────────────────────►
│                    │                    │        │
FCP                  LCP                  INP     Page
(first paint)       (main content)     (responds)  stable
                     ◄──── LCP ────►
                     (loading time)
```

## Testing Approaches

### 1. Lab Testing (Synthetic)

Lab testing measures performance in a controlled environment — typically a CI pipeline or developer machine with simulated network and device conditions.

**Tools:**
- **Lighthouse** — Integrated into Chrome DevTools; runs audits for performance, accessibility, SEO, and best practices
- **Lighthouse CI** — Automates Lighthouse in CI/CD, tracks scores per PR
- **WebPageTest** — Advanced lab testing with real device/emulated networks, waterfall analysis, multi-step testing

**Lighthouse scores** are a 0–100 composite based on multiple metrics. A score of **90+ is excellent**; 100 often has diminishing returns.

```bash
# Lighthouse CLI
npx lighthouse https://example.com \
  --output=json \
  --output-path=./lighthouse-report.json \
  --chrome-flags="--headless --no-sandbox"
```

### 2. Field Testing (Real User Monitoring — RUM)

Lab tests are controlled but artificial. Field testing measures real user experience across actual devices, networks, and conditions.

**Tools:**
- **SpeedCurve** — Continuous RUM monitoring with comparison dashboards
- **Datadog RUM** — Real user monitoring integrated with APM
- **Vercel Speed Insights** — RUM for Vercel-deployed apps
- **web-vitals npm package** — Google's lightweight library for in-app measurement

```typescript
// web-vitals library — report Core Web Vitals from real users
import { onLCP, onINP, onCLS, onFCP } from 'web-vitals'

function sendToAnalytics(metric) {
  const body = JSON.stringify({
    name: metric.name,
    value: metric.value,
    rating: metric.rating,    // 'good' | 'needs-improvement' | 'poor'
    delta: metric.delta,
    id: metric.id,
    navigationType: metric.navigationType,
  })

  // Use `navigator.sendBeacon()` for zero-impact reporting
  navigator.sendBeacon('/api/vitals', body)
}

onLCP(sendToAnalytics)
onINP(sendToAnalytics)
onCLS(sendToAnalytics)
onFCP(sendToAnalytics)
```

### 3. Performance Budgets

Performance budgets set explicit thresholds that CI enforces. If a build exceeds the budget, the pipeline fails or warns.

```json
// lighthouse-budget.json
[
  {
    "path": "/*",
    "resourceSizes": [
      { "resourceType": "script", "budget": 200 },
      { "resourceType": "stylesheet", "budget": 100 },
      { "resourceType": "image", "budget": 500 },
      { "resourceType": "total", "budget": 1000 }
    ],
    "timings": [
      { "metric": "first-contentful-paint", "budget": 1800 },
      { "metric": "largest-contentful-paint", "budget": 2500 },
      { "metric": "cumulative-layout-shift", "budget": 0.1 }
    ]
  }
]
```

```yaml
# GitHub Actions with Lighthouse CI
name: Performance
on: [pull_request]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci && npm run build

      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v10
        with:
          urls: |
            http://localhost:3000/
            http://localhost:3000/checkout
          budgetPath: ./lighthouse-budget.json
          uploadArtifacts: true
```

### 4. In-App Performance Markers

The Performance API provides fine-grained timing for application-specific code:

```typescript
// Mark and measure application-specific performance
performance.mark('app-init-start')
await initializeApp()
performance.mark('app-init-end')
performance.measure('app-init', 'app-init-start', 'app-init-end')

const entry = performance.getEntriesByName('app-init')[0]
console.log(`App init took ${entry.duration}ms`)

// Web Performance Observer for long tasks (main thread blocking)
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 200) {  // Long task > 200ms
      console.warn(`Long task: ${entry.duration}ms`, entry)
    }
  }
})
observer.observe({ entryTypes: ['longtask'] })
```

## LCP Optimization (Largest Contentful Paint)

LCP is usually the hero image, a headline, or a video thumbnail. The biggest levers:

| Technique | Impact | How |
|-----------|--------|-----|
| **Image optimization** | High | WebP/AVIF format, responsive sizes, lazy loading below fold |
| **Preload critical resources** | High | `<link rel="preload">` for LCP image |
| **Reduce render-blocking** | Medium | Defer non-critical CSS, async scripts |
| **Server response time (TTFB)** | High | Caching, CDN, server-side rendering |
| **Font loading** | Medium | `font-display: swap`, preload fonts |

```html
<!-- Preload the LCP image for faster loading -->
<link rel="preload" as="image" href="/hero.webp" type="image/webp">
```

## INP Optimization (Interaction to Next Paint)

INP measures how quickly the UI responds after user input. The biggest lever is reducing JavaScript execution time:

| Technique | Impact | How |
|-----------|--------|-----|
| **Reduce JavaScript bundle** | High | Code splitting, tree shaking, remove unused deps |
| **Defer non-critical JS** | High | `requestIdleCallback`, dynamic imports |
| **Optimize event handlers** | Medium | Debounce heavy handlers, use `startTransition` in React 18+ |
| **Web Workers** | Medium | Offload heavy computation off the main thread |
| **Break up long tasks** | High | Split work into chunks to avoid blocking main thread |

```typescript
// React 18 — mark expensive updates as non-urgent
import { startTransition } from 'react'

function SearchComponent({ query, setQuery }) {
  function handleChange(e) {
    // Urgent: update input immediately
    setQuery(e.target.value)

    // Non-urgent: defer the heavy search/list update
    startTransition(() => {
      performHeavySearch(e.target.value)
    })
  }

  return <input onChange={handleChange} />
}
```

## CLS Optimization (Cumulative Layout Shift)

CLS is caused by elements that shift position after initial render. The main causes:

| Cause | Solution |
|-------|----------|
| **Images without dimensions** | Always set `width`/`height` attributes (or CSS `aspect-ratio`) |
| **Ads/embeds injecting content** | Reserve space with min-height placeholders |
| **Web fonts causing reflow** | Use `font-display: swap` with size-adjusted fallbacks |
| **Dynamic content insertion** | Insert new content below existing content when possible |

```html
<!-- ✅ Good — dimensions prevent layout shift -->
<img src="photo.webp" width="800" height="600" alt="Product photo">

<!-- ❌ Bad — causes CLS as image loads -->
<img src="photo.webp" alt="Product photo">
```

## Sustainable Performance Testing

Performance testing is effective only when it's **continuous**, not one-off:

| Pattern | Why It Works |
|---------|-------------|
| **Performance budgets in CI** | Blocks regressions from merging; makes performance a build criterion |
| **Real user monitoring** | Synthetic tests miss real-device, real-network issues |
| **Performance in PR review** | Reviewers check bundle size and timing impact; prevents drift |
| **Regular audits** | Weekly RUM review catches trends before they become problems |

## Common Performance Testing Mistakes

| Mistake | Why It's Wrong |
|---------|---------------|
| **Testing only on dev machines** | Dev machines are fast; real users have slow devices and 3G connections |
| **Treating performance as one-off** | Performance degrades with every feature; needs continuous monitoring |
| **Ignoring real user data** | Lab tests are controlled; field data reflects reality |
| **Over-optimizing without measurement** | Premature optimization wastes time; measure first, optimize the biggest bottleneck |
| **Chasing Lighthouse 100** | 90+ is excellent; 100 often has diminishing returns for actual users |
| **Ignoring mobile** | Mobile performance differs substantially; test both |

## Interview Context

In a frontend system design interview, performance testing comes up when:
- Designing a dashboard or media-heavy page (loading performance)
- Discussing how to ensure quality at scale (CI/CD, budgets)
- Comparing lab testing vs. field testing strategies
- Talking about monitoring and observability

**Key talking points:**
- Four Core Web Vitals and their thresholds
- Lab (synthetic) vs. field (RUM) — both are needed, neither alone is sufficient
- Performance budgets in CI prevent regressions
- Biggest levers: image optimization (LCP), JS reduction (INP), image dimensions (CLS)

## References

- [web.dev: Core Web Vitals](https://web.dev/vitals/)
- [Lighthouse Documentation](https://developer.chrome.com/docs/lighthouse/overview)
- [web-vitals npm package](https://github.com/GoogleChrome/web-vitals)
- [WebPageTest](https://www.webpagetest.org/)
- [Vite Performance Guide](https://vitejs.dev/guide/performance)
