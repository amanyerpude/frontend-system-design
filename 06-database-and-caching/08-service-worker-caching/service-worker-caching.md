# Service Worker Caching

## I. Overview

A Service Worker is a **programmable network proxy** that runs in a background thread, separate from the main JavaScript thread. It intercepts network requests and can serve responses from cache, fulfill requests from the network, or combine both. It is the foundation of offline-capable Progressive Web Apps (PWAs).

## II. Core Concepts

| Concept | Description |
|---------|-------------|
| **Service Worker** | A JavaScript file that runs in a background thread; intercepts network requests |
| **Registration** | The process of telling the browser about a Service Worker |
| **Scope** | The URL path prefix the Service Worker controls |
| **Cache API** | Storage mechanism used by Service Workers to cache responses |
| **Fetch Event** | Triggered when the page makes a network request; Service Worker intercepts |
| **Install Event** | Triggered when the Service Worker is first installed |
| **Activate Event** | Triggered when the Service Worker takes control |

## III. Service Worker Lifecycle

```
1. Registration
   navigator.serviceWorker.register('/sw.js')
        │
        ▼
2. Install
   - Download sw.js
   - Pre-cache critical assets
   - Fires: install event
        │
        ▼
3. Waiting (Optional)
   - If an old Service Worker is active, new one waits
   - Activates when old one is gone (or skipWaiting())
        │
        ▼
4. Activate
   - Clean up old caches
   - Take control of pages
   - Fires: activate event
        │
        ▼
5. Idle / Fetch Interception
   - Intercepts network requests
   - Serves from cache or network
   - Fires: fetch event
```

### Registration

```typescript
// main.js — Register Service Worker
if ('serviceWorker' in navigator) {
  window.addEventListener('load', async () => {
    try {
      const registration = await navigator.serviceWorker.register('/sw.js', {
        scope: '/',
      })
      console.log('Service Worker registered:', registration.scope)
    } catch (error) {
      console.error('Service Worker registration failed:', error)
    }
  })
}
```

## IV. Caching Strategies

### Strategy 1: Cache First (Cache-First)

Serve from cache if available; fall back to network.

```javascript
// sw.js — Cache First strategy
const CACHE_NAME = 'cache-v1'
const ASSETS_TO_CACHE = [
  '/',
  '/styles.css',
  '/app.js',
  '/offline.html',
]

// Install: Pre-cache critical assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(ASSETS_TO_CACHE))
  )
  self.skipWaiting()  // Activate immediately
})

// Fetch: Serve from cache, fall back to network
self.addEventListener('fetch', (event) => {
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

**Best for:** Static assets that rarely change (images, fonts, CSS)

### Strategy 2: Network First

Try network first; fall back to cache if offline.

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    fetch(event.request)
      .then(response => {
        // Clone response (can only be consumed once)
        const responseClone = response.clone()
        caches.open(CACHE_NAME)
          .then(cache => cache.put(event.request, responseClone))
        return response
      })
      .catch(() => {
        // Network failed — serve from cache
        return caches.match(event.request)
      })
  )
})
```

**Best for:** API responses, HTML pages that update frequently

### Strategy 3: Stale-While-Revalidate

Serve cached version immediately, update cache in background.

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then(cachedResponse => {
      // Fetch fresh version in background
      const fetchPromise = fetch(event.request)
        .then(response => {
          const responseClone = response.clone()
          caches.open(CACHE_NAME)
            .then(cache => cache.put(event.request, responseClone))
          return response
        })
        .catch(() => {})

      // Serve cached version immediately (if available)
      return cachedResponse || fetchPromise
    })
  )
})
```

**Best for:** API responses that can tolerate slightly stale data

### Strategy 4: Network Only

Always fetch from network; never cache.

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(fetch(event.request))
})
```

**Best for:** Analytics, real-time data

### Strategy 5: Cache Only

Only serve from cache; never fetch.

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
  )
})
```

**Best for:** Pre-cached offline-only resources

## V. Cache Invalidation

### Version-Based Invalidation

```javascript
// Bump version on every deploy
const CACHE_NAME = 'cache-v2'  // Changed from v1

self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames
          .filter(name => name !== CACHE_NAME)  // Delete old caches
          .map(name => caches.delete(name))
      )
    })
  )
})
```

### Stale-While-Revalidate for Long-Term Invalidation

```javascript
// Serve stale, update in background — no manual invalidation needed
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then(cached => {
      const fetchPromise = fetch(event.request).then(response => {
        caches.open(CACHE_NAME).then(cache => {
          cache.put(event.request, response.clone())
        })
        return response
      })
      return cached || fetchPromise
    })
  )
})
```

## VI. Offline Fallback

```javascript
// Install: cache offline fallback page
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => {
      return cache.addAll([
        '/offline.html',  // Fallback page
        '/styles.css',
      ])
    })
  )
})

// Fetch: serve offline page when network fails and no cache
self.addEventListener('fetch', (event) => {
  if (event.request.mode === 'navigate') {
    event.respondWith(
      fetch(event.request)
        .catch(() => caches.match('/offline.html'))
    )
  }
})
```

## VII. Background Sync

```javascript
// Queue failed requests for retry when online
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-data') {
    event.waitUntil(syncPendingRequests())
  }
})

async function syncPendingRequests() {
  const pending = await getPendingRequests()  // From IndexedDB
  for (const request of pending) {
    try {
      await fetch(request.url, {
        method: request.method,
        body: request.body,
      })
      await removePendingRequest(request.id)
    } catch (error) {
      console.error('Sync failed:', error)
    }
  }
}
```

## VIII. Service Worker vs. HTTP Cache

| Aspect | HTTP Cache | Service Worker Cache |
|--------|-----------|---------------------|
| **Control** | Server headers | JavaScript code |
| **Granularity** | Per-resource | Per-request (custom logic) |
| **Offline support** | No | Yes |
| **Background sync** | No | Yes |
| **Custom strategies** | No | Yes (cache-first, network-first, SWR) |
| **Complexity** | Low (headers only) | High (requires JS) |
| **Scope** | Browser/CDN | Browser only |
| **Use case** | Standard caching | PWA, offline-first |

## IX. Performance Considerations

| Tip | Why |
|-----|-----|
| **Cache only what you need** | Excessive caching wastes storage |
| **Version your cache names** | Easy to invalidate old caches |
| **Use `stale-while-revalidate`** | Best perceived performance |
| **Clean up old caches on activate** | Prevents storage bloat |
| **Don't cache POST requests** | Cache API only supports GET responses |
| **Monitor cache usage** | `navigator.storage.estimate()` to check quota |

## References

- [MDN: Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [web.dev: Service Workers](https://web.dev/learn/pwa/service-workers/)
- [Jake Archibald: The Offline Cookbook](https://jakearchibald.com/2014/offline-cookbook/)
- [Workbox](https://developer.chrome.com/docs/workbox/)
