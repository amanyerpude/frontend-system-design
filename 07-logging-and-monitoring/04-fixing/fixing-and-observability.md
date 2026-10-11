# Fixing Issues (Observability → Action)

## I. The Observability-to-Action Pipeline

Collecting logs, telemetry, and alerts is only useful if it leads to **fixing problems**. The pipeline from detection to resolution follows a clear path:

```
Detection → Diagnosis → Resolution → Prevention
```

## II. Detection

### How Issues Are Detected

| Source | Detection Method |
|--------|-----------------|
| **Error tracking** | Real-time alerts on error rate spike |
| **Performance monitoring** | Core Web Vitals threshold breach |
| **User reports** | Support tickets, social media |
| **Logs** | Anomaly detection on log patterns |
| **Synthetic monitoring** | Automated checks from multiple locations |

### Prioritizing Detected Issues

```
Impact Matrix:

                    High Impact
                         │
         ┌───────────────┼───────────────┐
         │   P1: High    │  P0: Critical │
         │   Fix today   │  Fix now      │
         │               │               │
Low ─────┼───────────────┼───────────────┼───── High
Frequency│               │               │ Frequency
         │   P3: Low     │  P2: Medium   │
         │   Fix when    │  Fix this     │
         │   convenient  │  week         │
         └───────────────┼───────────────┘
                         │
                    Low Impact
```

## III. Diagnosis

### Frontend Debugging Workflow

```
1. IDENTIFY the error type
   ├── JavaScript exception (SyntaxError, TypeError, etc.)
   ├── API failure (network error, 4xx/5xx)
   ├── Performance regression (slow LCP, high CLS)
   └── Rendering issue (blank screen, layout broken)

2. REPRODUCE the error
   ├── Use session replay (LogRocket, FullStory)
   ├── Check error context (component tree, state)
   ├── Reproduce locally with same conditions
   └   └── Use browser DevTools

3. ISOLATE the cause
   ├── Bisect recent changes (git bisect)
   ├── Check dependency updates
   ├── Review deployment diff
   └── Check external services (API, CDN)

4. FIX and VERIFY
   ├── Write a failing test that reproduces the bug
   ├── Fix the code
   ├── Verify the test passes
   └── Deploy and monitor for resolution
```

### Using Session Replay for Diagnosis

Session replay records what the user saw and did before an error:

```
Before an error occurs:
1. User navigated to /checkout
2. Filled shipping form
3. Clicked "Pay Now"
4. API returned 500
5. Error boundary caught crash

With session replay, you see:
- Exact sequence of clicks
- Network request/response
- Console errors
- DOM state at time of error
```

### Using Error Boundaries for Context

```typescript
// React Error Boundary — captures component context on error
class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { error: Error | null }
> {
  state = { error: null }

  static getDerivedStateFromError(error: Error) {
    return { error }
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Send rich context to error tracker
    sendTelemetry('error', {
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      props: this.props,  // Be careful with sensitive props
      route: window.location.pathname,
      timestamp: new Date().toISOString(),
    })
  }

  render() {
    if (this.state.error) {
      return <ErrorFallback error={this.state.error} />
    }
    return this.props.children
  }
}
```

## IV. Resolution

### Hotfix Process

```
1. IDENTIFY the broken commit
   ├── Check deployment logs
   ├── Use error tracking to find first occurrence
   └── git bisect to find exact commit

2. REVERT or PATCH
   ├── Fastest: revert the offending commit
   └── Alternative: write a targeted fix

3. TEST locally
   ├── Verify fix resolves the issue
   ├── Run full test suite
   └── Test on staging environment

4. DEPLOY
   ├── Deploy to production
   ├── Monitor error rates post-deploy
   └── Confirm issue is resolved

5. POST-MORTEM (for P0/P1)
   ├── What happened?
   ├── Why did it happen?
   ├── How was it detected?
   ├── What was the impact?
   └── What prevents recurrence?
```

### Feature Flags for Rapid Rollback

```typescript
// Deploy behind a feature flag — disable without redeploying
function NewCheckout() {
  const enabled = useFeatureFlag('new-checkout')
  return enabled ? <NewCheckoutFlow /> : <OldCheckoutFlow />
}

// If new checkout has issues, disable via dashboard:
// featureFlags.set('new-checkout', false)
// Users instantly get old checkout — no deploy needed
```

## V. Prevention

### Post-Mortem Template

```markdown
# Post-Mortem: Checkout API 500 Errors

## Summary
On [date], checkout API returned 500 errors for 12% of users for 45 minutes.

## Impact
- 2,400 failed checkouts
- ~$48,000 estimated lost revenue
- 45 minutes of degraded service

## Root Cause
Database connection pool exhaustion caused by unbounded concurrent requests from new checkout flow.

## Detection
- Detected by: Sentry error rate alert (2 minutes after onset)
- Alerted: Slack #frontend-alerts, PagerDuty oncall

## Timeline
- 14:03 — New checkout deployed
- 14:05 — First error reported by Sentry
- 14:07 — Alert fired, oncall paged
- 14:23 — Root cause identified (connection pool)
- 14:48 — Feature flag disabled, errors stopped

## Action Items
- [ ] Add connection pool monitoring (owner: @backend, due: Friday)
- [ ] Add rate limiting to checkout API (owner: @backend, due: next sprint)
- [ ] Add canary deployment for checkout (owner: @platform, due: next sprint)
```

### Prevention Checklist

| Category | Prevention |
|----------|-----------|
| **Code quality** | Code review, static analysis, TypeScript strict mode |
| **Testing** | Unit tests, integration tests, E2E tests for critical flows |
| **Deployment** | Canary deployments, feature flags, automated rollback |
| **Monitoring** | Error tracking, performance monitoring, synthetic checks |
| **Process** | Post-mortems, runbooks, on-call rotation |

## VI. Key Takeaways

1. **Observability without action is wasted effort** — always close the loop
2. **Session replay** is the fastest way to diagnose frontend issues
3. **Error boundaries** provide rich component context for diagnosis
4. **Feature flags** enable rapid rollback without redeployment
5. **Post-mortems** prevent recurrence — focus on systems, not blame

## References

- [Google SRE: Postmortem Culture](https://sre.google/workbook/postmortem-analysis/)
- [Sentry: Debugging with Session Replay](https://docs.sentry.io/platforms/javascript/session-replay/)
- [Feature Flags: LaunchDarkly](https://launchdarkly.com/)
