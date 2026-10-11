# React Testing: Jest & React Testing Library

## The Testing Philosophy

React testing follows a core principle: **test from the user's perspective, not the developer's**. This means:

- **Do:** Test what the user sees and does (text, buttons, user interactions)
- **Don't:** Test implementation details (state, props, internal methods, refs)

The tool that enforces this philosophy is **React Testing Library (RTL)** — the standard for testing React components in 2026.

## The Toolset

| Tool | Role | Why It's Used |
|------|------|---------------|
| **Jest** | Test runner + assertion library | Fast, parallel execution, built-in mocking, snapshot testing |
| **React Testing Library (RTL)** | Component rendering + queries | Tests from user perspective, avoids testing internals |
| **jest-dom** | Extended DOM matchers | `toBeInTheDocument()`, `toHaveClass()`, `toBeVisible()` |
| **MSW** | API mocking | Intercepts at network level, no code refactor needed |
| **Vitest** | Alternative test runner | Faster than Jest, native ESM, Vite integration |

## Setup

```bash
# For React projects (create-react-app or Vite with Vitest)
npm install --save-dev jest @testing-library/react @testing-library/jest-dom @testing-library/user-event

# Optional but recommended
npm install --save-dev msw jest-extended
```

### Jest Configuration for React

```typescript
// jest.config.ts
import type { Config } from 'jest'

const config: Config = {
  testEnvironment: 'jsdom',        // DOM environment for component tests
  setupFilesAfterSetup: ['./jest.setup.ts'],
  moduleNameMapper: {
    '\\.(css|scss)$': '<rootDir>/__mocks__/styleMock.js',
    '\\.(jpg|png|svg)$': '<rootDir>/__mocks__/fileMock.js',
  },
  transform: {
    '^.+\\.(ts|tsx)$': ['ts-jest', { tsconfig: 'tsconfig.test.json' }],
  },
}

export default config
```

```typescript
// jest.setup.ts
import '@testing-library/jest-dom'

// Mock matchMedia (not available in jsdom)
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation((query) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
})
```

## Component Testing Basics

### Testing Component Rendering

```typescript
// Greeting.tsx
interface GreetingProps {
  name: string
  isLoggedIn: boolean
}

export function Greeting({ name, isLoggedIn }: GreetingProps) {
  if (!isLoggedIn) return <p>Please log in</p>
  return <h1>Welcome back, {name}!</h1>
}

// Greeting.test.tsx
import { render, screen } from '@testing-library/react'
import { Greeting } from './Greeting'

describe('Greeting', () => {
  it('should display welcome message for logged-in user', () => {
    render(<Greeting name="Alice" isLoggedIn={true} />)
    expect(screen.getByRole('heading', { name: /welcome back, alice/i }))
      .toBeInTheDocument()
  })

  it('should display login prompt for logged-out user', () => {
    render(<Greeting name="Alice" isLoggedIn={false} />)
    expect(screen.getByText(/please log in/i)).toBeInTheDocument()
  })
})
```

### Testing User Interactions

```typescript
// Counter.tsx
import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
      <button onClick={() => setCount((c) => c - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  )
}

// Counter.test.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { Counter } from './Counter'

describe('Counter', () => {
  it('should start at 0', () => {
    render(<Counter />)
    expect(screen.getByText('Count: 0')).toBeInTheDocument()
  })

  it('should increment when + button is clicked', async () => {
    const user = userEvent.setup()
    render(<Counter />)

    await user.click(screen.getByRole('button', { name: /increment/i }))

    expect(screen.getByText('Count: 1')).toBeInTheDocument()
  })

  it('should handle rapid clicks correctly', async () => {
    const user = userEvent.setup()
    render(<Counter />)

    await user.click(screen.getByRole('button', { name: /increment/i }))
    await user.click(screen.getByRole('button', { name: /increment/i }))
    await user.click(screen.getByRole('button', { name: /increment/i }))

    expect(screen.getByText('Count: 3')).toBeInTheDocument()
  })

  it('should reset when reset button is clicked', async () => {
    const user = userEvent.setup()
    render(<Counter />)

    await user.click(screen.getByRole('button', { name: /increment/i }))
    await user.click(screen.getByRole('button', { name: /reset/i }))

    expect(screen.getByText('Count: 0')).toBeInTheDocument()
  })
})
```

### Testing Async Behavior

```typescript
// UserProfile.tsx
import { useEffect, useState } from 'react'

interface User {
  id: number
  name: string
  email: string
}

export function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .catch((err) => setError(err.message))
      .finally(() => setLoading(false))
  }, [userId])

  if (loading) return <p>Loading...</p>
  if (error) return <p role="alert">Error: {error}</p>
  if (!user) return <p>User not found</p>

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  )
}

// UserProfile.test.tsx
import { render, screen, waitFor } from '@testing-library/react'
import { UserProfile } from './UserProfile'

// Mock fetch
global.fetch = jest.fn()

beforeEach(() => {
  (fetch as jest.Mock).mockClear()
})

it('should show loading state initially', () => {
  (fetch as jest.Mock).mockReturnValue(new Promise(() => {})) // Never resolves
  render(<UserProfile userId={1} />)
  expect(screen.getByText('Loading...')).toBeInTheDocument()
})

it('should display user data after fetch succeeds', async () => {
  (fetch as jest.Mock).mockResolvedValue({
    json: () => Promise.resolve({ id: 1, name: 'Alice', email: 'alice@example.com' }),
  })

  render(<UserProfile userId={1} />)

  await waitFor(() => {
    expect(screen.getByRole('heading', { name: 'Alice' })).toBeInTheDocument()
  })
  expect(screen.getByText('alice@example.com')).toBeInTheDocument()
})

it('should show error message when fetch fails', async () => {
  (fetch as jest.Mock).mockRejectedValue(new Error('Network error'))

  render(<UserProfile userId={1} />)

  await waitFor(() => {
    expect(screen.getByRole('alert')).toHaveTextContent('Error: Network error')
  })
})
```

## Query Priority in RTL

RTL provides multiple query methods. Use them in this order of priority:

```typescript
// 1. ✅ getByRole — Best: tests accessibility
screen.getByRole('button', { name: 'Submit' })
screen.getByRole('heading', { name: 'Welcome' })

// 2. ✅ getByLabelText — Best for form inputs
screen.getByLabelText('Email')

// 3. ✅ getByPlaceholderText — Good for inputs with placeholder
screen.getByPlaceholderText('Search...')

// 4. ✅ getByText — Best for non-interactive text
screen.getByText('Welcome back, Alice!')

// 5. ✅ getByDisplayValue — Best for controlled inputs
screen.getByDisplayValue('initial value')

// 6. ⚠️ getByTitle — Use sparingly (title attribute)
screen.getByTitle('Close')

// 7. ❌ getByTestId — Last resort, when no other query works
// Only use when: role/text/label don't work (complex widgets, canvas, SVG)
screen.getByTestId('chart-container')

// ❌ Avoid: querySelector, getElementById, container.querySelector
```

## Mocking

### Mocking Functions

```typescript
import { fireEvent, render, screen } from '@testing-library/react'

it('should call onSubmit with form data', () => {
  const handleSubmit = jest.fn()

  render(<LoginForm onSubmit={handleSubmit} />)

  fireEvent.change(screen.getByLabelText('Email'), {
    target: { value: 'test@example.com' },
  })
  fireEvent.change(screen.getByLabelText('Password'), {
    target: { value: 'password123' },
  })
  fireEvent.click(screen.getByRole('button', { name: /log in/i }))

  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123',
  })
})
```

### Mocking Modules

```typescript
// Mock an entire module
jest.mock('./api', () => ({
  fetchUser: jest.fn(),
  saveUser: jest.fn(),
}))

// Then import and configure the mock in tests
import { fetchUser } from './api'
const mockFetchUser = fetchUser as jest.MockedFunction<typeof fetchUser>
mockFetchUser.mockResolvedValue({ id: 1, name: 'Alice' })
```

### Mocking with MSW (Preferred for API Mocking)

```typescript
// __mocks__/handlers.ts
import { rest } from 'msw'

export const handlers = [
  rest.get('/api/user/:id', (req, res, ctx) => {
    const { id } = req.params
    return res(
      ctx.json({ id: Number(id), name: 'Alice', email: 'alice@example.com' })
    )
  }),
  rest.post('/api/user', (req, res, ctx) => {
    return res(ctx.json({ id: 2, ...req.body }), ctx.status(201))
  }),
]

// setupTests.ts
import { setupServer } from 'msw/node'
import { handlers } from './__mocks__/handlers'

export const server = setupServer(...handlers)

beforeAll(() => server.listen())
beforeEach(() => server.resetHandlers())
afterAll(() => server.close())

// Test uses real fetch code, MSW intercepts at network level
it('should load user via real fetch (intercepted by MSW)', async () => {
  render(<UserProfile userId={1} />)

  await waitFor(() => {
    expect(screen.getByRole('heading', { name: 'Alice' })).toBeInTheDocument()
  })
})
```

## Testing Custom Hooks

```typescript
// useDebounce.ts
import { useEffect, useState } from 'react'

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value)

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(timer)
  }, [value, delay])

  return debouncedValue
}

// useDebounce.test.ts
import { renderHook, act } from '@testing-library/react'
import { useDebounce } from './useDebounce'

it('should debounce value changes', () => {
  jest.useFakeTimers()

  const { result, rerender } = renderHook(
    ({ value }) => useDebounce(value, 500),
    { initialProps: { value: 'initial' } }
  )

  expect(result.current).toBe('initial')

  rerender({ value: 'updated' })
  expect(result.current).toBe('initial') // Not yet — still debounced

  act(() => {
    jest.advanceTimersByTime(500)
  })

  expect(result.current).toBe('updated') // Now debounced value updated

  jest.useRealTimers()
})
```

## Testing Context Providers

```typescript
// ThemeContext.tsx
import { createContext, useContext, useState } from 'react'

const ThemeContext = createContext<{ theme: string; toggle: () => void }>({
  theme: 'light',
  toggle: () => {},
})

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState('light')
  const toggle = () => setTheme((t) => (t === 'light' ? 'dark' : 'light'))

  return (
    <ThemeContext.Provider value={{ theme, toggle }}>
      {children}
    </ThemeContext.Provider>
  )
}

export const useTheme = () => useContext(ThemeContext)

// ThemeToggle.test.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { ThemeProvider, useTheme } from './ThemeContext'

function ThemeDisplay() {
  const { theme, toggle } = useTheme()
  return (
    <div>
      <p>Current theme: {theme}</p>
      <button onClick={toggle}>Toggle theme</button>
    </div>
  )
}

it('should toggle theme when button is clicked', async () => {
  const user = userEvent.setup()
  render(
    <ThemeProvider>
      <ThemeDisplay />
    </ThemeProvider>
  )

  expect(screen.getByText('Current theme: light')).toBeInTheDocument()

  await user.click(screen.getByRole('button', { name: /toggle theme/i }))

  expect(screen.getByText('Current theme: dark')).toBeInTheDocument()
})
```

## Testing Forms

```typescript
// SignupForm.tsx
import { useState } from 'react'

export function SignupForm({ onSubmit }: { onSubmit: (data: FormData) => void }) {
  const [errors, setErrors] = useState<Record<string, string>>({})

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    const form = new FormData(e.currentTarget)
    const email = form.get('email') as string
    const password = form.get('password') as string
    const newErrors: Record<string, string> = {}

    if (!email.includes('@')) newErrors.email = 'Invalid email'
    if (password.length < 8) newErrors.password = 'Password too short'

    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors)
      return
    }

    onSubmit({ email, password })
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input id="email" name="email" type="email" />
      {errors.email && <span role="alert">{errors.email}</span>}

      <label htmlFor="password">Password</label>
      <input id="password" name="password" type="password" />
      {errors.password && <span role="alert">{errors.password}</span>}

      <button type="submit">Sign up</button>
    </form>
  )
}

// SignupForm.test.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { SignupForm } from './SignupForm'

it('should show validation errors for invalid input', async () => {
  const user = userEvent.setup()
  render(<SignupForm onSubmit={jest.fn()} />)

  await user.type(screen.getByLabelText('Email'), 'not-an-email')
  await user.type(screen.getByLabelText('Password'), 'short')
  await user.click(screen.getByRole('button', { name: /sign up/i }))

  expect(screen.getByText('Invalid email')).toBeInTheDocument()
  expect(screen.getByText('Password too short')).toBeInTheDocument()
})

it('should submit form with valid data', async () => {
  const user = userEvent.setup()
  const handleSubmit = jest.fn()
  render(<SignupForm onSubmit={handleSubmit} />)

  await user.type(screen.getByLabelText('Email'), 'alice@example.com')
  await user.type(screen.getByLabelText('Password'), 'securepassword123')
  await user.click(screen.getByRole('button', { name: /sign up/i }))

  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'alice@example.com',
    password: 'securepassword123',
  })
})
```

## Common Testing Patterns

### Snapshot Testing

```typescript
// Use sparingly — snapshots are brittle and can hide regressions
it('should match snapshot', () => {
  const { container } = render(<UserProfile userId={1} />)
  expect(container).toMatchSnapshot()
})
```

### Testing Error Boundaries

```typescript
import { render, screen } from '@testing-library/react'
import { ErrorBoundary } from './ErrorBoundary'

function BuggyComponent() {
  throw new Error('Component crashed')
}

it('should show fallback UI when child component throws', () => {
  const consoleSpy = jest.spyOn(console, 'error').mockImplementation(() => {})

  render(
    <ErrorBoundary>
      <BuggyComponent />
    </ErrorBoundary>
  )

  expect(screen.getByText(/something went wrong/i)).toBeInTheDocument()

  consoleSpy.mockRestore()
})
```

### Testing Accessibility

```typescript
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { axe, toHaveNoViolations } from 'jest-axe'

expect.extend(toHaveNoViolations)

it('should have no accessibility violations', async () => {
  const { container } = render(<LoginForm onSubmit={jest.fn()} />)
  const results = await axe(container)
  expect(results).toHaveNoViolations()
})
```

## Best Practices Summary

1. **Test user behavior** — What the user sees and does, not internal state
2. **Use accessible queries** — `getByRole` > `getByLabelText` > `getByText` > `getByTestId`
3. **Mock at the network boundary** — Use MSW; keep child components real
4. **Use `userEvent` over `fireEvent`** — More realistic user interaction simulation
5. **Test async with `waitFor`** — Never use arbitrary timeouts
6. **Keep tests simple** — One behavior per test, descriptive test names
7. **Test edge cases** — Empty states, error states, loading states, boundary values
8. **Accessibility tests with axe** — Automated a11y checks in every test file

## References

- [React Testing Library Docs](https://testing-library.com/docs/react-testing-library/intro/)
- [Jest Docs](https://jestjs.io/docs/getting-started)
- [Kent C. Dodds: Common RTL Mistakes](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
- [MSW for React](https://mswjs.io/docs/)
- [jest-axe](https://github.com/nickcolley/jest-axe)
