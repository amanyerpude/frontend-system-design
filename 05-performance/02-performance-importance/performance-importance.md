# Performance Importance

## Why Performance Is a Business Metric

Web performance is not a technical luxury — it directly impacts revenue, user retention, search rankings, and brand perception. Every millisecond of delay has measurable business consequences.

### The Cost of Slow Performance

| Study | Finding |
|-------|---------|
| **Amazon** | A 100ms delay in load time → 1% drop in sales |
| **Google** | 53% of mobile users abandon a site that takes >3 seconds to load |
| **Walmart** | A 1-second improvement in speed → 2% increase in conversions |
| **Pinterest** | 40% improvement in load time → 15% increase in signups |
| **Shopify** | 100ms improvement → 1% increase in conversion |

These are not edge cases. They represent how real users behave across the web.

## Loading Speed vs. UI Speed

Performance must be evaluated in two directions:

| Dimension | What It Measures | Key Metric |
|-----------|-----------------|------------|
| **Loading Speed** | How fast the application loads | LCP, FCP, TTFB |
| **UI Speed** | How fast the application responds to interaction | INP, TTI |

An application is only truly performant when **both** are fast. A site that loads in 1 second but takes 500ms to respond to every click feels slow. A site that loads in 5 seconds but then runs smoothly also feels slow.

### When Loading Speed Is Critical

| Factor | Description |
|--------|-------------|
| **Competitive market** | Users have alternatives; slow loading drives them away |
| **Mobile users** | Poor connections amplify loading delays |
| **Conversion-dependent revenue** | Every second of delay directly costs revenue |
| **Impulse purchases** | Friction kills the purchase impulse |
| **High traffic** | At scale, even small percentage drops represent large absolute losses |

### When UI Speed Is More Critical

- **Dashboards and tools** — Users interact repeatedly; response lag compounds
- **Real-time applications** — Chat, collaboration, live data — UI responsiveness is the product
- **Long-session apps** — Users who stay for hours feel every interaction delay

## The Performance Budget

A performance budget sets explicit thresholds that the team commits to not exceeding. It translates abstract "be fast" goals into measurable constraints.

```json
// Performance budget example
{
  "budgets": {
    "first-contentful-paint": "1800ms",
    "largest-contentful-paint": "2500ms",
    "cumulative-layout-shift": "0.1",
    "interaction-to-next-paint": "200ms",
    "total-byte-weight": "1600KB",
    "script-size": "300KB",
    "image-size": "500KB"
  }
}
```

Budgets should be enforced in CI/CD — a build that exceeds the budget fails or triggers a warning.

## The ROI of Performance Work

Performance optimization follows diminishing returns. The first improvements yield the largest impact:

```
Performance Improvement ROI:

  3s → 2.5s  ████████████████████  High impact (biggest users gained)
  2.5s → 2s  ██████████████        High impact
  2s → 1.5s  ████████              Moderate impact
  1.5s → 1s  ████                  Lower impact
  1s → 0.8s  ██                    Diminishing returns
  0.8s → 0.6s █                    Marginal for most users
```

**Practical takeaway:** Aim for the "good" thresholds (LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1). Beyond that, optimize only for pages where performance directly impacts business metrics.

## Performance and SEO

Google uses Core Web Vitals as a ranking signal. Pages meeting the "good" thresholds have a competitive advantage in search results — all else being equal, faster pages rank higher.

| Metric | Good Threshold | SEO Impact |
|--------|---------------|------------|
| LCP | ≤ 2.5s | Strong — directly affects first impression |
| INP | ≤ 200ms | Moderate — affects engagement signals |
| CLS | ≤ 0.1 | Moderate — affects bounce rate |
| FCP | ≤ 1.8s | Moderate — affects perceived speed |

## Performance and Accessibility

Performance and accessibility overlap significantly:
- Slow pages increase cognitive load for users with disabilities
- Poor CLS makes pages harder to navigate for screen magnifier users
- Long load times test patience of users with motor impairments

Optimizing for performance often improves accessibility, and vice versa.

## Key Takeaways

1. **Performance is a feature**, not an afterthought — it should be designed and budgeted from day one
2. **Loading speed and UI speed** are different dimensions; both matter
3. **Business impact is measurable** — speed directly affects revenue and retention
4. **Set budgets and enforce them** — without enforcement, performance decays
5. **Aim for "good" thresholds**, not perfection — diminishing returns beyond Core Web Vitals targets

## References

- [Google: Why Performance Matters](https://web.dev/why-performance-matters/)
- [web.dev: Core Web Vitals](https://web.dev/vitals/)
- [Amazon: Amazon's 100ms Secret](https://www.amazon.science/publications/amazons-100ms-secret)
