# Focus Management

## I. What Is Focus Management?

Focus management is the practice of **controlling where keyboard focus moves** when the UI state changes — opening modals, navigating between pages, expanding accordions, or updating dynamic content. Poor focus management is one of the most common accessibility failures.

## II. Why Focus Management Matters

When focus is lost or misplaced, keyboard and screen reader users become disoriented:

```
❌ BAD: User clicks "Open Modal"
  → Modal appears on screen
  → But focus stays on the trigger button behind the modal
  → User tabs and gets lost, can't reach modal content
  → Screen reader doesn't announce the modal

✅ GOOD: User clicks "Open Modal"
  → Modal appears on screen
  → Focus moves to modal heading or first focusable element
  → Screen reader announces "Dialog: Checkout"
  → Tab cycles within modal (focus trap)
  → Escape closes modal, focus returns to trigger
```

## III. Focus Trapping

Focus trapping keeps focus within a container (typically a modal or dialog) until it's dismissed:

```typescript
function FocusTrap({ children, onClose }: { children: React.ReactNode; onClose: () => void }) {
  const containerRef = useRef<HTMLDivElement>(null)
  const previousFocus = useRef<HTMLElement | null>(null)

  useEffect(() => {
    // Store the element that had focus before modal opened
    previousFocus.current = document.activeElement as HTMLElement

    // Focus the first focusable element in the modal
    const firstFocusable = containerRef.current?.querySelector<HTMLElement>(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    )
    firstFocusable?.focus()

    return () => {
      // Restore focus when modal closes
      previousFocus.current?.focus()
    }
  }, [])

  const handleKeyDown = (event: KeyboardEvent) => {
    if (event.key === 'Escape') {
      onClose()
      return
    }

    if (event.key !== 'Tab') return

    const focusableElements = containerRef.current?.querySelectorAll<HTMLElement>(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    )
    if (!focusableElements) return

    const first = focusableElements[0]
    const last = focusableElements[focusableElements.length - 1]

    if (event.shiftKey) {
      // Shift+Tab: if on first element, move to last
      if (document.activeElement === first) {
        event.preventDefault()
        last.focus()
      }
    } else {
      // Tab: if on last element, move to first
      if (document.activeElement === last) {
        event.preventDefault()
        first.focus()
      }
    }
  }

  return (
    <div
      ref={containerRef}
      role="dialog"
      aria-modal="true"
      aria-labelledby="dialog-title"
      onKeyDown={handleKeyDown as any}
    >
      {children}
    </div>
  )
}
```

## IV. Focus Restoration

When a UI element closes, focus should return to the element that triggered it:

```typescript
function useFocusReturn() {
  const previousFocus = useRef<HTMLElement | null>(null)

  const saveFocus = () => {
    previousFocus.current = document.activeElement as HTMLElement
  }

  const restoreFocus = () => {
    previousFocus.current?.focus()
  }

  return { saveFocus, restoreFocus }
}

// Usage
function Modal() {
  const { saveFocus, restoreFocus } = useFocusReturn()

  const openModal = () => {
    saveFocus()
    // ... open modal
  }

  const closeModal = () => {
    // ... close modal
    restoreFocus()
  }
}
```

## V. Focus on Route Change

When navigating to a new page, focus should move to the main content or the page heading:

```typescript
import { useEffect, useRef } from 'react'
import { useLocation } from 'react-router-dom'

function RouteFocusManager() {
  const location = useLocation()
  const isFirstRender = useRef(true)

  useEffect(() => {
    // Don't focus on initial page load
    if (isFirstRender.current) {
      isFirstRender.current = false
      return
    }

    // Focus main content on route change
    const mainContent = document.getElementById('main-content')
    mainContent?.focus()

    // Announce page change for screen readers
    const announcement = document.getElementById('route-announcement')
    if (announcement) {
      announcement.textContent = `${document.title} page loaded`
    }
  }, [location.pathname])

  return (
    <>
      <div id="main-content" tabIndex={-1} />
      <div id="route-announcement" aria-live="polite" className="sr-only" />
    </>
  )
}
```

## VI. Focus in Dynamic Content

When content updates dynamically (search results, notifications), manage focus appropriately:

```typescript
// When new content is added, move focus to it
function SearchResults({ query, results }: Props) {
  const resultsRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    if (results.length > 0) {
      // Move focus to results heading
      resultsRef.current?.focus()
    }
  }, [results])

  return (
    <div>
      <h2 ref={resultsRef as any} tabIndex={-1}>
        {results.length} results for "{query}"
      </h2>
      <ul>{results.map(r => <li key={r.id}>{r.title}</li>)}</ul>
    </div>
  )
}
```

## VII. Focus Management Rules

| Scenario | Action |
|----------|--------|
| **Modal opens** | Move focus into modal, trap focus, announce dialog |
| **Modal closes** | Restore focus to trigger element |
| **Route changes** | Move focus to main content, announce page |
| **Accordion expands** | Keep focus on trigger, announce expanded state |
| **Dropdown opens** | Move focus to first item, arrow keys to navigate |
| **Toast/notification** | Announce via `aria-live`, don't steal focus |
| **Content updates** | Move focus to new content heading if significant |
| **Form validation fails** | Move focus to first error field |

## VIII. Common Focus Management Failures

| Failure | Impact | Fix |
|---------|--------|-----|
| **Lost focus on modal close** | User is "nowhere" in the tab order | Restore focus to trigger |
| **No focus trap in modal** | Tab escapes to background content | Implement focus trap |
| **Focus not moved on route change** | Screen reader doesn't announce new page | Focus main content on navigation |
| **Focus stolen by toast** | User loses context mid-task | Use `aria-live` instead of focus |
| **Dynamic content not announced** | User doesn't know content changed | Use `aria-live` for updates |

## IX. Testing Focus Management

1. **Tab through modal** — can you escape it? (Should not be able to)
2. **Close modal with Escape** — does focus return to the trigger?
3. **Navigate between pages** — does focus move to main content?
4. **Expand accordion** — does the content become accessible?
5. **Trigger form validation** — does focus move to the first error?

## X. Key Takeaways

1. **Manage focus when UI state changes** — modals, routes, dynamic content
2. **Trap focus in modals** — prevent escape to background
3. **Restore focus on close** — return to the trigger element
4. **Announce changes** — use `aria-live` for dynamic content, focus for navigation
5. **Never steal focus** for non-critical updates — use `aria-live` instead

## References

- [WAI-ARIA: Focus Management](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)
- [web.dev: Focus management](https://web.dev/learn/accessibility/focus/)
- [Reach UI: Focus Trap](https://reach-ui.com/docs/menu-button)
