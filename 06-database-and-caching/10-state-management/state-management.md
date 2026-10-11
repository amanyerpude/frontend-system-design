# State Management

## I. Overview

State management is the pattern of **storing, updating, and sharing data** across a frontend application. As applications grow in complexity, managing state becomes a central architectural concern — affecting performance, maintainability, and developer experience.

## II. Types of State

| Type | Description | Examples |
|------|-------------|---------|
| **Server state** | Data fetched from APIs | User profiles, product lists |
| **URL state** | State encoded in the URL | Route, query params, filters |
| **Form state** | Input values, validation errors | Signup form, search box |
| **UI state** | Component-level display state | Modal open/closed, sidebar collapsed |
| **Application state** | Global shared state | Auth status, theme, notifications |
| **Local component state** | State internal to a component | Counter value, toggle |

### Key Insight

Most state is **not** global. The majority of state should live as close to where it's used as possible — in component state or URL state, not in a global store.

```
State Locality Pyramid (most to least preferred):

      ┌─────────────────┐
      │  Component State │  ← Use most often
      ├─────────────────┤
      │   URL State      │  ← For shareable state
      ├─────────────────┤
      │  Context/Store   │  ← For truly global state
      ├─────────────────┤
      │  Server Cache    │  ← For API data
      └─────────────────┘
```

## III. State Management Approaches

### 1. Component State (useState)

Best for state that's local to a single component.

```tsx
function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}
```

### 2. Lifted State

When two sibling components need to share state, lift it to their common parent.

```tsx
function Parent() {
  const [value, setValue] = useState('')
  return (
    <>
      <Input value={value} onChange={setValue} />
      <Display value={value} />
    </>
  )
}
```

### 3. Context API

Best for truly global state that many components need (theme, auth, locale).

```tsx
// Context definition
const AuthContext = createContext<AuthState | null>(null)

function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    checkAuth().then(setUser).finally(() => setLoading(false))
  }, [])

  return (
    <AuthContext.Provider value={{ user, loading, login, logout }}>
      {children}
    </AuthContext.Provider>
  )
}

// Consumer
function useAuth() {
  const context = useContext(AuthContext)
  if (!context) throw new Error('useAuth must be used within AuthProvider')
  return context
}

function Profile() {
  const { user } = useAuth()
  return <div>{user?.name}</div>
}
```

**Context performance caveat:** Every consumer re-renders when any context value changes. For frequently changing state, split into multiple contexts.

```tsx
// ❌ BAD — single context with frequently changing value
const AppContext = createContext({ user, theme, notifications, ... })

// ✅ GOOD — separate contexts for independent concerns
const AuthContext = createContext(user)
const ThemeContext = createContext(theme)
const NotificationContext = createContext(notifications)
```

### 4. Redux / Zustand / Jotai / Recoil

Global state management libraries — use when Context isn't sufficient.

| Library | Pattern | Best For |
|---------|---------|----------|
| **Redux Toolkit** | Single store, slices, actions | Large teams, complex state logic |
| **Zustand** | Minimal boilerplate, selector-based | Small-medium apps, simple global state |
| **Jotai** | Atomic (bottom-up) | Fine-grained state, derived state |
| **Recoil** | Atom-based (top-down) | Complex dependency graphs |
| **MobX** | Observables, reactive | Highly reactive applications |

#### Zustand Example

```typescript
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

interface AppState {
  user: User | null
  theme: 'light' | 'dark'
  notifications: Notification[]
  setUser: (user: User | null) => void
  toggleTheme: () => void
  addNotification: (notification: Notification) => void
}

export const useAppStore = create<AppState>()(
  persist(
    (set) => ({
      user: null,
      theme: 'light',
      notifications: [],

      setUser: (user) => set({ user }),
      toggleTheme: () => set((state) => ({
        theme: state.theme === 'light' ? 'dark' : 'light',
      })),
      addNotification: (notification) => set((state) => ({
        notifications: [...state.notifications, notification],
      })),
    }),
    {
      name: 'app-storage',  // localStorage key
      partialize: (state) => ({ theme: state.theme }),  // Only persist theme
    }
  )
)

// Usage — only re-renders when the selected slice changes
function ThemeToggle() {
  const theme = useAppStore((state) => state.theme)
  const toggleTheme = useAppStore((state) => state.toggleTheme)

  return <button onClick={toggleTheme}>{theme}</button>
}
```

#### Jotai Example (Atomic State)

```typescript
import { atom, useAtom } from 'jotai'

// Primitive atoms
const countAtom = atom(0)
const userAtom = atom<User | null>(null)

// Derived atoms (computed)
const doubleCountAtom = atom((get) => get(countAtom) * 2)
const isLoggedInAtom = atom((get) => get(userAtom) !== null)

// Write atom (action)
const incrementAtom = atom(null, (get, set) => {
  set(countAtom, get(countAtom) + 1)
})

function Counter() {
  const [count] = useAtom(countAtom)
  const [, increment] = useAtom(incrementAtom)
  const [double] = useAtom(doubleCountAtom)

  return <button onClick={increment}>{count} (doubled: {double})</button>
}
```

## IV. Server State vs. Client State

| Aspect | Server State | Client State |
|--------|-------------|--------------|
| **Source of truth** | Server | Client |
| **Caching** | React Query, SWR, Apollo | Redux, Zustand, Context |
| **Refetching** | Automatic (stale-while-revalidate) | Manual |
| **Persistence** | Server-side | localStorage, sessionStorage |
| **Examples** | User data, product lists, API responses | Theme, form inputs, UI state |

**Key principle:** Server state and client state should be managed by different tools. Use React Query/SWR for server state; use Zustand/Redux/Context for client state.

```tsx
// ✅ GOOD — server state via React Query, client state via Zustand
function ProductList() {
  // Server state — fetched, cached, auto-refetched
  const { data: products, isLoading } = useQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
  })

  // Client state — UI preference
  const viewMode = useAppStore((state) => state.viewMode)

  return (
    <div className={viewMode}>
      {products?.map(product => <ProductCard key={product.id} product={product} />)}
    </div>
  )
}
```

## V. URL State

URL state is shareable and survives page reloads — ideal for filters, pagination, and sort orders.

```tsx
// Using react-router's useSearchParams
import { useSearchParams } from 'react-router-dom'

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams()

  const page = Number(searchParams.get('page')) || 1
  const sort = searchParams.get('sort') || 'name'
  const category = searchParams.get('category') || ''

  const updateFilter = (key: string, value: string) => {
    const params = new URLSearchParams(searchParams)
    if (value) {
      params.set(key, value)
    } else {
      params.delete(key)
    }
    setSearchParams(params)
  }

  // URL reflects state: /products?page=2&sort=price&category=electronics
  return <ProductGrid products={products} page={page} sort={sort} />
}
```

## VI. State Colocation

**Colocate state with the components that use it.** Don't lift state higher than necessary.

```tsx
// ❌ BAD — lifted too high
function App() {
  const [searchQuery, setSearchQuery] = useState('')  // Only used in SearchBar
  const [modalOpen, setModalOpen] = useState(false)   // Only used in Modal

  return (
    <>
      <SearchBar value={searchQuery} onChange={setSearchQuery} />
      <Modal open={modalOpen} onClose={() => setModalOpen(false)} />
    </>
  )
}

// ✅ GOOD — state colocated with the component that uses it
function App() {
  return (
    <>
      <SearchBar />    {/* Owns its own searchQuery state */}
      <Modal />         {/* Owns its own open state */}
    </>
  )
}
```

## VII. State Management Decision Framework

```
Where should this state live?
│
├── Used by ONE component?
│   └── useState (component state)
│
├── Used by SIBLING components?
│   └── Lift to common parent
│
├── Used by MANY components (but rarely changes)?
│   └── Context API
│
├── Used by MANY components (changes frequently)?
│   └── Zustand / Redux (with selectors)
│
├── Must be shareable via URL?
│   └── URL state (useSearchParams)
│
├── Comes from the server?
│   └── React Query / SWR / Apollo
│
└── Must survive page reload?
    └── localStorage / sessionStorage (+ sync with store)
```

## VIII. Performance Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| **Putting all state in one store** | Every change re-renders all consumers | Use selectors; split stores |
| **Context for frequently changing state** | Context value changes re-render all consumers | Use Zustand/Redux with selectors |
| **Lifting state too high** | Unnecessary re-renders of unrelated components | Colocate state with consumers |
| **Deriving state instead of computing** | Sync bugs, redundant stores | Use `useMemo` or derived atoms |
| **Storing server data in client store** | Duplicate caches, sync complexity | Use React Query/SWR for server state |

## IX. Key Takeaways

1. **Not all state is equal** — server, URL, form, UI, and application state have different needs
2. **Colocate state** — keep it as close to the consumer as possible
3. **Separate server and client state** — React Query for API data, Zustand/Redux for UI state
4. **URL state is shareable** — use it for filters, pagination, sort orders
5. **Context has performance implications** — use selectors or split contexts for frequently changing state
6. **Don't over-engineer** — not every app needs Redux; start with useState and Context

## References

- [React: State — You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
- [Zustand Documentation](https://github.com/pmndrs/zustand)
- [Jotai Documentation](https://jotai.org/)
- [TanStack Query Documentation](https://tanstack.com/query/latest)
