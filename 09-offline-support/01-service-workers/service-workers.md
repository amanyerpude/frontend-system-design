# Service Workers

## I. What Is a Service Worker?

A Service Worker is a **programmable network proxy** that runs in a background thread, separate from the main JavaScript thread. It intercepts network requests and can serve responses from cache, fulfill requests from the network, or combine both using custom strategies. It is the core technology enabling offline support, push notifications, and background sync in Progressive Web Apps.

## II. Core Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Runs in background** | Separate thread; doesn't block UI |
| **Programmable proxy** | Intercepts every network request within scope |
| **Cache API access** | Can read/write to a dedicated cache storage |
| **Event-driven** | Responds to events: install, activate, fetch, sync, push |
| **Origin-scoped** | Only controls requests from the same origin |
| **Path-scoped** | Controls URLs under its registration path |
| **No DOM access** | Cannot access `document`, `window`, or DOM elements |
| **Terminated when idle** | Browser terminates SW when not in use |

## III. Lifecycle

```
Registration → Install → Waiting → Activate → Idle/Fetch
```

### Registration

```typescript
// main.ts — register the service worker
if ('serviceWorker' in navigator) {
  window.addEventListener('load', async () => {
    try {
      const registration = await navigator.serviceWorker.register('/sw.js', {
        scope: '/',  // Controls all URLs under /
      })
      console.log('SW registered:', registration.scope)
    } catch (error) {
      console.error('SW registration failed:', error)
    }
  })
}
```

### Install

The install event fires when the Service Worker is first installed. Pre-cache critical assets here:

```javascript
// sw.js
const CACHE_NAME = 'app-cache-v1'
const PRE_CACHE_URLS = [
  '/',
  '/offline.html',
  '/styles.css',
  '/app.js',
  '/logo.png',
]

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(PRE_CACHE_URLS))
  )
  self.skipWaiting()  // Activate immediately, skip waiting phase
})
```

### Activate

The activate event fires when the Service Worker takes control. Clean up old caches here:

```javascript
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames
          .filter(name => name !== CACHE_NAME)
          .map(name => caches.delete(name))
      )
    }).then(() => self.clients.claim())  // Take control of all clients
  )
})
```

### Fetch Interception

The fetch event fires for every network request within the Service Worker's scope:

```javascript
self.addEventListener('fetch', (event) => {
  // Only handle GET requests
  if (event.request.method !== 'GET') return

  event.respondWith(
    caches.match(event.request)
      .then(cachedResponse => {
        if (cachedResponse) {
          return cachedResponse  // Serve from cache
        }
        return fetch(event.request)  // Fall back to network
      })
  )
})
```

## IV. Caching Strategies

### Cache First

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then(cached => cached || fetch(event.request))
  )
})
```

**Use for:** Static assets (images, fonts, CSS, JS with content hashes)

### Network First

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    fetch(event.request)
      .then(response => {
        const clone = response.clone()
        caches.open(CACHE_NAME).then(c => c.put(event.request, clone))
        return response
      })
      .catch(() => caches.match(event.request))
  )
})
```

**Use for:** API responses, HTML pages that update frequently

### Stale-While-Revalidate

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then(cached => {
      const fetchPromise = fetch(event.request).then(response => {
        caches.open(CACHE_NAME).then(c => c.put(event.request, response.clone()))
        return response
      })
      return cached || fetchPromise
    })
  )
})
```

**Use for:** API responses that can tolerate slightly stale data

## V. Offline Fallback

```javascript
self.addEventListener('fetch', (event) => {
  if (event.request.mode === 'navigate') {
    event.respondWith(
      fetch(event.request).catch(() => caches.match('/offline.html'))
    )
  }
})
```

## VI. Update Flow

```javascript
// main.ts — handle Service Worker updates
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.addEventListener('controllerchange', () => {
    // New Service Worker has taken control
    window.location.reload()
  })

  navigator.serviceWorker.ready.then((registration) => {
    // Check for updates periodically
    setInterval(() => {
      registration.update()
    }, 60 * 60 * 1000)  // Every hour
  })
}
```

## VII. Debugging

| Tool | Purpose |
|------|---------|
| **DevTools → Application → Service Workers** | Registration status, scope, update |
| **DevTools → Application → Cache Storage** | View cached responses |
| **DevTools → Network → SW column** | See which requests are intercepted |
| `chrome://serviceworker-internals` | Detailed SW state |

## VIII. Key Takeaways

1. **Service Worker is a network proxy** — it intercepts requests, not DOM events
2. **Install event** — pre-cache critical assets
3. **Activate event** — clean up old caches
4. **Fetch event** — implement caching strategies
5. **No DOM access** — SW runs in a separate thread without `document`/`window`
6. **Terminated when idle** — don't store state in SW variables

## References

- [MDN: Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [web.dev: Service Workers](https://web.dev/learn/pwa/service-workers/)
- [Jake Archibald: The Offline Cookbook](https://jakearchibald.com/2014/offline-cookbook/)
