# Session Storage

## I. Overview

Session Storage is a **per-tab, synchronous, key-value storage** built into the browser. It stores data per origin per tab and is cleared when the tab is closed. It shares the same API as Local Storage but with different scope and persistence.

## II. Core Characteristics

| Characteristic | Behavior |
|----------------|----------|
| **Persistence** | Survives page reloads; cleared when tab is closed |
| **Scope** | Per origin per tab (new tab = fresh storage) |
| **Capacity** | ~5–10MB (browser-dependent) |
| **API type** | Synchronous (blocks main thread) |
| **Data type** | Strings only (must serialize/deserialize) |
| **Expiration** | Tab close, navigation to other origin |
| **Sent with requests** | No |
| **Accessibility** | JavaScript, same-origin, same-tab only |

## III. API Reference

The API is identical to Local Storage:

```typescript
// Set an item
sessionStorage.setItem('formDraft', JSON.stringify({ step: 2, data: {...} }))

// Get an item
const draft = JSON.parse(sessionStorage.getItem('formDraft') || '{}')

// Remove an item
sessionStorage.removeItem('formDraft')

// Clear all items for this tab
sessionStorage.clear()

// Get key at index
const key = sessionStorage.key(0)
```

## IV. Session Storage vs. Local Storage

| Aspect | Session Storage | Local Storage |
|--------|----------------|---------------|
| **Persistence** | Tab lifetime | Forever (until cleared) |
| **Scope** | Per tab | Shared across all tabs |
| **Cross-tab sync** | No (each tab isolated) | Yes (via `storage` event) |
| **Capacity** | ~5–10MB | ~5–10MB |
| **Use case** | Session-scoped state | Long-term preferences |

**Key difference:** Opening a new tab gives you a fresh Session Storage — it does **not** copy Local Storage. Each tab has its own isolated Session Storage.

## V. When to Use Session Storage

| Use Case | Example |
|----------|---------|
| **Multi-step form wizard** | Preserve form state as user navigates between steps |
| **Tab-specific context** | Temporary filters, sort orders for current view |
| **Onboarding flow** | Track which step the user is on (cleared on completion) |
| **Temporary data** | Data that's only relevant for the current browsing session |
| **Preventing duplicate form submission** | Track whether form has been submitted in this tab |

```typescript
// Multi-step form wizard using sessionStorage
const WIZARD_KEY = 'wizard_state'

interface WizardState {
  currentStep: number
  step1: { name: string; email: string }
  step2: { address: string; city: string }
  step3: { payment: string }
}

function saveWizardStep(state: Partial<WizardState>) {
  const current = JSON.parse(sessionStorage.getItem(WIZARD_KEY) || '{}')
  sessionStorage.setItem(WIZARD_KEY, JSON.stringify({ ...current, ...state }))
}

function loadWizardStep(): Partial<WizardState> {
  return JSON.parse(sessionStorage.getItem(WIZARD_KEY) || '{}')
}

function completeWizard() {
  sessionStorage.removeItem(WIZARD_KEY)  // Clear on completion
}
```

## VI. When NOT to Use Session Storage

| Problem | Reason |
|---------|--------|
| **Cross-tab data sharing** | Each tab has isolated storage |
| **Long-term persistence** | Data is lost when tab closes |
| **Authentication tokens** | Same XSS vulnerability as localStorage |
| **Large datasets** | Limited to ~5–10MB; use IndexedDB |
| **Data that must survive refresh** | User might lose work on accidental refresh |

## VII. Cross-Tab Communication Limitation

Session Storage is **per-tab** and does **not** support cross-tab communication:

```typescript
// Tab A
sessionStorage.setItem('message', 'hello')

// Tab B — sessionStorage is EMPTY (different tab)
sessionStorage.getItem('message')  // null

// ❌ The storage event does NOT fire for sessionStorage changes
// ✅ For cross-tab communication, use localStorage or BroadcastChannel
```

### Cross-Tab Communication Alternatives

```typescript
// Option 1: localStorage with storage event
window.addEventListener('storage', (e) => {
  if (e.key === 'sync-data') { /* ... */ }
})

// Option 2: BroadcastChannel API (modern, preferred)
const channel = new BroadcastChannel('app-sync')
channel.postMessage({ type: 'data-update', payload: {...} })

channel.onmessage = (event) => {
  const { type, payload } = event.data
  // Handle message
}
```

## VIII. The `pageshow`/`pagehide` Event

Session Storage survives page reloads (including bfcache restores). Handle this appropriately:

```typescript
window.addEventListener('pagehide', () => {
  // Save state before page is hidden (including bfcache)
  saveWizardStep({ currentStep: getCurrentStep() })
})

window.addEventListener('pageshow', (event) => {
  // Restore state after page is shown (including from bfcache)
  if (event.persisted) {
    // Page restored from bfcache
    const state = loadWizardStep()
    if (state.currentStep) {
      restoreWizard(state)
    }
  }
})
```

## IX. Security Considerations

Session Storage has the same XSS vulnerability as Local Storage:

- Readable by any script on the page
- Not sent with HTTP requests
- Not encrypted
- Cleared when tab closes (reduces risk vs. localStorage)

**For authentication tokens**, use HttpOnly cookies instead — JavaScript cannot read them, making them immune to XSS theft.

## X. Performance

- **Synchronous API** — blocks main thread on read/write
- **Same performance profile** as Local Storage
- **Batch operations** — read once, cache in memory, write back on change
- **Avoid in loops** — never call sessionStorage in a loop or animation frame

## XI. Decision Framework

```
Data is session-scoped (only needed in current tab)?
├── Yes, tab-specific → Session Storage
├── Yes, but needs cross-tab → localStorage + storage event, or BroadcastChannel
├── No, needs to persist → Local Storage
├── No, needs server sync → Cookies or server-side
└── Large data → IndexedDB
```

## References

- [MDN: Window.sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [MDN: BroadcastChannel API](https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel)
- [web.dev: Storage for the web](https://web.dev/learn/pwa/storage/)
