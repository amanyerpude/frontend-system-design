# Understanding Quality Checks: UT, CT, E2E

## The Three Layers of Quality Checks

A frontend quality strategy is built on three distinct testing layers. Each layer tests a different scope, at a different speed, with a different purpose. Understanding where each layer fits — and what it does *not* cover — is a core frontend system design skill.

```
         ┌───────────────┐
         │      E2E      │  Few, slow, broad    ← Real browser + full stack
         ├───────────────┤
         │ Component Test │  Some, medium        ← Real browser-like DOM
         ├───────────────┤
         │   Unit Test    │  Many, fast, narrow  ← Pure functions, no DOM
         └───────────────┘
```

## UT — Unit Testing

**Scope:** A single function, hook, or utility in complete isolation. No DOM, no network, no framework runtime.

**Environment:** Node.js (or any JS runtime). No browser needed.

**Speed:** Milliseconds per test. A suite of thousands runs in seconds.

**Purpose:** Verify that pure logic — calculations, transformations, branching, state reducers — produces correct output for a given input.

```typescript
// Pure function — ideal unit test
export function paginate<T>(items: T[], page: number, size: number) {
  const start = (page - 1) * size
  return items.slice(start, start + size)
}

// UT: no DOM, no React, just logic
it('should return the correct slice for page 2', () => {
  const items = [1, 2, 3, 4, 5, 6, 7]
  expect(paginate(items, 2, 3)).toEqual([4, 5, 6])
})
```

### What belongs in UT:
- Utility functions (date formatting, validation, math)
- Custom hooks logic (state reducers, computed values)
- Data transformation and normalization
- Business rules and conditionals

### What does NOT belong in UT:
- Anything requiring a DOM or document
- Network calls or API interactions
- Full component rendering

## CT — Component Testing

**Scope:** A single component rendered in a browser-like DOM. The component is the "unit" — but its children, providers, and API calls may be real or mocked depending on the boundary.

**Environment:** Real browser engine (via jsdom or, ideally, a real browser through Playwright/Vitest Browser Mode).

**Speed:** Tens to hundreds of milliseconds per test.

**Purpose:** Verify that a component renders correctly, responds to user interaction, and integrates correctly with its immediate dependencies (contexts, child components, hooks).

```typescript
// CT: render the real component in a real DOM
import { render, screen, fireEvent } from '@testing-library/react'

it('should increment counter when + button is clicked', () => {
  render(<Counter initial={0} />)

  expect(screen.getByText('Count: 0')).toBeInTheDocument()

  fireEvent.click(screen.getByRole('button', { name: '+' }))

  expect(screen.getByText('Count: 1')).toBeInTheDocument()
})
```

### What belongs in CT:
- Component renders expected UI for given props
- User interactions trigger correct state changes
- Component integrates with context/providers correctly
- Conditional rendering based on state or props
- Async data loading states (loading → data → error)

### What does NOT belong in CT:
- Cross-page navigation flows (that's E2E)
- Internal reducer logic in isolation (that's UT)
- Backend/API correctness

### The CT boundary decision

A common question: at what point do you mock during component testing?

```
┌───────────────────────────────────────────────────────┐
│  Component under test                                 │
│  ┌─────────────┐    ┌──────────────┐    ┌──────────┐ │
│  │  Child UI   │    │  API layer   │    │ External │ │
│  │  components │    │  (fetch/api) │    │  service │ │
│  └─────────────┘    └──────────────┘    └──────────┘ │
│       REAL              MOCK               MOCK       │
└───────────────────────────────────────────────────────┘
```

**Rule of thumb:**
- **Child components:** Keep them REAL. They're part of your code and should work together.
- **Network boundaries:** MOCK with MSW. The API is not under test.
- **Third-party services:** MOCK. Payment, analytics, auth providers.

## E2E — End-to-End Testing

**Scope:** The entire application — real browser, real frontend build, real API layer (test environment), real database (test instance).

**Environment:** A real browser driven by Playwright, Cypress, or Selenium, running against a deployed build of the application.

**Speed:** Seconds to minutes per test.

**Purpose:** Verify that complete user journeys work — that the deployment, the routing, the API contracts, the data persistence, and the UI all behave correctly together.

```typescript
// E2E: full stack, real browser, real user flow
test('user can register and see their profile', async ({ page }) => {
  await page.goto('/register')
  await page.fill('[data-testid="email"]', 'new@example.com')
  await page.fill('[data-testid="password"]', 'securePassword123')
  await page.click('[data-testid="submit"]')

  await page.waitForURL('/dashboard')

  await page.goto('/profile')
  await expect(page.locator('[data-testid="email"]'))
    .toHaveText('new@example.com')
})
```

### What belongs in E2E:
- Critical user journeys (registration, checkout, login)
- Cross-system integration (frontend ↔ API ↔ database)
- Deployment verification (smoke tests post-release)
- Accessibility and visual regression of key pages

### What does NOT belong in E2E:
- Edge cases and error paths (test with UT/CT)
- Every combination of user inputs
- Third-party service behavior

## Side-by-Side Comparison

| Dimension | UT | CT | E2E |
|-----------|----|----|-----|
| **Scope** | Single function | Single component | Full user journey |
| **Environment** | Node.js | Browser-like DOM (jsdom) | Real browser |
| **Dependencies** | All mocked | API mocked, children real | All real |
| **Execution speed** | < 10ms | 10–200ms | 1–60s |
| **Feedback** | Instant | Fast | Slow |
| **Failure meaning** | Logic bug | Component bug | Integration/deployment bug |
| **Maintenance cost** | Low | Medium | High |
| **Flakiness risk** | None | Low | Medium–High |
| **Quantity** | Hundreds–thousands | Dozens–hundreds | Dozens |

## The Modern Testing Pyramid (and Its Variations)

The classic pyramid (many UT, some CT, few E2E) remains the default recommendation. But modern frontend testing acknowledges variations:

```
   Classic Pyramid          Honeycomb             Trophy
                                                      ╱╲
      ╱╲                     ╱╲                   ╱──────╲
     ╱E2E╲                  ╱    ╲                │  E2E  │
    ╱──────╲               │  CT  │               ├────────┤
   │   CT   │              │(many)│               │   CT   │  ← Focus here
   ├────────┤              │      │               │ (many) │
   │   UT   │              ╲      ╱               ├────────┤
   │ (many) │               ╲    ╱                │   UT   │
   └────────┘                ╲╱                   └────────┘
```

- **Classic Pyramid:** Balanced across all layers. Safe default.
- **Honeycomb (Netflix):** Minimal unit tests; focus on testing in production via integration and observability.
- **Trophy (Kent C. Dodds):** Integration (component) tests are the centerpiece; fewer unit tests, fewer E2E tests.

**For frontend system design interviews:** The classic pyramid is the expected answer. Mentioning the Trophy or Honeycomb as alternatives — with reasoning — demonstrates depth.

## Quantitative Guidelines

These ratios are not rules, but useful starting points:

| Metric | Guideline |
|--------|-----------|
| **Test distribution** | ~70% UT, ~20% CT, ~10% E2E |
| **Suite execution time** | UT: < 30s total; CT: < 2min; E2E: < 10min |
| **Coverage target** | 80–90% for UT on business logic; not a universal % goal |
| **Flaky rate** | < 1% — any flaky test must be fixed immediately |

## Real-World Failure Modes

Understanding what breaks at each layer helps justify the pyramid:

| Layer | Common Failure | Caught By |
|-------|---------------|-----------|
| Calculation bug | `discount * 0.5` instead of `discount * 0.15` | UT |
| Component doesn't update state | Button click doesn't re-render | CT |
| Context provider missing | `useContext` throws in production | CT |
| API contract changed | Backend renames a field, frontend breaks | E2E |
| Build/deploy broken | Environment variable missing in production | E2E (smoke) |
| Route navigation broken | React Router config has a typo | E2E |

## Key Takeaways

1. **UT, CT, and E2E are not interchangeable** — each catches different failure modes at different scopes.
2. **CT is the sweet spot** for most frontend bugs — it tests real component behavior without the cost of E2E.
3. **E2E is a safety net, not a test suite** — keep it focused on critical user journeys.
4. **Mocks have a boundary** — mock at the network edge (API), keep internal components real.
5. **Speed determines feedback quality** — a fast suite gets run often; a slow suite gets skipped.

## References

- [Kent C. Dodds: Write tests. Not too many. Mostly integration.](https://kentcdodds.com/blog/write-tests)
- [Testing Library Guiding Principles](https://testing-library.com/docs/guiding-principles)
- [Playwright: Component Testing](https://playwright.dev/docs/test-component-testing)
- [Vitest Browser Mode](https://vitest.dev/guide/browser/)
