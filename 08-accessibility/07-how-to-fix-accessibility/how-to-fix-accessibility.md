# How to Fix Common Accessibility Issues

## I. Overview

This guide provides **direct solutions** to the most common accessibility violations found in frontend applications. Each issue includes the problem, the fix, and working code.

## II. Missing Alt Text on Images

**Problem:** Screen readers can't describe the image.

```html
<!-- ❌ BROKEN -->
<img src="hero.jpg">

<!-- ✅ FIXED — descriptive alt text -->
<img src="hero.jpg" alt="Team collaborating around a whiteboard">

<!-- ✅ FIXED — decorative image -->
<img src="divider.png" alt="">

<!-- ✅ FIXED — icon button -->
<button aria-label="Close dialog">
  <svg aria-hidden="true">✕</svg>
</button>
```

## III. Missing Form Labels

**Problem:** Screen readers can't identify what each input is for.

```html
<!-- ❌ BROKEN — placeholder disappears on focus -->
<input type="email" placeholder="Email">

<!-- ✅ FIXED — explicit label -->
<label for="email">Email</label>
<input type="email" id="email" name="email">

<!-- ✅ FIXED — visually hidden label -->
<label for="email" class="sr-only">Email address</label>
<input type="email" id="email" name="email">

<!-- ✅ FIXED — aria-label for icon-only inputs -->
<input type="search" aria-label="Search products">
```

## IV. Non-Interactive Elements with Click Handlers

**Problem:** `<div>`, `<span>`, and `<img>` are not keyboard focusable or operable.

```html
<!-- ❌ BROKEN -->
<div onclick="submit()">Submit</div>
<span onclick="toggle()">Toggle</span>

<!-- ✅ FIXED — use native elements -->
<button type="button" onclick="submit()">Submit</button>

<!-- ✅ FIXED — if you must use a div, add ARIA + keyboard -->
<div
  role="button"
  tabindex="0"
  onclick="submit()"
  onkeydown="if(event.key==='Enter'||event.key===' ') { event.preventDefault(); submit(); }"
>
  Submit
</div>
```

## V. Low Color Contrast

**Problem:** Text is unreadable for low-vision users.

```css
/* ❌ BROKEN */
.error-text { color: #ff6600; }  /* 2.83:1 on white */
.placeholder { color: #cccccc; }  /* 1.61:1 on white */

/* ✅ FIXED */
.error-text { color: #cc0000; }   /* 5.9:1 on white */
.placeholder { color: #767676; }   /* 4.54:1 on white */
```

## VI. Missing Focus Indicators

**Problem:** Keyboard users can't see where focus is.

```css
/* ❌ BROKEN */
:focus { outline: none; }

/* ✅ FIXED */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```

## VII. No Skip Navigation Link

**Problem:** Keyboard users must tab through the entire navigation on every page.

```html
<!-- ❌ BROKEN — no skip link -->
<body>
  <header>
    <nav><!-- 30 links --></nav>
  </header>
  <main><!-- Content that's hard to reach --></main>
</body>

<!-- ✅ FIXED -->
<body>
  <a href="#main" class="skip-link">Skip to main content</a>
  <header>
    <nav><!-- 30 links --></nav>
  </header>
  <main id="main" tabindex="-1"><!-- Content --></main>
</body>
```

```css
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
}
.skip-link:focus { top: 0; }
```

## VIII. Modal Without Focus Management

**Problem:** Focus isn't trapped in modal; focus isn't restored on close.

```tsx
<!-- ❌ BROKEN — no focus trap, no focus restoration -->
{isOpen && (
  <div className="modal">
    <h2>Dialog</h2>
    <button onClick={close}>Close</button>
  </div>
)}

<!-- ✅ FIXED — with focus trap and restoration -->
function Modal({ isOpen, onClose, children }) {
  const modalRef = useRef(null)
  const previousFocus = useRef(null)

  useEffect(() => {
    if (isOpen) {
      previousFocus.current = document.activeElement
      modalRef.current?.querySelector('button')?.focus()
    }
    return () => {
      previousFocus.current?.focus()
    }
  }, [isOpen])

  const handleKeyDown = (e) => {
    if (e.key === 'Escape') onClose()
    if (e.key === 'Tab') {
      const focusable = modalRef.current?.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      )
      const first = focusable[0]
      const last = focusable[focusable.length - 1]
      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault(); last.focus()
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault(); first.focus()
      }
    }
  }

  if (!isOpen) return null
  return (
    <div
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      onKeyDown={handleKeyDown}
    >
      <h2 id="modal-title">Dialog</h2>
      {children}
      <button onClick={onClose}>Close</button>
    </div>
  )
}
```

## IX. Dynamic Content Not Announced

**Problem:** Screen readers don't announce changes to dynamic content (search results, notifications).

```html
<!-- ❌ BROKEN — no announcement -->
<div id="results">
  <p>10 results found</p>
</div>

<!-- ✅ FIXED — aria-live region -->
<div aria-live="polite" aria-atomic="true">
  <p>10 results found</p>
</div>

<!-- ✅ FIXED — error announcement (assertive) -->
<div role="alert" aria-live="assertive">
  Error: Payment failed
</div>
```

## X. Decorative Icons Not Hidden

**Problem:** Screen readers announce decorative SVGs and icons.

```html
<!-- ❌ BROKEN — screen reader reads "image" -->
<img src="icon-star.svg">
<svg><path d="..."/></svg>

<!-- ✅ FIXED — hide decorative elements -->
<img src="icon-star.svg" alt="" aria-hidden="true">
<svg aria-hidden="true"><path d="..."/></svg>
```

## XI. Heading Hierarchy Skipping Levels

**Problem:** Screen readers can't navigate the document structure.

```html
<!-- ❌ BROKEN — skips from h1 to h3 -->
<h1>Main Title</h1>
<h3>Section Heading</h3>

<!-- ✅ FIXED — sequential heading levels -->
<h1>Main Title</h1>
<h2>Section Heading</h2>
```

## XII. Tables Without Headers

**Problem:** Screen readers can't associate data cells with headers.

```html
<!-- ❌ BROKEN -->
<table>
  <tr><td>Name</td><td>Email</td></tr>
  <tr><td>Alice</td><td>alice@example.com</td></tr>
</table>

<!-- ✅ FIXED — with proper headers -->
<table>
  <thead>
    <tr>
      <th scope="col">Name</th>
      <th scope="col">Email</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Alice</th>
      <td>alice@example.com</td>
    </tr>
  </tbody>
</table>
```

## XIII. Color as Only Indicator

**Problem:** Color-blind users can't distinguish states.

```html
<!-- ❌ BROKEN — color only -->
<span style="color: red;">Required</span>

<!-- ✅ FIXED — color + text/icon -->
<span class="field-label">
  <span aria-hidden="true">*</span> Required
</span>
```

## XIV. Fix Priority Checklist

| Priority | Issue | Impact |
|----------|-------|--------|
| **P0** | Missing form labels | Screen reader users can't use forms |
| **P0** | No keyboard navigation | Motor-impaired users can't use the site |
| **P0** | Missing alt text | Blind users can't understand images |
| **P1** | Low contrast | Low-vision users can't read content |
| **P1** | No focus indicator | Keyboard users can't see where they are |
| **P1** | Missing skip navigation | Keyboard users waste time navigating |
| **P2** | Modal without focus trap | Focus escapes, users get lost |
| **P2** | Dynamic content not announced | Screen reader users miss updates |
| **P3** | Heading hierarchy issues | Minor navigation difficulty |
| **P3** | Decorative icons not hidden | Minor screen reader noise |

## XV. Key Takeaways

1. **Use semantic HTML first** — it solves most accessibility issues automatically
2. **Every input needs a label** — placeholder is not a label
3. **All interactive elements must be keyboard accessible**
4. **Contrast must be ≥4.5:1** for normal text
5. **Never remove focus outlines** without a visible replacement
6. **Test with axe + screen reader + keyboard** — automated tools alone are insufficient

## References

- [WebAIM: Web Accessibility In Mind](https://webaim.org/)
- [A11Y Project Checklist](https://www.a11yproject.com/checklist/)
- [WCAG 2.1 Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/)
