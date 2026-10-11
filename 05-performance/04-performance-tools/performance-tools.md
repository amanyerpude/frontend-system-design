# Performance Tools

## The Performance Tool Ecosystem

Performance testing and measurement require the right tool for the right job. The ecosystem spans lab testing (synthetic), field testing (real users), profiling, and continuous monitoring.

## Lab Testing Tools (Synthetic)

Lab tests measure performance in a controlled environment with simulated network and device conditions.

### Chrome DevTools Performance Panel

The most immediate tool — record and analyze runtime performance in the browser.

**What it reveals:**
- **Flame charts** — Shows function call stacks and execution time
- **Main thread activity** — Long tasks blocking the main thread
- **Layout thrashing** — Repeated forced synchronous reflows
- **Network waterfall** — Request timing, blocking, priority
- **Memory snapshots** — Heap allocations, garbage collection pauses

**How to use:**
1. Open DevTools → Performance tab
2. Click Record (or press Cmd/Ctrl + Shift + E)
3. Interact with the page
4. Stop recording and analyze the timeline

**Key things to look for:**
- Long tasks (>50ms) that block interaction
- Purple bars (layout) and green bars (paint) clustered together
- Repeated layout-paint cycles (layout thrashing)

### Lighthouse

Automated performance auditing built into Chrome DevTools. Runs a series of audits and produces a 0–100 score.

**What it measures:**
- Performance (Core Web Vitals)
- Accessibility
- Best Practices
- SEO
- PWA

```bash
# Lighthouse CLI
npx lighthouse https://example.com \
  --output=json \
  --output-path=./report.json \
  --chrome-flags="--headless --no-sandbox"

# Lighthouse in CI
npm install --save-dev @lhci/cli
lhci autorun
```

### WebPageTest

Advanced lab testing with real device emulation, multi-step tests, and detailed waterfall analysis.

**Strengths over Lighthouse:**
- Test from real locations worldwide
- Test on real devices (iPhone, Android)
- Multi-step transaction recording (login → navigate → purchase)
- Connection throttling (3G, 4G, cable)
- Filmstrip view (visual loading progression)

### SpeedCurve

Continuous lab testing across multiple pages and locations, with historical tracking and comparison dashboards.

## Field Testing Tools (RUM)

RUM measures real user experience across actual devices, networks, and conditions.

### web-vitals Library

Google's lightweight JavaScript library for measuring Core Web Vitals in production.

```typescript
import { onLCP, onINP, onCLS, onFCP, onTTFB } from 'web-vitals'

function reportMetric(metric) {
  navigator.sendBeacon('/api/vitals', JSON.stringify({
    name: metric.name,
    value: metric.value,
    rating: metric.rating,  // 'good' | 'needs-improvement' | 'poor'
    id: metric.id,
    navigationType: metric.navigationType,
  }))
}

onLCP(reportMetric)
onINP(reportMetric)
onCLS(reportMetric)
onFCP(reportMetric)
onTTFB(reportMetric)
```

### Commercial RUM Platforms

| Platform | Key Feature |
|----------|-------------|
| **SpeedCurve** | Lab + RUM, comparison dashboards |
| **Datadog RUM** | Integrated with APM and logs |
| **New Relic Browser** | Full-stack observability |
| **Vercel Speed Insights** | Built-in for Vercel deployments |
| **Cloudflare Web Analytics** | Privacy-focused, free |

## Profiling Tools

### Chrome DevTools Memory Panel

- **Heap snapshots** — Compare memory between two points to find leaks
- **Allocation instrumentation** — Track allocations over time
- **Allocation sampling** — Lightweight sampling profiler

### React DevTools Profiler

Profiles React component renders:
- Which components re-rendered and why
- Render duration per component
- Flame chart of render hierarchy
- "Highlight updates" mode — visual flash on re-render

### Why Did You Render?

Tracks unnecessary re-renders in React:

```typescript
import { wdry } from '@welldone-software/why-did-you-render'
wdry(React, {
  trackAllPureComponents: true,
})
```

### Bundler Analysis Tools

| Tool | Bundler | Purpose |
|------|---------|---------|
| **webpack-bundle-analyzer** | webpack | Visual treemap of bundle contents |
| **rollup-plugin-visualizer** | Rollup/Vite | Interactive bundle analysis |
| **source-map-explorer** | Any | Analyze bundle using source maps |
| **esbuild-visualizer** | esbuild | Bundle size visualization |

## CI/CD Integration Tools

### Lighthouse CI

```yaml
# .lighthouserc.js
module.exports = {
  ci: {
    collect: {
      url: ['http://localhost:3000/', 'http://localhost:3000/about'],
      numberOfRuns: 3,
    },
    assert: {
      assertions: {
        'largest-contentful-paint': ['error', { maxNumericValue: 2500 }],
        'cumulative-layout-shift': ['error', { maxNumericValue: 0.1 }],
        'first-contentful-paint': ['error', { maxNumericValue: 1800 }],
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
}
```

### Bundlewatch

Tracks bundle size over time and fails CI when size increases beyond a threshold.

```json
// bundlewatch.config.json
{
  "files": [
    { "path": "dist/*.js", "maxSize": "300KB" },
    { "path": "dist/*.css", "maxSize": "100KB" }
  ]
}
```

### Size Limit

Precise bundle size measurement with actionable error messages:

```json
// package.json
{
  "size-limit": [
    {
      "path": "dist/index.js",
      "limit": "300 KB"
    }
  ]
}
```

## Tool Selection Guide

| Need | Tool |
|------|------|
| Quick performance check | Lighthouse (DevTools) |
| Runtime profiling | Chrome DevTools Performance |
| Detailed loading analysis | WebPageTest |
| Real user monitoring | web-vitals + RUM platform |
| React render profiling | React DevTools Profiler |
| Bundle analysis | webpack-bundle-analyzer / source-map-explorer |
| CI performance gate | Lighthouse CI |
| Bundle size tracking | Size Limit / Bundlewatch |
| Memory leak detection | Chrome DevTools Memory |
| Continuous monitoring | SpeedCurve / Datadog |

## References

- [Chrome DevTools Performance](https://developer.chrome.com/docs/devtools/performance/)
- [Lighthouse](https://developer.chrome.com/docs/lighthouse/overview)
- [WebPageTest](https://www.webpagetest.org/)
- [web-vitals Library](https://github.com/GoogleChrome/web-vitals)
- [Size Limit](https://github.com/ai/size-limit)
