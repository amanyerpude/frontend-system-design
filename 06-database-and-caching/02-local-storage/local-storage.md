# Local Storage

## I. Overview

Local Storage is a **persistent, synchronous, key-value storage** built into the browser. It stores data per origin (protocol + domain + port) and survives page reloads, browser restarts, and even browser updates — until explicitly cleared by the user or application.

## II. Core Characteristics

| Characteristic | Behavior |
|----------------|----------|
| **Persistence** | Survives page reloads, browser restarts, OS restarts |
| **Scope** | Per origin (protocol + domain + port) |
| **Capacity** | ~5–10MB (browser-dependent) |
| **API type** | Synchronous (blocks main thread) |
| **Data type** | Strings only (must serialize/deserialize) |
| **Expiration** | Never (until explicitly cleared) |
| **Sent with requests** | No (not automatically sent to server) |
| **Accessibility** | JavaScript, same-origin only |

## III. API Reference

```typescript
// Set an item
localStorage.setItem('user', JSON.stringify({ name: 'Alice', age: 30 }))

// Get an item
const user = JSON.parse(localStorage.getItem('user') || '{}')

// Remove an item
localStorage.removeItem('user')

// Get the number of items
const count = localStorage.length

// Get key at index
const key = localStorage.key(0)

// Clear all items for this origin
localStorage.clear()

// Check if key exists
const hasUser = localStorage.getItem('user') !== null
```

## IV. When to Use Local Storage

| Use Case | Example |
|----------|---------|
| **User preferences** | Theme, language, timezone |
| **Recently viewed items** | Last 10 products viewed |
| **Form draft autosave** | Preserve unsaved form data |
| **UI state** | Sidebar collapsed, grid/list view |
| **Onboarding flags** | "Has seen tutorial" |
| **Cached data** | API responses for offline fallback |

## V. When NOT to Use Local Storage

| Problem | Reason |
|---------|--------|
| **Authentication tokens** | XSS vulnerability — any JS can read localStorage |
| **Large datasets** | Limited to ~5–10MB |
| **Sensitive data** | Readable by any script on the page |
| **Frequently changing data** | Synchronous API blocks main thread |
| **Data that must sync to server** | No automatic sync mechanism |
| **Shared data between tabs** | No built-in cross-tab sync (must use events) |

## VI. The Serialization Tax

Local Storage only stores strings. Every non-string value must be serialized and deserialized:

```typescript
// Writing
localStorage.setItem('settings', JSON.stringify({
  theme: 'dark',
  fontSize: 16,
  notifications: true,
}))

// Reading
const settings = JSON.parse(localStorage.getItem('settings') || '{}')

// ❌ BAD — parsing on every access
function getTheme() {
  return JSON.parse(localStorage.getItem('settings') || '{}').theme
}

// ✅ GOOD — cache in memory, write back on change
let cachedSettings: Settings | null = null

function getSettings(): Settings {
  if (!cachedSettings) {
    cachedSettings = JSON.parse(localStorage.getItem('settings') || '{}')
  }
  return cachedSettings
}

function updateSettings(patch: Partial<Settings>) {
  cachedSettings = { ...getSettings(), ...patch }
  localStorage.setItem('settings', JSON.stringify(cachedSettings))
}
```

## VII. Cross-Tab Communication

Local Storage is shared across tabs of the same origin. Changes trigger the `storage` event in **other** tabs (not the tab that made the change):

```typescript
// Tab A: Write
localStorage.setItem('user', JSON.stringify({ name: 'Bob' }))

// Tab B: Listen for changes
window.addEventListener('storage', (event) => {
  if (event.key === 'user') {
    const newUser = JSON.parse(event.newValue || '{}')
    console.log('User changed in another tab:', newUser)
    // Update UI
  }
})
```

## VIII. Security Considerations

| Concern | Detail |
|---------|--------|
| **XSS vulnerability** | Any script on the page can read localStorage. If an attacker injects `<script>`, they can steal tokens, preferences, everything |
| **No encryption** | Data is stored in plain text; visible in DevTools → Application → Local Storage |
| **No server access** | localStorage is never sent with HTTP requests (unlike cookies) |
| **Same-origin only** | Other origins cannot read your localStorage |

### Safer Alternative for Auth Tokens

```typescript
// ❌ UNSAFE — localStorage is vulnerable to XSS
localStorage.setItem('token', jwtToken)

// ✅ SAFER — HttpOnly cookie (JavaScript cannot read it)
// Server sets: Set-Cookie: token=abc123; HttpOnly; Secure; SameSite=Strict
// JavaScript cannot access this cookie via document.cookie
```

## IX. Performance Considerations

- **Synchronous API** — `localStorage.getItem()` and `setItem()` block the main thread
- **Small data** — Don't store large datasets; use IndexedDB
- **Batch operations** — Read once, cache in memory, write back on change
- **Avoid in loops** — Never call localStorage inside a loop or animation frame

```typescript
// ❌ BAD — synchronous reads in a loop (blocks main thread)
for (const key of keys) {
  const value = localStorage.getItem(key)  // Each call blocks
}

// ✅ GOOD — read once, process in memory
const values = keys.map(key => localStorage.getItem(key))
```

## X. Browser Quirks

| Quirk | Detail |
|-------|--------|
| **Private browsing** | Some browsers throw errors when exceeding quota in private mode |
| **Storage events** | Don't fire in the tab that made the change (only in other tabs) |
| **Quota** | Exceeding quota throws `QuotaExceededError` |
| **Deletion** | Only the user can clear data (via browser settings), or via `clear()` |
| **Iframe origins** | Each iframe has its own localStorage (different origin = separate storage) |

## XI. Decision Framework

```
Need to persist data on the client?
├── Is it sensitive/auth-related? → Use HttpOnly cookies
├── Is it small (<5MB)? → Local Storage
├── Is it large or structured? → IndexedDB
├── Is it session-only? → Session Storage or memory
└── Does it need server sync? → Server-side storage
```

## References

- [MDN: Window.localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [web.dev: Storage for the web](https://web.dev/learn/pwa/storage/)
- [OWASP: XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Scripting_Prevention_Cheat_Sheet.html)
