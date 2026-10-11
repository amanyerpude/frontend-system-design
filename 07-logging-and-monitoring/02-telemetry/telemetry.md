# Telemetry

## I. What Is Telemetry?

Telemetry is the **automated collection and transmission of data** from a running application to a central system for analysis. In frontend, telemetry captures real user interactions, performance data, and errors — giving teams visibility into how the application behaves in production.

## II. Types of Frontend Telemetry

### 1. Performance Telemetry

Captures how fast the application loads and responds:

```typescript
import { onLCP, onINP, onCLS, onFCP, onTTFB } from 'web-vitals'

function reportVital(metric) {
  sendTelemetry('web_vital', {
    name: metric.name,        // 'LCP', 'INP', 'CLS', 'FCP', 'TTFB'
    value: metric.value,      // Numeric value
    rating: metric.rating,    // 'good', 'needs-improvement', 'poor'
    id: metric.id,
    navigationType: metric.navigationType,
    page: window.location.pathname,
    device: navigator.userAgent,
  })
}

onLCP(reportVital)
onINP(reportVital)
onCLS(reportVital)
onFCP(reportVital)
onTTFB(reportVital)
```

### 2. Error Telemetry

Captures JavaScript errors, unhandled rejections, and API failures:

```typescript
// Global error handler
window.addEventListener('error', (event) => {
  sendTelemetry('error', {
    type: 'javascript_error',
    message: event.message,
    filename: event.filename,
    lineno: event.lineno,
    colno: event.colno,
    stack: event.error?.stack,
    page: window.location.pathname,
    timestamp: new Date().toISOString(),
  })
})

// Unhandled promise rejections
window.addEventListener('unhandledrejection', (event) => {
  sendTelemetry('error', {
    type: 'unhandled_rejection',
    reason: String(event.reason),
    stack: event.reason?.stack,
    page: window.location.pathname,
  })
})
```

### 3. User Behavior Telemetry

Captures how users interact with the application:

```typescript
// Page views
function trackPageView() {
  sendTelemetry('page_view', {
    url: window.location.href,
    referrer: document.referrer,
    timestamp: new Date().toISOString(),
    sessionId: getSessionId(),
  })
}

// User interactions
function trackClick(element: HTMLElement) {
  sendTelemetry('interaction', {
    type: 'click',
    element: element.tagName,
    text: element.textContent?.slice(0, 50),
    page: window.location.pathname,
    timestamp: new Date().toISOString(),
  })
}

// Form submissions
function trackFormSubmit(formName: string, fields: string[]) {
  sendTelemetry('interaction', {
    type: 'form_submit',
    formName,
    fieldCount: fields.length,
    page: window.location.pathname,
  })
}
```

### 4. Network Telemetry

Captures API call performance and errors:

```typescript
// Intercept fetch
const originalFetch = window.fetch
window.fetch = async (...args) => {
  const startTime = performance.now()
  const url = args[0] instanceof Request ? args[0].url : String(args[0])

  try {
    const response = await originalFetch(...args)
    const duration = performance.now() - startTime

    sendTelemetry('api_call', {
      url,
      method: args[1]?.method || 'GET',
      status: response.status,
      duration: Math.round(duration),
      success: response.ok,
    })

    return response
  } catch (error) {
    const duration = performance.now() - startTime
    sendTelemetry('api_call', {
      url,
      method: args[1]?.method || 'GET',
      status: 0,
      duration: Math.round(duration),
      success: false,
      error: String(error),
    })
    throw error
  }
}
```

## III. Telemetry Collection Approaches

| Approach | Description | Trade-off |
|----------|-------------|-----------|
| **Beacon API** | Fire-and-forget, survives page unload | No response, limited data |
| **sendBeacon()** | Wraps Beacon API, browser-optimized | Not available in all browsers |
| **Fetch with keepalive** | Uses fetch with keepalive option | More control, slightly more overhead |
| **XHR** | Traditional XMLHttpRequest | Blocks thread, legacy |

### Recommended: Beacon API for Page Unload

```typescript
function sendTelemetry(type: string, data: any) {
  const payload = JSON.stringify({ type, data, timestamp: Date.now() })

  // Use sendBeacon — survives page unload, fire-and-forget
  if ('sendBeacon' in navigator) {
    navigator.sendBeacon('/api/telemetry', payload)
  } else {
    // Fallback: fetch with keepalive
    fetch('/api/telemetry', {
      method: 'POST',
      body: payload,
      keepalive: true,
      headers: { 'Content-Type': 'application/json' },
    })
  }
}
```

## IV. Sampling

Not every event needs to be sent. Sampling reduces volume and cost:

```typescript
// Rate-based sampling (send 10% of events)
function shouldSample(rate: number): boolean {
  return Math.random() < rate
}

// Performance telemetry: send all poor ratings, sample good ratings
function reportVital(metric) {
  const shouldSend = metric.rating === 'poor' || shouldSample(0.1)
  if (shouldSend) {
    sendTelemetry('web_vital', { ... })
  }
}

// Error telemetry: send all errors (they're rare and important)
// Interaction telemetry: sample at 5%
// Page view telemetry: send all
```

## V. Telemetry Data Pipeline

```
Browser (Client)
    │
    ├── sendBeacon / fetch
    │
    ▼
Ingestion Endpoint (HTTP)
    │
    ├── Batch and queue
    │
    ▼
Processing Pipeline
    │
    ├── Parse, enrich, aggregate
    │
    ▼
Storage
    │
    ├── Time-series DB (metrics)
    │
    ├── Log aggregation (logs)
    │
    └── Trace store (traces)
    │
    ▼
Visualization & Alerting
    │
    ├── Dashboards
    │
    └── Alerts
```

## VI. Popular Telemetry Tools

| Category | Tools |
|----------|-------|
| **Error tracking** | Sentry, Bugsnag, Rollbar |
| **Performance** | SpeedCurve, Datadog RUM, New Relic |
| **Session replay** | LogRocket, FullStory, Hotjar |
| **Analytics** | Google Analytics, Mixpanel, Amplitude |
| **Custom pipeline** | OpenTelemetry, Segment, RudderStack |

### OpenTelemetry for JavaScript

```typescript
import { WebTracerProvider } from '@opentelemetry/sdk-trace-web'
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http'

const provider = new WebTracerProvider()
provider.addSpanProcessor(
  new SimpleSpanProcessor(
    new OTLPTraceExporter({ url: 'https://otel-collector.example.com/v1/traces' })
  )
)
provider.register()

const tracer = provider.getTracer('my-app')
const span = tracer.startSpan('page_load')
// ... work ...
span.end()
```

## VII. Key Takeaways

1. **Telemetry is automated data collection** — performance, errors, behavior, network
2. **Use `sendBeacon`** for telemetry that must survive page unload
3. **Sample aggressively** for high-volume events; send all errors
4. **Scrub PII** before sending; respect user consent
5. **OpenTelemetry** is the industry standard for vendor-neutral telemetry

## References

- [web-vitals Library](https://github.com/GoogleChrome/web-vitals)
- [OpenTelemetry JS](https://opentelemetry.io/docs/languages/js/)
- [MDN: Beacon API](https://developer.mozilla.org/en-US/docs/Web/API/Beacon_API)
- [Sentry JavaScript SDK](https://docs.sentry.io/platforms/javascript/)
