# CSS Positioning

## Overview

CSS positioning determines how elements are placed in the page layout. The positioning model directly impacts rendering performance — some positioning strategies trigger expensive layout recalculations, while others are handled by the compositor on the GPU.

## Position Values

### `static` (Default)

Every element is `position: static` by default. Elements are positioned in normal document flow:
- Block-level elements stack vertically
- Inline elements flow horizontally
- `top`, `right`, `bottom`, `left`, and `z-index` have **no effect**

```css
.element {
  position: static; /* default */
}
```

**Performance:** No layout impact. The browser handles static elements in normal layout flow.

### `relative`

The element is positioned **relative to its normal position** in the document flow. It still occupies its original space in the layout.

```css
.element {
  position: relative;
  top: 10px;     /* Shift down 10px from normal position */
  left: 20px;    /* Shift right 20px from normal position */
}
```

**Performance:** Creates a positioning context but does not remove the element from flow. Minimal performance impact.

### `absolute`

The element is **removed from normal document flow** and positioned relative to its nearest positioned ancestor (any ancestor with `position` other than `static`).

```css
.parent {
  position: relative; /* Creates positioning context */
}

.child {
  position: absolute;
  top: 0;
  right: 0;
}
```

**Performance:** Removing an element from flow can trigger reflow of sibling elements. The element no longer affects the layout of other elements.

### `fixed`

The element is positioned **relative to the viewport** — it stays in place during scrolling. Also removed from normal flow.

```css
.header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
}
```

**Performance:** The compositor can handle `fixed` elements on the GPU (in modern browsers), making them cheap during scroll. However, creating a fixed element can trigger initial reflow.

### `sticky`

A hybrid of `relative` and `fixed`. The element behaves like `relative` until it reaches a specified scroll threshold, then behaves like `fixed`.

```css
.sidebar {
  position: sticky;
  top: 20px; /* Sticks 20px from top of viewport */
}
```

**Performance:** Modern browsers handle sticky positioning efficiently on the compositor. However, sticky elements can cause jank if the sticky container is complex.

## The Stacking Context

Elements with certain position values (or properties like `z-index`, `opacity`, `transform`) create a **stacking context** — a group of elements that are painted together as a unit.

```css
/* Creates a stacking context */
.container {
  position: relative;
  z-index: 10;
}
```

**Performance implication:** Each stacking context is composited as a separate layer. More stacking contexts → more layers → more GPU memory. Excessive layer creation can degrade performance.

## Static vs. Dynamic Positioning Performance

| Property | Layout Impact | Compositor Handled | Best For |
|----------|--------------|-------------------|----------|
| `static` | Normal flow | N/A | Default content |
| `relative` | Minimal | No | Subtle offsets, stacking context creation |
| `absolute` | Removes from flow | No | Overlays, badges, tooltips |
| `fixed` | Removes from flow | Yes | Headers, modals, floating buttons |
| `sticky` | Keeps in flow | Yes | Section headers, sidebars |

## Compositor-Only Properties (GPU-Accelerated)

These properties are handled entirely by the compositor — they do **not** trigger layout or paint, only composite:

```css
.element {
  transform: translateX(100px);  /* ✅ Compositor-only */
  opacity: 0.5;                   /* ✅ Compositor-only */
  filter: blur(5px);             /* ✅ Compositor-only (modern browsers) */
}
```

Compare with properties that **do** trigger layout:

```css
.element {
  top: 100px;           /* ❌ Triggers layout */
  left: 50px;           /* ❌ Triggers layout */
  width: 200px;         /* ❌ Triggers layout */
  margin: 10px;         /* ❌ Triggers layout */
  font-size: 20px;      /* ❌ Triggers layout */
}
```

### Animating Position

```css
/* ❌ BAD — Animating layout properties */
.slow-animation {
  animation: moveSlow 1s ease-in-out;
}
@keyframes moveSlow {
  0% { top: 0; left: 0; }
  100% { top: 100px; left: 100px; }
}

/* ✅ GOOD — Animating compositor-only properties */
.fast-animation {
  animation: moveFast 1s ease-in-out;
}
@keyframes moveFast {
  0% { transform: translate(0, 0); }
  100% { transform: translate(100px, 100px); }
}
```

## CSS Containment

CSS `contain` property tells the browser that an element's internal layout is independent of the rest of the page, enabling optimizations:

```css
.card {
  contain: layout style paint; /* This element's layout is isolated */
}

.heavy-widget {
  contain: content; /* Equivalent to layout style paint */
}

.image-gallery {
  contain: size layout paint; /* Also isolates size calculations */
}
```

**Performance impact:** The browser can skip recalculation of the entire page when a contained element changes — only recalculate within the containment boundary.

## Practical Positioning Performance Rules

1. **Animate `transform` and `opacity`** — They're compositor-only; all other properties trigger layout
2. **Use `position: fixed` for scroll-animated elements** — Modern browsers composite them on GPU
3. **Minimize stacking contexts** — Each one adds a GPU layer
4. **Use CSS containment** for complex independent components (cards, widgets)
5. **Prefer `transform: translate()`** over `top`/`left` for any positional animation
6. **Use `will-change` sparingly** — It hints to the browser to promote an element to its own layer, but each layer costs GPU memory

```css
/* ✅ Hint to compositor (use temporarily, not permanently) */
.animated-element {
  will-change: transform;
}
```

## References

- [MDN: Position](https://developer.mozilla.org/en-US/docs/Web/CSS/position)
- [MDN: Stacking context](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Positioning/Understanding_z_index/The_stacking_context)
- [web.dev: CSS containment](https://web.dev/strict-csp/)
- [MDN: CSS containment](https://developer.mozilla.org/en-US/docs/Web/CSS/contain)
