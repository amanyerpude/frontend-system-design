# A/B Testing

## What Is A/B Testing?

A/B testing (split testing) is a controlled experiment where two or more versions of a UI element, flow, or feature are shown to statistically similar segments of users, and the difference in outcomes is measured against a defined metric. It replaces subjective opinion ("I think the green button is better") with evidence ("version B increased signups by 14% at 95% confidence").

A/B testing is the primary mechanism for **data-driven product decisions** — it lets teams validate changes with real users before committing to a full rollout.

## A/B Testing vs. Multivariate Testing

| Dimension | A/B Test | Multivariate Test (MVT) |
|-----------|----------|------------------------|
| Variables changed | One | Multiple simultaneously |
| Versions | 2 | Many (2ⁿ combinations) |
| Sample size needed | Moderate | Very large |
| Time to results | Faster | Slower |
| Best for | Focused optimization | Understanding element interactions |
| Complexity | Low | High |

## The A/B Testing Process

```
┌─────────────────────────────────────────────────────────────────┐
│  1. OBSERVE    → Metric underperforming (e.g., CTR at 2.1%)    │
│  2. HYPOTHESIZE→ "Shorter CTA will increase CTR by 15%"        │
│  3. BUILD      → Create variation B with the change             │
│  4. SPLIT      → Randomly assign users to A (control) or B     │
│  5. RUN        → Execute until sample size reached              │
│  6. ANALYZE    → Statistical significance test (p < 0.05)      │
│  7. DECIDE     → Ship winner, or iterate further                │
└─────────────────────────────────────────────────────────────────┘
```

## Writing a Hypothesis

A strong hypothesis has three components:

> **"If we [change], then [metric] will [improve/decrease] because [reasoning]."**

| Example |
|---------|
| "If we shorten the signup form from 6 fields to 3, then completion rate will increase by 15% because reduced friction lowers perceived effort." |
| "If we add social proof above the CTA, then conversion will increase because users trust peer validation." |

The reasoning matters — even a wrong hypothesis generates learning, while a guess does not.

## Sample Size: The Calculation

Running a test without calculating the required sample size is the most common A/B testing mistake. You need enough data for the result to be statistically meaningful.

**Three inputs determine sample size:**

1. **Baseline conversion rate** — current performance (e.g., 5%)
2. **Minimum detectable effect (MDE)** — smallest improvement worth detecting (e.g., 20% relative → 5% to 6%)
3. **Significance level** — false positive tolerance (typically 5%, p < 0.05)
4. **Statistical power** — probability of detecting a real effect (typically 80%)

**Formula:**

```
n = (Z_α/2 + Z_β)² × [p₁(1-p₁) + p₂(1-p₂)] / (p₂ - p₁)²

Where:
  Z_α/2 = 1.96    (for 95% confidence)
  Z_β   = 0.84    (for 80% power)
  p₁    = 0.05    (baseline rate)
  p₂    = 0.06    (expected improved rate)

n = (2.80)² × [0.0475 + 0.0564] / (0.01)²
n = 7.84 × 0.1039 / 0.0001
n ≈ 8,146 per variation
```

**Key insight:** Smaller effects require exponentially larger sample sizes. Detecting a 1% improvement from a 5% baseline requires ~20× more traffic than detecting a 10% improvement.

**Practical reference table** (detecting a 10% relative lift, 95% confidence, 80% power):

| Baseline | MDE (abs) | Sample/variation |
|----------|-----------|------------------|
| 1% | 0.1% | ~1,475,000 |
| 5% | 0.5% | ~62,000 |
| 10% | 1% | ~31,000 |
| 25% | 2.5% | ~12,400 |

## Randomization and Assignment

Proper randomization is critical for valid results:

- **Truly random:** Each user has equal probability of seeing either version — not assigned by geography, device, or arrival time.
- **Consistent assignment:** A user sees the same version across sessions (sticky bucketing) to avoid "flickering."
- **Statistically equivalent groups:** Randomization ensures the groups are comparable at the start; no need to "balance" manually.

```typescript
// Simple deterministic bucketing (stable per user)
function assignVariant(userId: string): 'control' | 'variant' {
  const hash = hashString(userId)  // e.g., FNV-1a
  return hash % 100 < 50 ? 'control' : 'variant'
}
```

## Statistical Significance

A result is **statistically significant** when the probability that the observed difference occurred by random chance is below the significance threshold (typically p < 0.05, or 95% confidence).

| p-value | Interpretation |
|---------|----------------|
| p < 0.01 | Strong evidence variant is better |
| p < 0.05 | Standard threshold; acceptable confidence |
| p < 0.10 | Weak evidence; needs more data |
| p ≥ 0.10 | No reliable difference detected |

### Common Statistical Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| **Peeking** | Checking results repeatedly inflates false positives (checking 5× → effective p = 0.15–0.25) | Pre-calculate sample size; run to completion |
| **Stopping early** | Ending when one version "looks" like a winner captures noise, not signal | Run to pre-calculated sample size |
| **Ignoring sample size** | 200 visitors is unreliable regardless of what the numbers show | Always run a power analysis |
| **Testing too many variations** | A/B/C/D/E splits power five ways | Limit to 2–3 variations |
| **Survivorship bias** | Only reporting winning tests distorts the picture | Log all tests including losses |
| **Segment peeking** | Finding a significant segment post-hoc is usually noise | Pre-register segments before the test |

## Frontend Implementation Patterns

### Pattern 1: Feature Flag–Based

The most common pattern: a feature flag controls which version a user sees.

```typescript
// feature-flags.ts
export const flags = {
  newCheckout: {
    control: () => <OldCheckout />,
    variant: () => <NewCheckout />,
  },
}

// In the component tree
function CheckoutPage() {
  const variant = useVariant('newCheckout', userId)
  return variant === 'variant' ? <NewCheckout /> : <OldCheckout />
}
```

### Pattern 2: Server-Side Assignment

The server decides which variant a user gets (based on a stable user ID) and returns it with the initial payload or API response. Avoids client-side flicker (FOUC).

```typescript
// Server returns variant assignment with initial page data
const response = await fetch('/api/experiment/checkout')
const { variant, config } = await response.json()

// Client renders based on server decision
render(<Checkout variant={variant} config={config} />)
```

### Pattern 3: Client-Side Cookie/LocalStorage

```typescript
// assignVariant.ts — runs before app initialization
function getVariant(experimentId: string): string {
  const key = `exp_${experimentId}`
  let variant = localStorage.getItem(key)
  if (!variant) {
    variant = Math.random() < 0.5 ? 'control' : 'variant'
    localStorage.setItem(key, variant)
  }
  return variant
}
```

## Tools

| Tool | Best For | Key Feature |
|------|----------|-------------|
| **Optimizely** | Enterprise experimentation | Feature flags, server-side testing, full-stack |
| **VWO** | Mid-market websites | Visual editor, heatmaps, session recordings |
| **LaunchDarkly** | Feature flags + experiments | Developer-first, server-side flags |
| **Google Optimize (sunset)** | — | GA4 + GTM as free alternative |
| **GrowthBook** | Open-source | Self-hosted, SQL-based analysis |
| **PostHog** | Product analytics + A/B | Open-source, combines analytics and experiments |

## A/B Testing Pitfalls

1. **No hypothesis** — Random tests generate data, not knowledge. Always start with a reasoned prediction.
2. **Ending too early** — Early results are noisy. Commit to the sample size calculation upfront.
3. **Testing trivial changes** — Button color `#FF0000` vs `#FF1100` won't produce measurable results. Test meaningfully different approaches.
4. **Ignoring segments** — An overall "no difference" result may mask a variant that wins on mobile but loses on desktop. Analyze by segment when sample sizes allow.
5. **External factors** — Holiday sales, marketing campaigns, and news events skew results. Run tests for at least one full business cycle (7–14 days for web).
6. **Multiple simultaneous tests on same page** — Tests on the same page element create interaction effects. Run one test per page element at a time.

## What to A/B Test (Frontend-Specific)

| Element | Examples | Impact |
|---------|----------|--------|
| **CTA copy and placement** | "Get started" vs "Start free trial"; above fold vs below | High |
| **Form length** | 6 fields vs 3 fields; progressive disclosure | High |
| **Navigation structure** | Top nav vs sidebar; hamburger menu vs visible links | Medium-High |
| **Onboarding flow** | Guided tour vs empty state; sample data vs blank | High |
| **Pricing page** | Annual-first vs monthly-first; 3 tiers vs 2 | High |
| **Hero section** | Image vs video; headline copy variants | Medium |
| **Social proof** | Testimonials vs logos vs counters | Medium |

## ICE Framework for Prioritization

Rank test ideas to focus effort on high-value experiments:

| Score | Meaning |
|-------|---------|
| **Impact** (1–10) | How much could this improve the metric? |
| **Confidence** (1–10) | How sure are we this will produce a result? |
| **Ease** (1–10) | How easy is it to implement? |

**ICE = Impact × Confidence × Ease.** Test the highest ICE-score ideas first.

## Interview Context

In a frontend system design interview, A/B testing comes up when:
- The interviewer asks how you'd validate a design change ("How would you know if the new checkout is better?")
- Designing experimentation infrastructure (feature flags, variant assignment, event tracking)
- Discussing metrics and how to measure success of a design

**Key talking points:**
- Hypothesis → sample size → randomization → significance → decision
- Sticky assignment to avoid flicker
- Server-side assignment avoids FOUC
- Segment analysis to uncover heterogeneous treatment effects

## References

- [Evan Miller's Sample Size Calculator](https://www.evanmiller.org/ab-testing/sample-size.html)
- [Optimizely: Statistics for A/B Testing](https://www.optimizely.com/optimization-glossary/statistical-significance/)
- [GrowthBook Open-Source A/B Testing](https://www.growthbook.io/)
- [PostHog Experiments](https://posthog.com/docs/experiments)
