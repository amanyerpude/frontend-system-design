# Webpage Rendering Cycle

## Overview

When a browser loads a page, it transforms HTML, CSS, and JavaScript into pixels on screen through a multi-step pipeline called the **Critical Rendering Path (CRP)**. Understanding this pipeline is essential for diagnosing and fixing performance issues.

## The Critical Rendering Path

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CRITICAL RENDERING PATH                          │
│                                                                     │
│  HTML → DOM ──────────┐                                             │
│                       ├→ Render Tree → Layout → Paint → Composite  │
│  CSS  → CSSOM ───────┘                                             │
│                                                                     │
│  JavaScript can modify DOM and CSSOM at any point before Paint      │
└─────────────────────────────────────────────────────────────────────┘
```

### Step 1: HTML Parsing → DOM

The browser reads HTML bytes and constructs the **Document Object Model (DOM)** — a tree of nodes.

```
Bytes: 3C 68 74 6D 6C 3E ...
         ↓
Characters: < h t m l >
         ↓
Tokens: <html>, <head>, <body>, <p>...
         ↓
Nodes: DOM Tree
```

**Performance impact:** HTML parsing is fast but can be blocked by:
- Render-blocking CSS (`<link rel="stylesheet">`)
- Synchronous JavaScript (`<script>` without `async`/`defer`)
- Large HTML documents

### Step 2: CSS Parsing → CSSOM

The browser reads CSS files and constructs the **CSS Object Model (CSSOM)** — a tree of style rules.

```
CSS: body { font-family: Arial; }
         ↓
CSSOM: { body: { font-family: Arial } }
```

**Performance impact:** CSS parsing blocks rendering. The browser will **not paint anything** until all render-blocking CSS is parsed and the CSSOM is built. This is why CSS in `<head>` blocks the first paint.

### Step 3: DOM + CSSOM → Render Tree

The browser combines the DOM and CSSOM to create the **Render Tree** — a tree of visible nodes with their computed styles.

**Key rule:** Elements with `display: none` are excluded from the render tree. Elements with `visibility: hidden` ARE included (they occupy space but are invisible).

```
DOM:                 CSSOM:              Render Tree:
html                 html{}              html (display: block)
├── head             body{font: Arial}   body (display: block, font: Arial)
├── body             h1{color: red}      h1 (display: block, color: red)
│   ├── h1           p{margin: 10px}     p (display: block, margin: 10px)
│   └── p
│       └── span     span{display:none}  span ← EXCLUDED
```

### Step 4: Layout (Reflow)

The browser calculates the **exact position and size** of every visible element. This step produces a "box model" for every node.

```
┌──────────────────────────────────────┐
│  Layout: Calculating box positions   │
│                                      │
│  body:    0,0,  1920×1080            │
│  header:  0,0,  1920×80              │
│  main:    0,80, 1920×920             │
│  footer:  0,1000,1920×80             │
└──────────────────────────────────────┘
```

**Performance impact:** Layout is one of the most expensive operations. Changes to any layout-affecting property (width, height, position, font-size, padding, margin, border) trigger a full re-layout of affected elements.

### Step 5: Paint

The browser fills in pixels for each visible element — colors, text, shadows, borders, backgrounds. This generates **paint records** (drawing instructions), not actual pixels yet.

```
Paint Records:
- Fill rect (0,0,1920,80) with #ffffff
- Draw text "Welcome" at (20, 20) in Arial 16px red
- Draw border (0,80,1920,920) 1px solid #ccc
```

**Performance impact:** Painting is expensive for complex elements (shadows, gradients, text with sub-pixel positioning). Large areas repaint slowly.

### Step 6: Composite

The browser combines all paint records into final pixels on screen. Modern browsers split the page into **layers** and composite them on the GPU.

```
Composite:
- Layer 1: Background
- Layer 2: Content
- Layer 3: Fixed header (separate layer)
- GPU combines all layers → final screen pixels
```

**Performance impact:** Compositing is fast when layers are separate. If the GPU must readback layers (e.g., during `mix-blend-mode` or `filter`), performance degrades.

## JavaScript's Role in the Pipeline

JavaScript can modify the DOM and CSSOM, which invalidates previous steps and forces re-computation:

```
JavaScript modifies DOM
         ↓
    Rebuild DOM (partial)
         ↓
    Rebuild Render Tree (partial)
         ↓
    Re-layout (reflow) if layout properties changed
         ↓
    Repaint if visual properties changed
         ↓
    Recomposite if layers changed
```

### Blocking Behavior

| Script Type | Blocking | When It Executes |
|-------------|----------|-----------------|
| `<script>` (synchronous) | **Blocks parsing AND rendering** | Immediately, halts HTML parsing |
| `<script async>` | **Blocks rendering, not parsing** | When downloaded (may block HTML parsing briefly) |
| `<script defer>` | **No blocking** | After HTML parsing completes, before DOMContentLoaded |
| `<script type="module">` | **No blocking** (default defer) | After HTML parsing, before DOMContentLoaded |

```html
<!-- ❌ BAD — blocks rendering -->
<script src="app.js"></script>

<!-- ✅ GOOD — async doesn't block HTML parsing -->
<script src="analytics.js" async></script>

<!-- ✅ GOOD — defer doesn't block anything -->
<script src="app.js" defer></script>
```

## The Rendering Pipeline in Practice

```
Navigation
    │
    ▼
HTML Response Received
    │
    ├──▶ Parse HTML → DOM Tree
    │         │
    │         ├──▶ Fetch CSS → CSSOM
    │         │         │
    │         │         ▼
    │         │    Build Render Tree
    │         │         │
    │         ▼         ▼
    │    JavaScript modifies DOM
    │         │
    │         ▼
    │    Re-layout / Repaint
    │         │
    │         ▼
    │    Composite
    │         │
    │         ▼
    │    Paint to Screen
    │
    └──▶ First Paint (FCP)
              │
              ▼
         Largest Contentful Paint (LCP)
              │
              ▼
         Interactive (TTI)
```

## Key Metrics Mapped to Pipeline

| Metric | Pipeline Stage | What It Measures |
|--------|---------------|------------------|
| **TTFB** | HTML response | Server response time |
| **FCP** | First paint | First pixels on screen |
| **LCP** | Largest paint | Main content rendered |
| **TTI** | After JavaScript | Page fully interactive |
| **CLS** | Layout shifts | Visual stability |

## Optimization Strategies

| Stage | Optimization |
|-------|-------------|
| **HTML** | Minify, compress, server-side rendering |
| **CSS** | Minify, remove unused CSS, critical CSS inline, async non-critical |
| **DOM** | Minimize depth, avoid synchronous layout reads |
| **Render Tree** | Use `display: none` for hidden elements |
| **Layout** | Avoid layout-triggering properties in animations |
| **Paint** | Reduce visual complexity, use GPU-composited properties |
| **Composite** | Use `transform` and `opacity` for animations |

## References

- [MDN: Critical Rendering Path](https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path)
- [web.dev: Rendering performance](https://web.dev/rendering-performance/)
- [Google: How the Browser Works](https://developer.chrome.com/blog/how-the-browser-runs-your-code/)
