# Logging & Monitoring Overview

## I. What Is Logging & Monitoring?

**Logging** is the practice of recording events, errors, and user interactions in an application. **Monitoring** is the practice of collecting, visualizing, and alerting on those records to detect issues before users notice them. Together, they form the **observability** layer of a production frontend application.

Without logging and monitoring, frontend teams operate blind — they rely on user reports to discover bugs, performance degradation, and outages. With it, they can detect, diagnose, and fix issues proactively.

## II. The Observability Stack

```
┌──────────────────────────────────────────────────────────┐
│                    Observability Stack                    │
│                                                          │
│  Logs          Metrics           Traces                  │
│  ────          ───────           ─────                   │
│  Event         Aggregated        Request                 │
│  records       numeric           propagation             │
│  (strings)     measurements      across services         │
│                                                          │
│  What          How much/when     Where did the            │
│  happened      is happening      request go               │
└──────────────────────────────────────────────────────────┘
```

| Signal | Purpose | Frontend Example |
|--------|---------|-----------------|
| **Logs** | Discrete events with context | "User clicked checkout button" |
| **Metrics** | Aggregated numeric measurements | "LCP = 3.2s across 1000 page loads" |
| **Traces** | Request lifecycle across services | "API call from component → fetch → backend" |

## III. What to Log in a Frontend Application

### Error Logging

| Event | What to Capture |
|-------|----------------|
| **JavaScript errors** | Error message, stack trace, component tree, user action |
| **Unhandled rejections** | Promise failure with context |
| **API errors** | Endpoint, status code, request payload (sanitized) |
| **Render errors** | Component name, props, error boundary context |

### Performance Logging

| Event | What to Capture |
|-------|----------------|
| **Core Web Vitals** | LCP, INP, CLS, FCP values per page |
| **Long tasks** | Duration, source script |
| **Resource timing** | Slow API calls, large assets |
| **Memory usage** | Heap size, GC pauses |

### User Behavior Logging

| Event | What to Capture |
|-------|----------------|
| **Page views** | URL, referrer, user agent |
| **Interactions** | Click, scroll, form submission (with consent) |
| **Funnels** | Multi-step flow progress and drop-off |
| **Feature usage** | Which features are used, how often |

## IV. Logging Levels

| Level | Purpose | Example |
|-------|---------|---------|
| **Debug** | Detailed diagnostic info for developers | "Fetching user data for ID 42" |
| **Info** | General application events | "User logged in successfully" |
| **Warn** | Something unexpected but not an error | "API response slow (>2s)" |
| **Error** | Something failed but app can continue | "Failed to load avatar image" |
| **Fatal** | Application cannot continue | "Unhandled exception in root component" |

### Structured Logging

```typescript
// ❌ BAD — unstructured string
console.log('User clicked checkout')

// ✅ GOOD — structured, searchable, actionable
logger.info('checkout_started', {
  userId: 'user_123',
  cartId: 'cart_456',
  itemCount: 3,
  totalAmount: 89.99,
  page: '/checkout',
  timestamp: new Date().toISOString(),
})
```

## V. The Three Pillars of Frontend Observability

### 1. Error Tracking

Captures JavaScript errors, unhandled rejections, and API failures with full context (stack trace, component tree, user session).

**Tools:** Sentry, Bugsnag, Rollbar, LogRocket

### 2. Performance Monitoring

Captures Core Web Vitals, long tasks, and resource timing from real users.

**Tools:** SpeedCurve, Datadog RUM, New Relic Browser, web-vitals

### 3. Session Replay

Records user sessions (DOM mutations, clicks, network) to replay exactly what happened before an error.

**Tools:** LogRocket, FullStory, Hotjar, Datadog Session Replay

## VI. Client-Side vs. Server-Side Logging

| Aspect | Client-Side | Server-Side |
|--------|-------------|-------------|
| **Environment** | Browser (user's device) | Server |
| **Reliability** | Can be blocked, offline, or manipulated | More reliable |
| **Volume control** | Sample to reduce noise | Log everything |
| **Sensitive data** | Risk of exposing to client | Safer for PII |
| **Real-time** | Can be delayed | Immediate |
| **Context** | Rich (DOM state, user action) | Limited to request data |

## VII. Privacy and Consent

Frontend logging must respect user privacy:

- **GDPR/CCPA compliance** — obtain consent before logging personal data
- **PII scrubbing** — automatically remove emails, passwords, credit cards from logs
- **Data minimization** — log only what's needed for debugging
- **Anonymization** — hash user IDs when individual tracking isn't needed

```typescript
// PII scrubbing middleware
logger.addProcessor((entry) => {
  entry.message = entry.message
    .replace(/[\w.+-]+@[\w-]+\.[\w.]+/g, '[EMAIL]')
    .replace(/\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b/g, '[CARD]')
  return entry
})
```

## VIII. Key Takeaways

1. **Logs, metrics, and traces** are three complementary observability signals
2. **Structured logging** (JSON) is searchable; unstructured strings are not
3. **Log errors, performance, and user behavior** — each answers different questions
4. **Respect privacy** — scrub PII, get consent, minimize data
5. **Frontend logging is inherently less reliable** than server-side — plan for gaps

## References

- [OpenTelemetry for JavaScript](https://opentelemetry.io/docs/languages/js/)
- [Sentry JavaScript Docs](https://docs.sentry.io/platforms/javascript/)
- [web.dev: Logging and Monitoring](https://web.dev/learn/pwa/)
