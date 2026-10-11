# Progressive Web Apps (PWAs)

## I. What Is a PWA?

A **Progressive Web App (PWA)** is a web application that uses modern web capabilities to deliver an app-like experience — installable on devices, working offline, and sending push notifications. PWAs are built with standard web technologies (HTML, CSS, JavaScript) and enhanced with Service Workers, a Web App Manifest, and HTTPS.

## II. The Three Pillars of a PWA

```
┌─────────────────────────────────────────────────────┐
│                    PWA                               │
│                                                     │
│  ┌──────────────┐  ┌──────────┐  ┌──────────────┐  │
│  │ Service      │  │ Web App  │  │ HTTPS        │  │
│  │ Worker       │  │ Manifest │  │              │  │
│  │              │  │          │  │              │  │
│  │ Offline      │  │ Install  │  │ Secure       │  │
│  │ caching      │  │ prompt   │  │ context      │  │
│  │ Push         │  │ App icon │  │ Required for │  │
│  │ Background   │  │ Name     │  │ Service      │  │
│  │ sync         │  │ Theme    │  │ Workers      │  │
│  └──────────────┘  └──────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────┘
```

### Pillar 1: Service Workers

Provides offline capability, push notifications, and background sync.

### Pillar 2: Web App Manifest

A JSON file that tells the browser how the app should behave when installed (name, icons, theme, display mode).

### Pillar 3: HTTPS

Required for Service Workers — the browser refuses to register a Service Worker on an insecure origin.

## III. Web App Manifest

```json
// manifest.json
{
  "name": "My Application",
  "short_name": "MyApp",
  "description": "A Progressive Web App example",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "orientation": "portrait",
  "background_color": "#ffffff",
  "theme_color": "#0066cc",
  "icons": [
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-maskable.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ],
  "shortcuts": [
    {
      "name": "New Post",
      "url": "/compose",
      "icons": [{ "src": "/icons/new-post.png", "sizes": "96x96" }]
    }
  ]
}
```

### Link in HTML

```html
<head>
  <link rel="manifest" href="/manifest.json">
  <meta name="theme-color" content="#0066cc">
  <link rel="apple-touch-icon" href="/icons/icon-192.png">
</head>
```

### Manifest Properties

| Property | Purpose | Values |
|----------|---------|--------|
| `name` | Full app name | Any string |
| `short_name` | Name shown on home screen | Short string |
| `start_url` | URL loaded when app launches | Path |
| `scope` | Which URLs are in scope | Path |
| `display` | How the app is displayed | `standalone`, `fullscreen`, `minimal-ui`, `browser` |
| `orientation` | Preferred orientation | `portrait`, `landscape`, `any` |
| `background_color` | Splash screen background | Hex color |
| `theme_color` | Browser UI color | Hex color |
| `icons` | App icons | Array of icon objects |

## IV. Installability Criteria

Browsers show the install prompt when these criteria are met:

1. **HTTPS** — served over a secure connection
2. **Service Worker** — registered and controlling the page
3. **Web App Manifest** — linked in HTML with `name`, `icons`, `start_url`
4. **Engagement** — user has interacted with the site (varies by browser)

### Install Prompt

```typescript
let deferredPrompt: BeforeInstallPromptEvent | null = null

window.addEventListener('beforeinstallprompt', (event) => {
  // Prevent the default install prompt
  event.preventDefault()
  // Store the event for later use
  deferredPrompt = event
  // Show custom install button
  showInstallButton()
})

async function installPWA() {
  if (!deferredPrompt) return
  deferredPrompt.prompt()
  const { outcome } = await deferredPrompt.userChoice
  if (outcome === 'accepted') {
    console.log('PWA installed')
  }
  deferredPrompt = null
}
```

## V. Offline Support Patterns

### Pattern 1: App Shell (Cache First)

Cache the app shell (HTML, CSS, JS) and serve it offline:

```javascript
// sw.js
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('app-shell-v1').then(cache => {
      return cache.addAll([
        '/',
        '/styles.css',
        '/app.js',
        '/offline.html',
      ])
    })
  )
})

self.addEventListener('fetch', (event) => {
  if (event.request.mode === 'navigate') {
    event.respondWith(
      fetch(event.request)
        .then(response => {
          const clone = response.clone()
          caches.open('app-shell-v1').then(c => c.put(event.request, clone))
          return response
        })
        .catch(() => caches.match('/'))
    )
  }
})
```

### Pattern 2: Offline-First Data (Stale-While-Revalidate)

```javascript
// Serve cached API data, update in background
self.addEventListener('fetch', (event) => {
  if (event.request.url.includes('/api/')) {
    event.respondWith(
      caches.match(event.request).then(cached => {
        const fetchPromise = fetch(event.request).then(response => {
          caches.open('api-cache-v1').then(c => c.put(event.request, response.clone()))
          return response
        })
        return cached || fetchPromise
      })
    )
  }
})
```

### Pattern 3: Background Sync

Queue failed requests and retry when the user comes back online:

```javascript
// sw.js — background sync
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-posts') {
    event.waitUntil(syncPendingPosts())
  }
})

async function syncPendingPosts() {
  const pending = await getPendingPosts()  // From IndexedDB
  for (const post of pending) {
    try {
      await fetch('/api/posts', {
        method: 'POST',
        body: JSON.stringify(post.data),
        headers: { 'Content-Type': 'application/json' },
      })
      await removePendingPost(post.id)
    } catch (error) {
      console.error('Sync failed:', error)
    }
  }
}
```

## VI. Push Notifications

```typescript
// Request permission
async function requestNotificationPermission() {
  const permission = await Notification.requestPermission()
  if (permission === 'granted') {
    console.log('Notification permission granted')
  }
}

// Subscribe to push
async function subscribeToPush() {
  const registration = await navigator.serviceWorker.ready
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(
      'YOUR_VAPID_PUBLIC_KEY'
    ),
  })

  // Send subscription to server
  await fetch('/api/push/subscribe', {
    method: 'POST',
    body: JSON.stringify(subscription),
  })
}
```

```javascript
// sw.js — handle push notification
self.addEventListener('push', (event) => {
  const data = event.data.json()
  event.waitUntil(
    self.registration.showNotification(data.title, {
      body: data.body,
      icon: '/icons/icon-192.png',
      badge: '/icons/badge.png',
      data: data.url,
    })
  )
})

self.addEventListener('notificationclick', (event) => {
  event.notification.close()
  event.waitUntil(
    clients.openWindow(event.notification.data)
  )
})
```

## VII. Performance Benefits

| Metric | Impact |
|--------|--------|
| **First Load** | Same as regular web app |
| **Repeat Visits** | Faster (Service Worker serves from cache) |
| **Offline** | Fully functional (cached assets + data) |
| **Install** | Native app-like launch (no browser chrome) |
| **Push** | Re-engage users with notifications |

## VIII. PWA vs. Native App

| Aspect | PWA | Native App |
|--------|-----|------------|
| **Installation** | Browser install prompt | App Store |
| **Offline** | Yes (with Service Worker) | Yes |
| **Push notifications** | Yes (limited on iOS) | Yes |
| **Performance** | Near-native | Native |
| **Access to hardware** | Limited (camera, geolocation) | Full |
| **Distribution** | URL — no app store approval | App store review |
| **Updates** | Instant (server-side) | App store review |
| **Cost** | Free to distribute | App store fees |

## IX. iOS Limitations

iOS Safari has limited PWA support:

| Feature | iOS Support |
|---------|------------|
| Install to home screen | Yes (via Share → Add to Home Screen) |
| Service Worker | Yes (since iOS 11.3) |
| Push notifications | Limited (iOS 16.4+ for installed PWAs) |
| Background sync | No |
| Storage eviction | iOS may evict PWA storage after 7 days of no use |

### iOS Workaround for Storage Eviction

```typescript
// Call periodically to prevent storage eviction
navigator.storage.persist()
```

## X. Key Takeaways

1. **Three pillars:** Service Worker + Web App Manifest + HTTPS
2. **Installability** requires: HTTPS, Service Worker, manifest with name/icons/start_url
3. **Offline support** via Service Worker caching strategies
4. **Push notifications** re-engage users (limited on iOS)
5. **iOS has limitations** — background sync not supported, storage may be evicted
6. **PWAs are not a replacement for native** — they're a complement for web-first experiences

## References

- [web.dev: Learn PWA](https://web.dev/learn/pwa/)
- [MDN: Progressive Web Apps](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps)
- [Workbox](https://developer.chrome.com/docs/workbox/)
- [PWA Builder](https://www.pwabuilder.com/)
