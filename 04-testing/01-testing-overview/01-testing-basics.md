# Testing Basics: Concepts and Terminology

## What Is Testing?

Testing is the process of evaluating a software application to detect differences between expected and actual outcomes. In frontend system design, testing ensures that user interfaces render correctly, user interactions produce expected behavior, and the application remains reliable as complexity grows.

At its core, testing answers three questions:
- **Does it work?** (Functional correctness)
- **Does it work reliably?** (Consistency across environments)
- **Does it work well?** (Performance, accessibility, security)

## Why Testing Matters for Frontend Systems

Modern frontend applications have moved far beyond static HTML pages. Single-page applications (SPAs) built with React, Vue, Angular, and Svelte are full-fledged programs running in the browser. This complexity introduces new failure modes:

- **State management bugs** — components rendering stale or inconsistent data
- **Race conditions** — asynchronous operations resolving out of order
- **Breaking changes** — refactoring one component breaking another through shared state
- **Browser compatibility** — features working in Chrome but failing in Safari
- **Performance regressions** — bundle bloat slowing down user interactions
- **Accessibility gaps** — new UI elements not navigable via keyboard

Without a testing strategy, frontend applications become increasingly fragile as they scale. Each new feature risks breaking existing functionality, and manual testing becomes infeasible.

## Core Testing Concepts

### Test Types

| Type | Scope | Speed | Purpose |
|------|-------|-------|---------|
| **Unit Test** | A single function or component in isolation | Fastest | Verify individual units work correctly |
| **Integration Test** | Multiple units working together | Medium | Verify modules integrate correctly |
| **End-to-End (E2E) Test** | Full application flow across all layers | Slowest | Verify user journeys work end-to-end |
| **Component Test** | A single UI component in a browser-like environment | Medium-Fast | Verify component renders and behaves correctly |

### Test Anatomy

Every test follows the same structure:

1. **Arrange** — Set up the initial state, mock dependencies, prepare inputs
2. **Act** — Execute the code or interaction being tested
3. **Assert** — Verify the output matches expectations
4. **Cleanup** — Reset state (often automated by test runners)

```javascript
// Arrange
const user = { name: 'Alice', age: 25 }

// Act
const result = formatUserGreeting(user)

// Assert
expect(result).toBe('Hello, Alice!')
```

### Key Terminology

- **Assertion** — A boolean check that validates if a condition is true (e.g., `expect(x).toBe(5)`)
- **Mock** — A fake implementation of a dependency used to isolate the unit under test
- **Fixture** — Predefined data or state used as input for tests
- **Test Runner** — The framework that discovers, executes, and reports on tests (Jest, Vitest, Playwright Test)
- **Flaky Test** — A test that sometimes passes and sometimes fails with the same code (the enemy of CI/CD)
- **Test Coverage** — The percentage of code paths exercised by tests. High coverage doesn't guarantee quality, but low coverage guarantees gaps.
- **Test Double** — A generic term for substitutes (mocks, stubs, fakes, spies) used in place of real dependencies

## The Testing Pyramid

The testing pyramid is a mental model that guides how to distribute testing effort across different test types:

```
    ┌─────────────────────────┐
    │       E2E Tests         │  ← Few (5-10%), slow, broad coverage
    │    (User journeys)      │
    └─────────────────────────┘
    ┌─────────────────────────┐
    │  Integration Tests      │  ← Some (15-25%), medium speed
    │ (Module interactions)   │
    └─────────────────────────┘
    ┌─────────────────────────┐
    │    Unit Tests           │  ← Many (60-80%), fast, focused
    │ (Individual functions)  │
    └─────────────────────────┘
```

**Principle:** Write many fast, focused unit tests; a moderate number of integration tests; and a few critical E2E tests covering the most important user journeys. This balance maximizes confidence while minimizing test execution time.

## Quality Attributes Tested

Testing validates more than just functionality. A comprehensive frontend testing strategy covers:

| Attribute | What's Tested | Tools |
|-----------|--------------|-------|
| **Functional** | Does it do what it's supposed to? | Jest, RTL, Cypress, Playwright |
| **Performance** | Load time, interactivity, responsiveness | Lighthouse, WebPageTest, Vitest bench |
| **Accessibility** | Can users with disabilities use it? | axe-core, Lighthouse, Playwright |
| **Security** | Is it vulnerable to XSS, CSRF, etc.? | OWASP ZAP, WSTG, SAST/DAST tools |
| **Usability** | Is the UX smooth and intuitive? | Manual review, user testing |
| **Compatibility** | Does it work across browsers/devices? | Browser testing tools, Sauce Labs |

## Testing in the Development Lifecycle

Testing should be **shifted left** — integrated as early as possible in the development process:

```
┌─────────────────────────────────────────────────────────────┐
│ Plan → Design → Code → Test → Deploy → Monitor             │
│                                                             │
│ Shift-left means tests are written:                         │
│ - BEFORE code (TDD)                                         │
│ - DURING development (unit, component)                      │
│ - AFTER integration (E2E, visual)                           │
│ - IN PRODUCTION (RUM, monitoring)                           │
└─────────────────────────────────────────────────────────────┘
```

## Common Pitfalls

| Pitfall | Why It's Problematic | How to Avoid |
|---------|---------------------|--------------|
| **No testing strategy** | Bugs reach production, manual testing overhead | Start with one type (unit), expand gradually |
| **100% E2E testing** | Extremely slow, flaky, expensive to maintain | Follow the testing pyramid |
| **Testing implementation details** | Tests break on refactors even if behavior is same | Test behavior from user's perspective |
| **Skipping edge cases** | Bugs in error paths, boundary conditions | Use property-based or table-driven tests |
| **Ignoring flaky tests** | Erodes trust in CI/CD, tests get disabled | Fix flakiness immediately, never ignore |
| **Tests coupled to environment** | Tests pass locally but fail in CI | Use mocks for external dependencies |

## Next Steps

This foundation sets up the concepts needed for each testing type. The following sections dive deeper into:
- **Unit & Integration Testing** — Fast, focused tests for individual functions and module interactions
- **E2E & Automation Testing** — Full user journey validation using modern tools
- **Testing Pyramid** — Understanding UT, CT, E2E quality checks and their balance
- **A/B Testing** — Experimentation and data-driven decisions
- **Performance Testing** — Measuring and improving Core Web Vitals
- **TDD Overview** — Writing tests before code for better design
- **Security Testing** — Validating application security through testing
- **React Testing** — Component testing with Jest and React Testing Library

## References

- [Testing Library Philosophy](https://testing-library.com/docs/guiding-pinciples/)
- [MDN: Test Your HTML](https://developer.mozilla.org/en-US/docs/Web/API)
- [Web.dev: Test and monitor](https://web.dev/uses-web-stale-resource-hints/)
