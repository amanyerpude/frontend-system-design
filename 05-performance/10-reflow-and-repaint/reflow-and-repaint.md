# Reflow and Repaint

## Overview

**Reflow** (layout) and **repaint** are the two most expensive operations in the browser rendering pipeline. When DOM changes trigger these operations excessively, the page becomes janky — animations stutter, scrolling lags, and interactions feel sluggish.

## Reflow (Layout)

Reflow is the process of **recalculating the position and size** of elements after a DOM or style change. It's the most expensive rendering operation.

### What Triggers Reflow

| Trigger | Example |
|---------|---------|
| **DOM manipulation** | Adding, removing, or changing elements |
| **Content changes** | Text update, image loading |
| **Style changes** | Width, height, padding, margin, border |
| **Font changes** | Loading a new web font |
| **Viewport resize** | Window resize, orientation change |
| **Reading layout properties** | `offsetHeight`, `getBoundingClientRect()`, `scrollTop` |

### Layout Thrashing

**Layout thrashing** occurs when JavaScript alternates between reading layout properties and writing DOM changes. Each read forces a synchronous reflow; each write invalidates the layout for the next read.

```typescript
// ❌ BAD — Layout thrashing
const elements = document.querySelectorAll('.item')
for (const el of elements) {
  const height = el.offsetHeight       // READ → forces synchronous reflow
  el.style.height = height * 2 + 'px'  // WRITE → invalidates layout
}

// ✅ GOOD — Batch reads, then batch writes
const elements = document.querySelectorAll('.item')
const heights = elements.map(el => el.offsetHeight)  // All reads
elements.forEach((el, i) => {
  el.style.height = heights[i] * 2 + 'px'  // All writes
})

// ✅ BETTER — Use requestAnimationFrame
requestAnimationFrame(() => {
  const heights = elements.map(el => el.offsetHeight)
  elements.forEach((el, i) => {
    el.style.height = heights[i] * 2 + 'px'
  })
})
```

### Forcing Synchronous Layout

Some JavaScript operations **force the browser to recalculate layout immediately** before returning a value:

```typescript
// These all force synchronous reflow:
el.offsetWidth
el.offsetHeight
el.offsetTop
el.offsetLeft
el.clientWidth
el.clientHeight
el.scrollTop
el.scrollLeft
el.getBoundingClientRect()
el.getClientRects()
window.getComputedStyle(el).width
window.innerHeight
```

**Rule:** Never read these properties after writing to the DOM in a loop. Batch reads first, then writes.

## Repaint

Repaint is the process of **redrawing the pixels** of visible elements that have changed visually but not in layout.

### What Triggers Repaint

| Trigger | Example |
|---------|---------|
| **Color changes** | `color`, `background-color` |
| **Visual changes** | `visibility`, `opacity` |
| **Shadow changes** | `box-shadow`, `text-shadow` |
| **Border style** | `border-style` (if width unchanged) |
| **Outline changes** | `outline`, `outline-color` |
| **Text decoration** | `text-decoration`, `text-decoration-color` |

### Repaint vs. Reflow

| Aspect | Reflow | Repaint |
|--------|--------|---------|
| **What it recalculates** | Position and size | Pixels for visual changes |
| **Cost** | More expensive | Less expensive |
| **When it runs** | After layout property changes | After visual property changes |
| **Forced by** | Reading layout properties | Reading paint properties |

**Key insight:** Reflow **always** triggers repaint (you must repaint after repositioning). But repaint does **not** trigger reflow. Always prefer repaint-only changes when possible.

## The DOM → Screen Pipeline (With Invalidations)

```
DOM/Style Change
       │
       ├─── Visual property change (color, shadow)
       │         │
       │         ▼
       │    Repaint → Composite → Screen
       │
       └─── Layout property change (width, position)
                  │
                  ▼
             Reflow (recalculate layout)
                  │
                  ▼
             Repaint (redraw pixels)
                  │
                  ▼
             Composite → Screen
```

## Performance Impact of DOM Operations

Not all DOM operations are equal:

| Operation | Cost | Trigger |
|-----------|------|---------|
| `element.textContent` | Low | Repaint only |
| `element.style.color` | Low | Repaint only |
| `element.style.width` | High | Reflow + repaint |
| `element.style.transform` | None | Composite only (GPU) |
| `element.style.opacity` | None | Composite only (GPU) |
| `element.remove()` | Very high | Reflow + repaint (full subtree) |
| `element.insertAdjacentHTML()` | Medium | Partial reflow |

## Compositor-Only Properties (No Reflow/Repaint)

These properties are handled by the GPU compositor — they don't trigger layout or paint:

```css
.animated {
  transform: translateX(100px);   /* ✅ Compositor-only */
  transform: scale(1.5);          /* ✅ Compositor-only */
  transform: rotate(45deg);       /* ✅ Compositor-only */
  opacity: 0.5;                   /* ✅ Compositor-only */
  filter: blur(10px);             /* ✅ Compositor-only (modern browsers) */
}
```

Compare with layout-triggering properties:

```css
.expensive {
  top: 100px;        /* ❌ Triggers reflow */
  left: 50px;        /* ❌ Triggers reflow */
  width: 200px;      /* ❌ Triggers reflow */
  margin: 10px;      /* ❌ Triggers reflow */
  font-size: 20px;   /* ❌ Triggers reflow */
  padding: 15px;     /* ❌ Triggers reflow */
}
```

## React and Reflow

React batches DOM updates, but certain patterns still cause unnecessary reflows:

```tsx
// ❌ BAD — Reading layout during render
function Component() {
  const height = document.querySelector('.sidebar').offsetHeight  // Forces reflow
  return <div style={{ marginTop: height }}>Content</div>
}

// ✅ GOOD — Use ref and useEffect for layout measurements
function Component() {
  const [sidebarHeight, setSidebarHeight] = useState(0)
  const sidebarRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    if (sidebarRef.current) {
      setSidebarHeight(sidebarRef.current.offsetHeight)
    }
  }, [])

  return (
    <>
      <div ref={sidebarRef}>Sidebar</div>
      <div style={{ marginTop: sidebarHeight }}>Content</div>
    </>
  )
}

// ✅ BETTER — Use ResizeObserver for dynamic sidebar
function Component() {
  const [sidebarHeight, setSidebarHeight] = useState(0)
  const sidebarRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    if (!sidebarRef.current) return

    const observer = new ResizeObserver((entries) => {
      for (const entry of entries) {
        setSidebarHeight(entry.contentRect.height)
      }
    })

    observer.observe(sidebarRef.current)
    return () => observer.disconnect()
  }, [])

  // ...
}
```

## Optimization Rules

1. **Batch DOM reads and writes** — Separate them; don't interleave
2. **Use `requestAnimationFrame`** for DOM updates that aren't user-input-driven
3. **Use `transform` and `opacity`** for animations (compositor-only)
4. **Minimize forced synchronous reflows** — Don't read layout properties in loops
5. **Use CSS containment** (`contain: layout`) to isolate expensive components
6. **Use `IntersectionObserver`** instead of scroll-based position checks
7. **Use `ResizeObserver`** instead of window resize handlers for element sizing
8. **Cache DOM measurements** — Don't read `offsetHeight` repeatedly if it hasn't changed

## References

- [MDN: Reflow](https://developer.mozilla.org/en-US/docs/Web/Performance/Understanding_Reflow)
- [What forces layout/reflow](https://gist.github.com/paulirish/5d52fb081b3570c81e3a)
- [web.dev: Reflow and repaint](https://web.dev/why-performance-matters/)
