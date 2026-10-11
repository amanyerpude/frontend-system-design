# API Caching

## I. Overview

API caching reduces network requests by storing API responses and reusing them when the same request is made again. In frontend applications, API caching operates at multiple layers: the HTTP cache, the Service Worker cache, and the application-level cache (React Query, SWR, Apollo, etc.).

## II. Caching Layers for APIs

```
┌──────────────────────────────────────────────────────┐
│                     API Request                       │
│                         │                             │
│  Layer 1: Application Cache (React Query, SWR, Apollo)│
│      │ Cache hit → return data                        │
│      │ Cache miss → continue to Layer 2               │
│  Layer 2: HTTP Cache (Cache-Control, ETag)            │
│      │ Cache hit → return from browser cache          │
│      │ Cache miss → continue to Layer 3               │
│  Layer 3: Service Worker Cache                        │
│      │ Cache hit → return from SW cache               │
│      │ Cache miss → continue to Layer 4               │
│  Layer 4: CDN Cache                                   │
│      │ Cache hit → return from CDN edge               │
│      │ Cache miss → continue to Layer 5               │
│  Layer 5: Origin Server                               │
│      │ Process request → return response              │
│      │ Store in cache layers above                    │
└──────────────────────────────────────────────────────┘
```

## III. HTTP Cache Headers for APIs

```nginx
# Short cache for frequently changing data
Cache-Control: public, max-age=60, s-maxage=300, stale-while-revalidate=3600

# No cache for sensitive data
Cache-Control: no-store

# Private cache (browser only, not CDN)
Cache-Control: private, max-age=300

# Cache with ETag for precise revalidation
Cache-Control: public, max-age=3600
ETag: "user-profile-v1"
```

### When to Use Which

| Data Type | Cache-Control | Reason |
|-----------|--------------|--------|
| **Static reference data** (countries, currencies) | `public, max-age=86400` | Changes rarely |
| **User profile** | `private, max-age=300` | Personal data; don't cache at CDN |
| **Product catalog** | `public, max-age=60, s-maxage=300` | Changes frequently but not in real-time |
| **Real-time data** (stock price, chat) | `no-store` | Must always be fresh |
| **Authentication** | `no-store` | Never cache credentials |

## IV. Application-Level Caching

### React Query (TanStack Query)

React Query provides automatic caching, background refetching, and stale-while-revalidate:

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'

function UserProfile({ userId }: { userId: number }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(r => r.json()),
    staleTime: 5 * 60 * 1000,      // Data is fresh for 5 minutes
    gcTime: 30 * 60 * 1000,        // Cache garbage collected after 30 min
    refetchOnWindowFocus: true,     // Refetch when tab regains focus
    refetchOnReconnect: true,       // Refetch when network reconnects
  })

  if (isLoading) return <Spinner />
  if (error) return <Error message={error.message} />

  return <div>{data.name}</div>
}

// Mutation with cache invalidation
function UpdateProfile() {
  const queryClient = useQueryClient()

  const mutation = useMutation({
    mutationFn: (updates: Partial<User>) =>
      fetch('/api/users/me', {
        method: 'PATCH',
        body: JSON.stringify(updates),
      }).then(r => r.json()),
    onSuccess: (data) => {
      // Invalidate all user-related queries
      queryClient.invalidateQueries({ queryKey: ['user'] })
    },
  })

  return (
    <button onClick={() => mutation.mutate({ name: 'Alice' })}>
      Update Profile
    </button>
  )
}
```

### SWR (Vercel)

```typescript
import useSWR from 'swr'

function UserProfile({ userId }: { userId: number }) {
  const { data, error, isLoading, mutate } = useSWR(
    `/api/users/${userId}`,
    fetcher,
    {
      revalidateOnFocus: true,
      revalidateOnReconnect: true,
      refreshInterval: 0,         // No auto-refresh
      dedupingInterval: 2000,     // Deduplicate requests within 2s
      focusThrottleInterval: 5000, // Throttle focus revalidation
    }
  )
  // ...
}
```

### Apollo Client (GraphQL)

```typescript
import { useQuery, useMutation } from '@apollo/client'

const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
    }
  }
`

function UserProfile({ userId }: { userId: string }) {
  const { data, loading } = useQuery(GET_USER, {
    variables: { id: userId },
    fetchPolicy: 'cache-first',    // Use cache if available
    nextFetchPolicy: 'cache-first',
    notifyOnNetworkStatusChange: true,
  })

  // Apollo caches by query + variables
  // Same query with same variables → served from cache
}
```

## V. Cache Key Strategy

The cache key determines when cached data is reused. A good key captures all inputs to the request:

```typescript
// React Query — cache key includes all variables
useQuery({
  queryKey: ['users', { page, limit, search }],  // Key changes when inputs change
  queryFn: () => fetchUsers({ page, limit, search }),
})

// ❌ BAD — key doesn't change when inputs change
useQuery({
  queryKey: ['users'],  // Same key regardless of page/search
  queryFn: () => fetchUsers({ page, limit, search }),
})
```

### Cache Key Patterns

| Pattern | Key | Use Case |
|---------|-----|----------|
| Single resource | `['user', userId]` | User profile, product detail |
| List with filters | `['users', { page, sort, filter }]` | Paginated list |
| Related resource | `['user-posts', userId]` | Posts by a user |
| Compound | `['user', userId, 'settings']` | User settings |

## VI. Cache Invalidation Strategies

| Strategy | When | Implementation |
|----------|------|----------------|
| **Time-based** | Data is fresh for N minutes | `staleTime: 5 * 60 * 1000` |
| **Mutation-based** | After create/update/delete | `invalidateQueries` |
| **Focus-based** | Tab regains focus | `refetchOnWindowFocus: true` |
| **Network-based** | Network reconnects | `refetchOnReconnect: true` |
| **Manual** | User triggers refresh | `mutate()` |

### Invalidation Patterns

```typescript
// Pattern 1: Invalidate specific query
queryClient.invalidateQueries({ queryKey: ['user', userId] })

// Pattern 2: Invalidate all queries with prefix
queryClient.invalidateQueries({ queryKey: ['user'] })

// Pattern 3: Optimistic update (update cache before server responds)
queryClient.setQueryData(['user', userId], (old) => ({
  ...old,
  name: newName,
}))
// Then invalidate in background
queryClient.invalidateQueries({ queryKey: ['user', userId] })

// Pattern 4: Prefetch (warm cache before user navigates)
queryClient.prefetchQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
})
```

## VII. Request Deduplication

Multiple components requesting the same data should share one network request:

```typescript
// React Query: automatic deduplication
// Two components using the same queryKey share one fetch

// Manual deduplication for custom fetchers
const pendingRequests = new Map<string, Promise<any>>()

function deduplicatedFetch(url: string) {
  if (pendingRequests.has(url)) {
    return pendingRequests.get(url)  // Return existing promise
  }

  const promise = fetch(url).then(r => r.json())
  pendingRequests.set(url, promise)

  promise.finally(() => pendingRequests.delete(url))
  return promise
}
```

## VIII. Pagination Caching

```typescript
// Cache each page separately
function useInfiniteUsers() {
  return useInfiniteQuery({
    queryKey: ['users', 'infinite'],
    queryFn: ({ pageParam }) => fetchUsers(pageParam),
    getNextPageParam: (lastPage) => lastPage.nextCursor,
    initialPageParam: undefined,
  })
}

// Each page is cached independently
// Going back to page 1 → served from cache
// Fetching page 3 → new network request, cached on success
```

## IX. Performance Impact

| Scenario | Without Caching | With Caching | Improvement |
|----------|----------------|--------------|-------------|
| Same user visits profile | 1 API call | 0 API calls | 100% faster |
| User navigates back | 1 API call | 0 API calls | 100% faster |
| Two components need same data | 2 API calls | 1 API call | 50% fewer requests |
| Tab refocus | 1 API call | 0 (if fresh) | 100% faster |
| Stale data (5 min old) | 1 API call | 0 (SWR) | Instant, background update |

## X. Key Takeaways

1. **Layer your caching** — HTTP cache + SW cache + application cache
2. **Use React Query or SWR** — they handle deduplication, background refetch, and invalidation automatically
3. **Cache keys must capture all inputs** — key changes when query parameters change
4. **Invalidate on mutation** — after create/update/delete, invalidate related queries
5. **Use stale-while-revalidate** — serve cached data immediately, update in background
6. **Never cache sensitive data** — auth endpoints should use `no-store`

## References

- [TanStack Query Documentation](https://tanstack.com/query/latest)
- [SWR Documentation](https://swr.vercel.app/)
- [Apollo Client Caching](https://www.apollographql.com/docs/react/caching/)
- [web.dev: HTTP caching](https://web.dev/learn/pwa/)
