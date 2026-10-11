# Unit and Integration Testing

## Unit Testing

### What Is a Unit Test?

A unit test verifies that a single "unit" of code (typically a function) produces the correct output for a given input. The unit should be tested in **isolation** — its dependencies are replaced with mocks, stubs, or fakes so that the test focuses solely on the unit's logic.

### Characteristics of Good Unit Tests

| Principle | Description |
|-----------|-------------|
| **Fast** | Unit tests should run in milliseconds. If they're slow, you're testing too much. |
| **Isolated** | Each test should be independent — no shared state, no reliance on execution order. |
| **Repeatable** | Running the same test multiple times should produce the same result every time. |
| **Self-Checking** | Tests should have assertions that determine pass/fail automatically. |
| **Thorough** | Cover happy paths, edge cases, boundary conditions, and error cases. |

### Unit Testing in JavaScript/TypeScript

The standard toolset for frontend unit testing is **Jest** (or **Vitest**), often paired with **React Testing Library** for React applications.

#### Setup Example

```bash
# Using Jest
npm install --save-dev jest @types/jest

# Using Vitest (faster alternative)
npm install --save-dev vitest @vitest/ui
```

#### Basic Unit Test Structure

```typescript
// math.ts — the unit under test
export function calculateTotal(items: { price: number; quantity: number }[]) {
  return items.reduce((total, item) => total + item.price * item.quantity, 0)
}

// math.test.ts — the test
import { calculateTotal } from './math'

describe('calculateTotal', () => {
  it('should sum up total price * quantity for all items', () => {
    const items = [
      { price: 10, quantity: 2 },
      { price: 5, quantity: 3 },
    ]
    expect(calculateTotal(items)).toBe(35) // 10*2 + 5*3 = 35
  })

  it('should return 0 for an empty array', () => {
    expect(calculateTotal([])).toBe(0)
  })

  it('should handle zero-priced items', () => {
    const items = [{ price: 0, quantity: 5 }]
    expect(calculateTotal(items)).toBe(0)
  })
})
```

#### Mocking Dependencies

Mocks isolate the unit under test by replacing external dependencies:

```typescript
// api.ts — a function we want to mock
export async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`)
  return response.json()
}

// userService.ts — depends on fetchUser
import { fetchUser } from './api'

export async function getUserName(id: string) {
  const user = await fetchUser(id)
  return user.name.toUpperCase()
}

// userService.test.ts
import { getUserName } from './userService'
import * as api from './api'

jest.mock('./api')

describe('getUserName', () => {
  it('should return uppercase user name', async () => {
    // Arrange — mock the dependency
    const mockedFetchUser = api.fetchUser as jest.MockedFunction<typeof api.fetchUser>
    mockedFetchUser.mockResolvedValue({ name: 'alice', id: '1' })

    // Act
    const result = await getUserName('1')

    // Assert
    expect(result).toBe('ALICE')
    expect(mockedFetchUser).toHaveBeenCalledWith('1')
  })
})
```

### Testing Pure Functions

Pure functions (no side effects, same input always produces same output) are the ideal candidates for unit testing:

```typescript
// ✅ Easy to test — pure function
export function isValidEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
}

// ✅ Easy to test — pure function
export function formatCurrency(amount: number, currency = 'USD'): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
  }).format(amount)
}
```

### What NOT to Unit Test

| Category | Reason |
|----------|--------|
| **Framework code** | React, Vue, Angular are already tested by their maintainers |
| **Simple getters/setters** | Trivial implementation, high risk of false value |
| **Third-party libraries** | Already tested by their authors |
| **Configuration files** | Declarative, low risk of bugs |
| **UI layout/styling** | Better covered by visual/E2E tests |

## Integration Testing

### What Is Integration Testing?

Integration tests verify that **multiple units work together correctly**. While a unit test isolates a function and mocks its dependencies, an integration test allows those dependencies to interact — whether they're other modules within your application, a database, an API, or a combination.

The key distinction: **unit tests ask "does this function work?"** while **integration tests ask "do these pieces work together?"**

### Integration Testing Scenarios

#### Testing Module Interactions

```typescript
// auth.service.ts
import { fetchUser } from './user.api'
import { validatePassword } from './password.utils'

export async function login(email: string, password: string) {
  const user = await fetchUser(email)
  if (!user) throw new Error('User not found')
  if (!validatePassword(password, user.hashedPassword)) {
    throw new Error('Invalid password')
  }
  return { token: generateToken(user), user }
}

// auth.service.test.ts — integration test
import { login } from './auth.service'
import { fetchUser } from './user.api'
import { validatePassword } from './password.utils'

// Note: no jest.mock() — we test real interactions
describe('login integration', () => {
  it('should authenticate valid user and return token', async () => {
    // This tests the real interaction between auth.service,
    // user.api module, and password.utils module
    const result = await login('test@example.com', 'correct-password')

    expect(result.token).toBeDefined()
    expect(result.user.email).toBe('test@example.com')
  })
})
```

#### Testing Database Interactions

```typescript
// Using MSW (Mock Service Worker) for integration testing
import { setupServer } from 'msw/node'
import { rest } from 'msw'
import { getUserData } from './user-api'

const server = setupServer(
  rest.get('/api/user/:id', (req, res, ctx) => {
    return res(
      ctx.json({ id: 1, name: 'Alice', email: 'alice@example.com' })
    )
  })
)

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())

describe('getUserData', () => {
  it('should fetch user data from API', async () => {
    const data = await getUserData(1)
    expect(data.name).toBe('Alice')
  })

  it('should handle API errors', async () => {
    server.use(
      rest.get('/api/user/:id', (req, res, ctx) => {
        return res(ctx.status(500))
      })
    )
    await expect(getUserData(1)).rejects.toThrow('Failed to fetch user')
  })
})
```

#### Testing React Component Integration

```typescript
// Testing how a component interacts with a context/provider
import { render, screen, fireEvent } from '@testing-library/react'
import { ThemeProvider } from './theme-context'
import { Button } from './button'

describe('Button with Theme', () => {
  it('should apply primary styling in light theme', () => {
    render(
      <ThemeProvider theme="light">
        <Button variant="primary">Click me</Button>
      </ThemeProvider>
    )

    const button = screen.getByRole('button', { name: /click me/i })
    expect(button).toHaveClass('btn-primary')
  })
})
```

### MSW (Mock Service Worker)

MSW is the modern standard for mocking network requests in integration and component tests. Unlike traditional mocking (which intercepts at the library level), MSW intercepts requests at the **network level** — meaning the same code runs in both tests and production.

```typescript
// test-setup.ts
import { setupServer } from 'msw/node'
import { handlers } from './handlers'

export const server = setupServer(...handlers)

// In your test setup
beforeAll(() => server.listen())
beforeEach(() => server.resetHandlers())
afterAll(() => server.close())
```

This approach:
- Tests code with the same HTTP call patterns used in production
- No need to refactor code specifically for testability
- Works with any HTTP client (fetch, axios, Apollo, etc.)
- Provides a realistic testing environment without a real backend

### When to Use Unit vs. Integration Tests

| Scenario | Unit Test | Integration Test |
|----------|-----------|------------------|
| Testing a pure calculation function | ✅ Yes | ❌ No |
| Testing component renders with props | ✅ Possibly | ⚠️ Maybe |
| Testing component + context/provider interaction | ❌ No | ✅ Yes |
| Testing API call + response transformation | ❌ No | ✅ Yes |
| Testing module-to-module data flow | ❌ No | ✅ Yes |
| Testing database query logic | ❌ No (mock DB) | ✅ Yes (use test DB) |
| Testing error handling across modules | ❌ No | ✅ Yes |

## Testing Strategies: Arrange-Act-Assert

All good tests follow the AAA pattern, regardless of type:

```typescript
it('should apply 10% tax to order subtotal', () => {
  // ─── ARRANGE ─────────────────────────────────────
  const order = createOrder([{ price: 100, quantity: 1 }])

  // ─── ACT ─────────────────────────────────────────
  const total = calculateTotalWithTax(order, 0.10)

  // ─── ASSERT ─────────────────────────────────────
  expect(total).toBe(110)
})
```

### Boundary Value Analysis

Test edge cases and boundary conditions:

```typescript
describe('calculateDiscount', () => {
  it('should return 0 discount for 0 items', () => {
    expect(calculateDiscount(0)).toBe(0)
  })

  it('should return 50 discount for 10 items (boundary)', () => {
    expect(calculateDiscount(10)).toBe(50)
  })

  it('should return 100 discount for 20 items (boundary)', () => {
    expect(calculateDiscount(20)).toBe(100)
  })

  it('should return maximum 100 discount for 50 items', () => {
    expect(calculateDiscount(50)).toBe(100)
  })
})
```

## Test-Driven Development (TDD) Connection

While this section focuses on unit and integration testing techniques, the broader testing strategy connects to TDD (covered in [TDD Overview](./07-tdd-overview/tdd-overview.md)). TDD flips the order:
1. Write a failing test first
2. Write minimal code to pass
3. Refactor with confidence

This approach ensures testability is built-in from the start, making both unit and integration tests more effective and maintainable.

## References

- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Vitest Documentation](https://vitest.dev/guide/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Mock Service Worker](https://mswjs.io/docs/)
- [Testing JavaScript Applications](https://www.manning.com/books/testing-javascript-applications)
