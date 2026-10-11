# Real DOM vs Virtual DOM

## Overview

The **Real DOM** is the browser's actual representation of the page. The **Virtual DOM** is a lightweight JavaScript representation used by frameworks like React to minimize expensive DOM operations. Understanding the trade-offs is central to frontend performance.

## Real DOM

The Real DOM is the tree of nodes the browser creates from HTML. Every change to the Real DOM — adding an element, changing text, modifying styles — triggers the browser's rendering pipeline (reflow, repaint, composite).

### Characteristics

| Aspect | Behavior |
|--------|----------|
| **Representation** | Direct object model of HTML elements in memory |
| **Updates** | Any change triggers reflow and/or repaint |
| **Granularity** | Updates affect the entire document or subtree |
| **Speed per operation** | Slow — each operation triggers rendering pipeline work |
| **Memory** | Higher — stores full document structure, styles, events |
| **Complexity** | Direct, no abstraction layer |

### How DOM Updates Work (Without Virtual DOM)

```javascript
// Direct DOM manipulation — browser recalculates everything
const list = document.getElementById('list')

// Each of these triggers reflow + repaint
list.innerHTML = ''                              // Clear
list.appendChild(createItem('Item 1'))           // Add
list.appendChild(createItem('Item 2'))           // Add
list.appendChild(createItem('Item 3'))           // Add

// Even a small change triggers reflow
document.getElementById('count').textContent = '3 items'
```

Every DOM operation above triggers the browser's rendering pipeline. With many operations per frame, this causes visible jank.

## Virtual DOM

The Virtual DOM is a **lightweight JavaScript object tree** that mirrors the Real DOM. Frameworks use it to calculate the minimal set of DOM operations needed, then apply them in a batch.

### How It Works

```
1. State Change (e.g., setState in React)
         │
         ▼
2. Create new Virtual DOM tree (re-render)
         │
         ▼
3. Diff old Virtual DOM vs. new Virtual DOM
         │
         ▼
4. Calculate minimal set of Real DOM changes
         │
         ▼
5. Apply changes to Real DOM (batched)
```

### React's Fiber Architecture

React uses **Fiber** — a reimplementation of the Virtual DOM reconciler:

```
Fiber Node:
{
  type: 'div',
  key: null,
  props: { children: [...] },
  stateNode: DOMElement,     // Reference to Real DOM node
  child: FiberNode,          // First child
  sibling: FiberNode,        // Next sibling
  return: FiberNode,         // Parent
  effectTag: 'UPDATE',       // What to do when committing
}
```

**Key features:**
- **Incremental rendering** — Can pause and resume rendering
- **Priority lanes** — Different updates have different priorities
- **Time slicing** — Breaks work into chunks to avoid blocking main thread

### Diffing Algorithm

React's diffing algorithm compares old and new Virtual DOM trees:

| Strategy | What It Does |
|----------|-------------|
| **Tree diff** | If node type changes, destroy and rebuild subtree |
| **Component diff** | If component type changes, destroy and rebuild |
| **Element diff** | If same type, update only changed attributes |

```javascript
// Old Virtual DOM
<div className="container">
  <h1>Title</h1>
  <p>Content</p>
</div>

// New Virtual DOM (after state change)
<div className="container active">   {/* className changed */}
  <h1>Title</h1>                      {/* unchanged */}
  <p>Content</p>                      {/* unchanged */}
</div>

// Minimal Real DOM update: element.className = 'container active'
```

## Key Differences

| Aspect | Real DOM | Virtual DOM |
|--------|----------|-------------|
| **Representation** | Direct browser object model | JavaScript object tree |
| **Update speed** | Slow (triggers reflow/repaint) | Fast (in-memory comparison) |
| **Update granularity** | Per-element operations | Diffed, minimal batch updates |
| **Rendering** | Direct, no abstraction | Indirect via diff + batch |
| **Memory usage** | Browser-managed | Additional JS object overhead |
| **Debugging** | Direct inspection in DevTools | Requires framework DevTools |
| **Learning curve** | Direct API | Framework-specific abstraction |
| **Best for** | Simple pages, performance-critical | Complex applications, UI-heavy |

## Performance: When Virtual DOM Wins

The Virtual DOM wins when there are **many updates per frame**:

```javascript
// ❌ Real DOM — 100 individual operations
for (let i = 0; i < 100; i++) {
  list.appendChild(item)  // 100 reflow + repaint cycles
}

// ✅ Virtual DOM — 1 batch update
// React batches all 100 items into a single reconciliation
setState({ items: [...items, newItem] })
// React applies all 100 changes in one DOM update
```

## Performance: When Real DOM Wins

Direct DOM manipulation wins when there are **few, targeted updates**:

```javascript
// ✅ Real DOM — single, surgical update
document.getElementById('counter').textContent = '42'

// ❌ Virtual DOM — unnecessary overhead for one text change
// React re-renders the component, diffs the tree, then updates the text
setState({ count: 42 })
```

**Key insight:** The Virtual DOM adds overhead for simple updates. For complex applications with many concurrent state changes, the batching benefit outweighs the abstraction cost.

## React's Approach: Selective Virtual DOM

React 18+ optimizes by avoiding unnecessary Virtual DOM work:

- **`useMemo` / `useCallback`** — Memoize components to skip re-rendering
- **`React.memo`** — Skip re-render if props haven't changed
- **Concurrent features** — `useTransition` to deprioritize non-urgent updates
- **`useSyncExternalStore`** — Avoid tearing in concurrent mode

```tsx
// Skip re-render when props haven't changed
const ExpensiveList = React.memo(({ items }: { items: Item[] }) => {
  return <ul>{items.map(item => <li key={item.id}>{item.name}</li>)}</ul>
})
```

## Framework Comparison

| Framework | DOM Strategy | Notes |
|-----------|-------------|-------|
| **React** | Virtual DOM (Fiber) | Diff + batch |
| **Vue 3** | Virtual DOM (Patched) | Compiler-optimized, skips diff for static content |
| **Svelte** | No Virtual DOM | Compiler-based, directly updates Real DOM |
| **Solid.js** | No Virtual DOM | Fine-grained reactivity, surgical DOM updates |
| **Angular** | Incremental DOM | Similar to Virtual DOM but with Ivy renderer |

## Key Takeaways

1. **Virtual DOM is not always faster** — it's a trade-off between abstraction overhead and batching efficiency
2. **Real DOM is faster for simple updates** — one text change doesn't need diffing
3. **Virtual DOM wins for complex UIs** — many concurrent state changes benefit from batching
4. **Modern frameworks are converging** — Svelte, Solid.js, and Vue 3's compiler optimization reduce Virtual DOM overhead
5. **Measure, don't assume** — Profile before deciding which approach is faster for your use case

## References

- [React: Virtual DOM and Internals](https://reactjs.org/docs/faq-internals.html)
- [React Fiber Architecture](https://github.com/acdlite/react-fiber-architecture)
- [Why Virtual DOM? — Snabbdom](https://github.com/snabbdom/snabbdom)
- [Svelte: Reactivity](https://svelte.dev/docs/svelte/overview)
