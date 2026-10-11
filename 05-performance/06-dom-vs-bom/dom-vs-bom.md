# DOM vs BOM

## Overview

The **DOM (Document Object Model)** and **BOM (Browser Object Model)** are two distinct interfaces for interacting with web pages. Understanding the difference is fundamental to frontend development and performance optimization.

## DOM — Document Object Model

The DOM is the **programmatic representation of the HTML document structure**. It's a tree of nodes that the browser creates when it parses HTML. The DOM API lets you read, modify, add, and remove elements and attributes.

```
DOM Tree:
document
├── html
│   ├── head
│   │   ├── title
│   │   └── link[rel="stylesheet"]
│   └── body
│       ├── header
│       │   └── nav
│       │       └── ul
│       │           └── li
│       └── main
│           ├── article
│           └── footer
```

### What the DOM Covers

| API | Purpose |
|-----|---------|
| `document.getElementById()` | Find element by ID |
| `document.querySelector()` | CSS selector query |
| `element.appendChild()` | Add child node |
| `element.addEventListener()` | Attach event handlers |
| `element.style` | Modify inline styles |
| `element.textContent` | Read/change text content |
| `document.createElement()` | Create new element |
| `element.getBoundingClientRect()` | Get position and size |
| `MutationObserver` | Watch for DOM changes |
| `IntersectionObserver` | Watch for viewport visibility |
| `ResizeObserver` | Watch for element size changes |

### DOM Performance Implications

- **DOM manipulation is expensive** — Each change can trigger layout recalculation (reflow) and paint
- **Accessing the DOM is slow** — Reading properties like `offsetHeight` forces layout
- **Batch DOM reads and writes** — Separate reading from writing to avoid layout thrashing
- **Minimize DOM depth** — Deeper trees increase query time and layout complexity

## BOM — Browser Object Model

The BOM is the **programmatic interface for the browser window itself** — not the document. It provides APIs for interacting with the browser environment.

```
BOM Hierarchy:
window
├── navigator    (browser info)
├── location     (URL, navigation)
├── history      (back/forward)
├── screen       (display info)
├── localStorage (persistent storage)
├── sessionStorage (session storage)
├── document     (reference to DOM)
├── fetch()      (network requests)
├── setTimeout() / setInterval() (timers)
├── requestAnimationFrame() (frame callbacks)
└── performance  (timing metrics)
```

### What the BOM Covers

| API | Purpose |
|-----|---------|
| `window.location` | Read/modify URL, navigate |
| `window.history` | Back/forward navigation |
| `window.navigator` | Browser/OS information |
| `window.screen` | Screen dimensions, color depth |
| `window.localStorage` | Persistent key-value storage |
| `window.sessionStorage` | Session-scoped storage |
| `window.fetch()` | HTTP requests |
| `window.setTimeout()` | Timer callbacks |
| `window.requestAnimationFrame()` | Frame-synced callbacks |
| `window.performance` | Timing and profiling |
| `window.open()` / `window.close()` | Window management |
| `window.matchMedia()` | Media query evaluation |
| `window.getComputedStyle()` | Computed CSS properties |
| `window.scrollTo()` | Programmatic scrolling |

### BOM Performance Implications

- **`window.getComputedStyle()`** — Forces style recalculation; don't call in loops
- **`window.scroll` events** — Fire continuously during scroll; use throttling or IntersectionObserver
- **`localStorage`** — Synchronous API; blocks the main thread; avoid for large data
- **`requestAnimationFrame`** — Preferred over `setTimeout` for animations (syncs to frame rate)

## Key Differences

| Aspect | DOM | BOM |
|--------|-----|-----|
| **Represents** | The HTML document structure | The browser window/environment |
| **Root** | `document` | `window` |
| **Standardized** | W3C standard | Browser-specific (no single standard) |
| **Portability** | Same across browsers | May differ between browsers |
| **Accessed via** | `document.*` | `window.*` |
| **Includes** | Elements, attributes, text nodes | navigator, location, history, storage, timers |
| **Performance concern** | DOM manipulation → reflow/repaint | Synchronous APIs block main thread |

## DOM and BOM Work Together

In practice, the DOM and BOM are deeply interconnected:

```typescript
// BOM: Get viewport dimensions
const { innerWidth, innerHeight } = window

// DOM: Check if element is within viewport
const element = document.querySelector('.hero')
const rect = element.getBoundingClientRect()
const isInViewport = rect.top < innerHeight && rect.bottom > 0

// BOM: Listen for resize, then check DOM
window.addEventListener('resize', () => {
  const newRect = element.getBoundingClientRect()
  // Update DOM based on new viewport
})
```

## Performance Best Practices

| Practice | Reason |
|----------|--------|
| **Minimize DOM depth** | Faster queries, less layout work |
| **Batch DOM operations** | Read all, then write all (avoid layout thrashing) |
| **Use `requestAnimationFrame`** | Sync animations to display refresh rate |
| **Use `IntersectionObserver`** | Instead of scroll events for viewport detection |
| **Use `requestIdleCallback`** | Defer non-urgent work |
| **Avoid synchronous storage** | `localStorage` blocks main thread |
| **Cache `getComputedStyle`** | Avoid repeated calls in loops |
| **Use `DocumentFragment`** | Batch DOM insertions before appending to DOM |

### Example: Avoiding Layout Thrashing

```typescript
// ❌ BAD — Layout thrashing (alternating reads and writes)
for (const el of elements) {
  const height = el.offsetHeight    // READ → forces layout
  el.style.height = height * 2 + 'px'  // WRITE → invalidates layout
}

// ✅ GOOD — Batch reads, then batch writes
const heights = elements.map(el => el.offsetHeight)  // All reads
elements.forEach((el, i) => {
  el.style.height = heights[i] * 2 + 'px'  // All writes
})
```

## References

- [MDN: Document Object Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
- [MDN: Window Interface](https://developer.mozilla.org/en-US/docs/Web/API/Window)
- [MDN: Layout thrashing](https://developer.mozilla.org/en-US/docs/Web/API/Window/scroll)
- [What the heck is the event loop?](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
