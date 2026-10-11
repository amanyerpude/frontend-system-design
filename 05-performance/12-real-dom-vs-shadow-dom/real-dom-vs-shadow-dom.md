# Real DOM vs Shadow DOM

## Overview

The **Real DOM** is the standard document object model for web pages. The **Shadow DOM** is a separate, encapsulated DOM tree attached to a specific element. Both are trees of nodes, but they serve fundamentally different purposes: the Real DOM represents page content; the Shadow DOM provides **style and DOM encapsulation** for reusable components.

## Real DOM

The Real DOM is the tree of all elements in a page. Any CSS on the page affects any element, and any JavaScript can access any element.

```
Real DOM:
document
├── html
│   ├── head
│   │   └── link[rel="stylesheet"] ← Affects ALL elements
│   └── body
│       ├── header
│       ├── my-button             ← Component with Shadow DOM
│       └── footer
```

### Characteristics

| Aspect | Behavior |
|--------|----------|
| **Scope** | Global — all elements in one tree |
| **Styling** | Global CSS applies to all elements |
| **Access** | Any JS can read/modify any element |
| **Encapsulation** | None — components are vulnerable to style leakage |
| **Performance** | Larger tree = slower queries |

## Shadow DOM

The Shadow DOM is a **hidden, encapsulated DOM tree** attached to a specific element (the "host"). It has its own styles, its own scope, and its own events. Styles from the main document do not leak in, and styles inside the shadow tree do not leak out.

### Structure

```
Real DOM:
<body>
  <my-card>                     ← Host element
    #shadow-root                ← Shadow root (hidden)
      <style>                   ← Scoped styles (don't leak out)
        .title { color: blue; }  ← Only affects elements inside shadow root
      </style>
      <div class="title">Hello</div>
      <div class="content">Content</div>
  </my-card>
</body>
```

### Encapsulation Rules

| What | Inside Shadow DOM | Outside (Real DOM) |
|------|-------------------|-------------------|
| **CSS selectors** | Can select shadow elements | Cannot select shadow elements |
| **CSS inheritance** | Inherits from host | Does not affect shadow content |
| **JavaScript access** | `shadowRoot.querySelector()` | `document.querySelector()` does NOT find shadow elements |
| **Events** | Bubble (can be composed) | Can listen on host element |
| **Custom events** | Work normally | Must use `composed: true` to cross boundary |

### Web Components API

The Shadow DOM is the foundation of Web Components:

```javascript
class MyCard extends HTMLElement {
  constructor() {
    super()

    // Attach shadow DOM
    const shadow = this.attachShadow({ mode: 'open' })

    // Build scoped styles and markup
    shadow.innerHTML = `
      <style>
        :host { display: block; border: 1px solid #ccc; padding: 16px; }
        .title { font-size: 20px; font-weight: bold; color: #333; }
        .content { color: #666; margin-top: 8px; }
      </style>
      <div class="title"><slot name="title">Default Title</slot></div>
      <div class="content"><slot>Default Content</slot></div>
    `
  }
}

customElements.define('my-card', MyCard)
```

### Shadow DOM Modes

| Mode | Access | Use Case |
|------|--------|----------|
| **`open`** | Accessible via `element.shadowRoot` | Default, allows external inspection |
| **`closed`** | Not accessible via `element.shadowRoot` | Security-sensitive components |

## Key Differences

| Aspect | Real DOM | Shadow DOM |
|--------|----------|------------|
| **Scope** | Global (entire page) | Encapsulated (per component) |
| **Styling** | Global CSS applies everywhere | Styles are scoped to the shadow tree |
| **CSS leakage** | Yes — global styles affect components | No — styles don't cross the boundary |
| **JS access** | `document.querySelector()` | `element.shadowRoot.querySelector()` |
| **Performance** | Larger global tree | Smaller scoped trees |
| **Browser DevTools** | Elements panel | Separate shadow root panel in Elements |
| **Events** | Standard event bubbling | Events can be composed (cross boundary) |
| **Use case** | Standard page structure | Reusable, encapsulated components |

## CSS Encapsulation: The Main Difference

This is the primary reason Shadow DOM exists:

```html
<!-- Without Shadow DOM — style leakage -->
<style>
  .card { border: 1px solid red; }  /* Affects ALL .card elements */
  p { color: blue; }                 /* Affects ALL <p> elements */
</style>

<my-card>
  <!-- Shadow content — GLOBAL CSS ABOVE AFFECTS THIS -->
  <p>Card text — this <p> is affected by global styles</p>
</my-card>

<!-- With Shadow DOM — no style leakage -->
<my-card>
  #shadow-root
    <style>
      .card { border: 1px solid red; }  /* Only affects shadow content */
    </style>
    <div class="card">Shadow content — unaffected by global CSS</div>
</my-card>
```

### `:host` and `::slotted`

```css
/* Inside shadow root: style the host element */
:host {
  display: block;
  border: 1px solid #ccc;
}

:host(.active) {
  border-color: blue;
}

/* Style slotted content from outside */
::slotted(p) {
  color: red;
}
```

## Performance Implications

| Aspect | Real DOM | Shadow DOM |
|--------|----------|------------|
| **Query performance** | Slower — larger tree | Faster — smaller scoped trees |
| **Style recalculation** | Global invalidation | Scoped invalidation |
| **Rendering** | Single render tree | Separate render trees per shadow root |
| **Memory** | Single tree in memory | Additional tree per shadow root |
| **Event propagation** | Standard | Composed events cross boundaries |

### Performance Benefit: Scoped Style Recalculation

When a style changes inside a Shadow DOM, the browser only needs to recalculate styles within that shadow tree — not the entire page:

```
Without Shadow DOM:
Style change → Recalculate ALL elements in document

With Shadow DOM:
Style change in shadow tree → Recalculate only that shadow tree
```

## When to Use Each

| Scenario | Use |
|----------|-----|
| Standard page structure | Real DOM |
| Reusable, encapsulated component | Shadow DOM (Web Components) |
| Design system / component library | Shadow DOM |
| Application with controlled styling | Real DOM |
| Third-party component isolation | Shadow DOM |
| Micro-frontends | Shadow DOM for isolation |

## References

- [MDN: Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM)
- [MDN: Web Components](https://developer.mozilla.org/en-US/docs/Web/Web_Components)
- [Web Components MDN Guide](https://developer.mozilla.org/en-US/docs/Web/Web_Components)
- [CSS Scoping with Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/CSS/:host)
