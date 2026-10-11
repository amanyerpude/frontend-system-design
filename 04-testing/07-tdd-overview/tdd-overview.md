# Test-Driven Development (TDD)

## What Is TDD?

Test-Driven Development is a software development methodology where you **write a failing test before writing any production code**. It inverts the traditional approach of write-code-then-test, making tests the specification for what the code should do.

TDD is not just a testing technique — it's a **design technique**. Writing tests first forces you to think about how code will be used (API design) before thinking about how it's implemented, which leads to simpler, more modular, better-designed code.

## The Red-Green-Refactor Cycle

TDD follows a strict three-step cycle:

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│    ┌────────┐                                           │
│    │  RED   │  Write a failing test for behavior        │
│    │        │  that doesn't exist yet.                  │
│    │        │  Run it → ❌ FAIL                         │
│    └───┬────┘  (proves the test actually tests something)│
│        │                                                 │
│        ▼                                                 │
│    ┌────────┐                                           │
│    │ GREEN  │  Write the MINIMUM code to make           │
│    │        │  the test pass. Don't optimize.           │
│    │        │  Run it → ✅ PASS                         │
│    └───┬────┘                                            │
│        │                                                 │
│        ▼                                                 │
│    ┌────────┐                                           │
│    │REFACTOR│  Improve code quality: remove             │
│    │        │  duplication, clarify naming, extract.    │
│    │        │  Run tests → ✅ still PASS                │
│    └────────┘                                            │
│        │                                                 │
│        └──────► Repeat for next behavior                 │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Step 1: RED — Write a Failing Test

Write a test for the next piece of behavior you want to add. Run it — it must fail because the code doesn't exist yet. This proves the test is actually testing something.

```typescript
// Test: calculator-tdd.test.ts
import { Calculator } from './calculator'

it('should add two numbers', () => {
  const calc = new Calculator()
  expect(calc.add(2, 3)).toBe(5)  // Calculator doesn't exist yet → FAIL
})
```

**Why run a failing test?** If the test passes before you write any code, it's not testing anything. The red state confirms the test is valid.

### Step 2: GREEN — Write Minimal Code to Pass

Write just enough code to make the test pass. Resist the urge to over-engineer or anticipate future requirements.

```typescript
// calculator.ts — minimal implementation
export class Calculator {
  add(a: number, b: number): number {
    return a + b
  }
}

// Test now passes → ✅ GREEN
```

### Step 3: REFACTOR — Improve the Design

Now that the test passes, clean up the code. Remove duplication, clarify names, extract methods. Run the test after every change to ensure it still passes.

```typescript
// calculator.ts — refactored (extracted validation)
export class Calculator {
  private validateNumbers(a: number, b: number) {
    if (typeof a !== 'number' || typeof b !== 'number') {
      throw new Error('Inputs must be numbers')
    }
  }

  add(a: number, b: number): number {
    this.validateNumbers(a, b)
    return a + b
  }
}
```

**Green is not done.** Technical debt accumulates if you skip the refactor step. Refactor before moving to the next test.

## Full TDD Example

```
Feature: Calculate order total with discount

─── CYCLE 1 ────────────────────────────────────────────
RED:    test('calculates total for single item', () => {
          const order = new Order([{ price: 100, qty: 1 }])
          expect(order.total()).toBe(100)    // ❌ Order doesn't exist
        })

GREEN:  class Order {
          constructor(items) { this.items = items }
          total() { return 100 }             // Hardcoded minimal pass
        }

REFACTOR: (nothing to refactor yet — move on)

─── CYCLE 2 ────────────────────────────────────────────
RED:    test('calculates total for multiple items', () => {
          const order = new Order([
            { price: 100, qty: 1 },
            { price: 50, qty: 2 },
          ])
          expect(order.total()).toBe(200)    // ❌ Returns hardcoded 100
        })

GREEN:  total() { return this.items.reduce((sum, i) => sum + i.price * i.qty, 0) }

REFACTOR: extract subtotal logic, add validation

─── CYCLE 3 ────────────────────────────────────────────
RED:    test('applies 10% discount', () => {
          const order = new Order([{ price: 100, qty: 1 }])
          order.applyDiscount(0.1)
          expect(order.total()).toBe(90)     // ❌ applyDiscount doesn't exist
        })

GREEN:  applyDiscount(rate) { this.discount = rate }
        total() { return subtotal * (1 - (this.discount || 0)) }

REFACTOR: extract tax calculation, normalize discount handling

... continue for each behavior
```

## Why TDD Works

### 1. Better Design

Writing tests first forces you to think about **how the code will be consumed** — its API — before thinking about its implementation. This naturally leads to:

- Loose coupling (the code is testable in isolation)
- Clear interfaces (the test describes how to use the function)
- Small, focused functions (easier to test)

### 2. Confidence

Without tests: "Does this change break something?" → Manual testing, uncertainty, fear.

With TDD: "Does this change break something?" → Run tests → know in seconds.

### 3. Executable Documentation

Tests document what the code should do. Unlike comments, tests can't go stale — if they go stale, they fail. New developers read tests to understand behavior.

### 4. Less Debugging

```
WITHOUT TDD:
Write code → Test manually → Find bugs → Debug → Fix
Time: 2h coding + 3h debugging = 5h total

WITH TDD:
Test → Code → Pass → Repeat
Time: 3h (includes tests) + 0.5h debugging = 3.5h total
```

Bugs are caught immediately at the moment of creation, not hours or days later.

## What to TDD

### ✅ Good candidates for TDD

| Type | Examples |
|------|----------|
| **Business logic** | Calculations, validation rules, state transitions |
| **Domain behavior** | Order processing, user authentication flow, pricing rules |
| **Data transformations** | Normalization, formatting, filtering, sorting |
| **Custom hooks** | React hooks with complex state logic |
| **API handlers** | Request processing, response shaping |

### ❌ Poor candidates for TDD

| Type | Reason |
|------|--------|
| **Framework code** | React, Vue, etc. are already tested by their maintainers |
| **Simple getters/setters** | Trivial, high risk of false value |
| **UI layout/styling** | Better covered by visual regression or CT |
| **Throwaway prototypes** | TDD overhead not worth it for exploratory code |
| **Integration glue** | Network/DB code — use integration tests instead |

## TDD vs. Test-After vs. Test-Anytime

| Approach | When Tests Are Written | Design Benefit | Discipline Required |
|----------|----------------------|----------------|-------------------|
| **TDD** | Before production code | High — tests drive design | High |
| **Test-after** | After production code | Low — implementation drives tests | Medium |
| **Test-anytime** | Whenever convenient | Variable | Low |

The key benefit of TDD is not the tests themselves — it's the **design feedback** that comes from writing tests first. Test-after produces the same tests but misses the design benefit.

## Common TDD Challenges

### "It's slower!"

Initially, yes — expect a 10–20% velocity dip in the first weeks. But over time, TDD often **increases** velocity because debugging time drops dramatically. Track both metrics during adoption to see the net effect.

### "I don't know what test to write."

Start with the simplest behavior. Ask: "What's the most basic thing this should do?" Write that test. Then ask: "What else should it do?" Write that test next.

### "Testing this complex thing is hard."

If it's hard to test, the design likely needs work. TDD pushes you toward better design — smaller functions, clear interfaces, loose coupling.

### "We have too much legacy code."

Apply TDD to new code. When touching existing code (bug fix or feature), add tests first (characterization tests). Gradual improvement over time.

## TDD in a Team

### Adoption Phases

```
Phase 1: LEARN (Week 1-2)
  - TDD katas / workshops
  - Practice on toy problems
  - Pair with experienced TDD practitioners

Phase 2: NEW CODE (Week 3-4)
  - Apply TDD to new features
  - Start with simple user stories
  - Pair programming recommended

Phase 3: EXPAND (Month 2+)
  - More complex features
  - Add tests when modifying existing code
  - Develop team testing patterns
```

### Measuring TDD Impact

| Metric | Before TDD | After TDD | Typical Change |
|--------|-----------|-----------|----------------|
| Bugs in production | 8/sprint | 3/sprint | **-60%** |
| Rework time | 25% of sprint | 10% of sprint | **-60%** |
| Test coverage | ~40% | ~80% | **+100%** |
| Feature velocity (short-term) | Baseline | -10–20% | Slight dip during learning |
| Feature velocity (long-term) | Baseline | +10–30% | Net gain as debugging drops |

## TDD Best Practices

1. **One assertion per test** — Easier to identify failures and understand intent
2. **Small steps** — Write tests requiring 5–10 lines of code, not 100
3. **Don't skip refactor** — Green is not done; technical debt accumulates
4. **Descriptive test names** — `should calculate total with tax`, not `test1`
5. **Test behavior, not implementation** — Tests should survive refactoring
6. **Commit at green** — Each green state is a known-good checkpoint

## TDD vs. Other Testing Methodologies

| Methodology | Focus | When |
|-------------|-------|------|
| **TDD** | Write test → code, drives design | During development |
| **BDD (Behavior-Driven Development)** | User-facing behavior in natural language | During requirement discussion |
| **ATDD (Acceptance Test-Driven Development)** | Acceptance criteria as tests | During sprint planning |
| **Regression Testing** | Prevent previously fixed bugs from recurring | After fixes |

BDD and ATDD complement TDD — they operate at a higher level (user stories, acceptance criteria) while TDD operates at the code level.

## References

- [Test-Driven Development by Example — Kent Beck](https://www.oreilly.com/library/view/test-driven-development/0321146530/)
- [The Pragmatic Programmer — Hunt & Thomas](https://pragprog.com/titles/tpp20/)
- [TDD Reference — Jason C. McDonald](https://testdrivendevelopment.io/)
- [Growing Object-Oriented Software, Guided by Tests — Freeman & Pryce](https://www.growing-object-oriented-software.com/)
