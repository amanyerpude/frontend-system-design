# Alerting

## I. What Is Alerting?

Alerting is the practice of **automatically notifying the right people when monitored metrics cross defined thresholds**. It's the bridge between observation (logs, metrics, telemetry) and action (fixing the problem). Good alerting catches issues before users notice; bad alerting causes alert fatigue and missed critical incidents.

## II. Alert Design Principles

| Principle | Description |
|-----------|-------------|
| **Actionable** | Every alert must require a specific action from a specific team |
| **Specific** | "Error rate high" is bad; "Checkout API error rate >5% for 5min" is good |
| **Timely** | Alert before users notice, not after |
| **Low noise** | Avoid alert fatigue — page humans only for critical issues |
| **Documented** | Every alert runbook tells the responder what to do |

## III. Alert Severity Levels

| Level | Response | Examples |
|-------|----------|---------|
| **P0 (Critical)** | Page immediately (SMS, phone call) | Site down, payment system broken, data loss |
| **P1 (High)** | Page during business hours | Major feature broken, significant error rate increase |
| **P2 (Medium)** | Ticket / Slack notification | Performance degradation, non-critical feature broken |
| **P3 (Low)** | Dashboard / weekly review | Minor issue, slow degradation, tech debt |

## IV. Frontend Alert Metrics

### Error Rate

```typescript
// Alert: JavaScript error rate exceeds 1% of sessions
// Condition: error_sessions / total_sessions > 0.01 for 5 minutes
// Severity: P1
```

### Performance Degradation

```typescript
// Alert: LCP p75 exceeds 4 seconds
// Condition: p75(LCP) > 4000ms for 15 minutes
// Severity: P1 (poor Core Web Vitals impact SEO and UX)
```

### API Failure Rate

```typescript
// Alert: Checkout API failure rate exceeds 5%
// Condition: failed_requests / total_requests > 0.05 for 5 minutes
// Severity: P0 (direct revenue impact)
```

### JavaScript Bundle Size

```typescript
// Alert: Bundle size increases by >10% on any deploy
// Condition: bundle_size_after_deploy > bundle_size_before * 1.10
// Severity: P2 (performance regression)
```

## V. Alert Configuration Example

```yaml
# alerting-config.yaml (Datadog/PagerDuty-style)
alerts:
  - name: "high_error_rate"
    metric: "frontend.error_rate"
    threshold: 0.01          # 1%
    window: "5m"
    severity: "P1"
    channels: ["slack-frontend", "pagerduty-oncall"]
    runbook: "https://runbooks.example.com/high-error-rate"

  - name: "lcp_degraded"
    metric: "web_vital.lcp_p75"
    threshold: 4000           # 4 seconds
    window: "15m"
    severity: "P1"
    channels: ["slack-frontend"]
    runbook: "https://runbooks.example.com/lcp-degraded"

  - name: "checkout_api_failures"
    metric: "api.checkout.error_rate"
    threshold: 0.05           # 5%
    window: "5m"
    severity: "P0"
    channels: ["pagerduty-oncall", "slack-payments"]
    runbook: "https://runbooks.example.com/checkout-failures"

  - name: "bundle_size_increase"
    metric: "build.bundle_size_delta"
    threshold: 0.10           # 10% increase
    window: "1 deploy"
    severity: "P2"
    channels: ["slack-frontend"]
```

## VI. Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| **Alert on symptoms, not causes** | "CPU high" doesn't tell you what to fix | Alert on user-facing symptoms (error rate, latency) |
| **No thresholds (always alerting)** | Constant noise → alert fatigue | Set data-driven thresholds with hysteresis |
| **Alert storms** | One issue triggers 50 alerts | Group related alerts; use deduplication |
| **No runbooks** | Responder doesn't know what to do | Every alert links to a runbook with steps |
| **Alert on every metric** | Only alert on what matters to users | Focus on user-facing SLIs |
| **No escalation** | P0 alerts go to a Slack channel no one watches | P0 must page a human via SMS/phone |

## VII. Alert Fatigue

Alert fatigue occurs when responders receive so many alerts that they start ignoring them. It's the most common failure mode in alerting.

**Causes:**
- Too many low-severity alerts
- Noisy thresholds (flapping)
- Alerts for problems no one can fix
- No alert review process

**Solutions:**
- Review and tune alerts monthly
- Delete alerts that haven't fired in 90 days
- Increase thresholds for noisy metrics
- Use anomaly detection instead of static thresholds

## VIII. Key Takeaways

1. **Alert on user-facing symptoms** — error rate, latency, Core Web Vitals
2. **Every alert needs a runbook** — a human should know what to do
3. **Page only for P0/P1** — everything else goes to Slack/tickets
4. **Review alerts monthly** — delete stale ones, tune noisy ones
5. **Prevent alert fatigue** — noise is worse than no alerting

## References

- [Google SRE: Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
- [PagerDuty: Alerting Best Practices](https://www.pagerduty.com/resources/learn/alerting-best-practices/)
- [Datadog: Frontend Monitoring](https://docs.datadoghq.com/real_user_monitoring/)
