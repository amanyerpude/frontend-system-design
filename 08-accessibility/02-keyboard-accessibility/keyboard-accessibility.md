# Keyboard Accessibility

## I. Why Keyboard Accessibility Matters

Keyboard accessibility ensures that **all interactive elements can be reached and operated using only a keyboard**. This is essential for users with motor impairments, users of switch devices, and power users. It's also a proxy for screen reader accessibility — screen readers use the same focus/navigation model.

## II. The Tab Order

The tab order determines the sequence in which elements receive focus when pressing Tab. By default, it follows the **DOM order** (not visual order).

```
Tab Order (DOM order):
1. Header nav links (in DOM order)
2. Main content links and buttons
3. Form inputs
4. Footer links
```

### Best Practices

| Rule | Why |
|------|-----|
| **DOM order should match visual order** | Users tabbing expect to move visually, not jump around |
| **Avoid positive `tabindex`** | Use `tabindex="0"` (focusable) or `tabindex="-1"` (programmatically focusable only) |
| **Don't use `tabindex="5"`** | Positive values create custom tab orders that are hard to maintain |
| **Use logical DOM structure** | Screen readers read in DOM order |

```html
<!-- ❌ BAD — positive tabindex creates custom order -->
<button tabindex="3">Third</button>
<button tabindex="1">First</button>
<button tabindex="2">Second</button>

<!-- ✅ GOOD — DOM order matches visual order -->
<button>First</button>
<button>Second</button>
<button>Third</button>
```

## III. Focusable Elements

By default, these elements are keyboard focusable:

| Element | Default Focusable | Notes |
|---------|-------------------|-------|
| `<a href="...">` | Yes | Links with `href` |
| `<button>` | Yes | Buttons |
| `<input>`, `<select>`, `<textarea>` | Yes | Form controls |
| `<summary>` | Yes | Disclosure widget |
| `<details>` | Yes | Expandable content |
| Elements with `tabindex="0"` | Yes | Any element made focusable |
| Elements with `tabindex="-1"` | No (programmatic only) | Focusable via JS but not Tab |

### Custom Interactive Elements

```html
<!-- ❌ BAD — div is not keyboard focusable -->
<div class="button" onclick="submit()">Submit</div>

<!-- ✅ GOOD — use a real button -->
<button type="button" onclick="submit()">Submit</button>

<!-- ✅ ACCEPTABLE — if you must use div, add tabindex and keyboard handler -->
<div
  class="button"
  role="button"
  tabindex="0"
  onclick="submit()"
  onkeydown="if(event.key==='Enter'||event.key===' ')submit()"
>
  Submit
</div>
```

## IV. Keyboard Navigation Patterns

### Tab Key

Moves focus forward through focusable elements.

### Shift + Tab

Moves focus backward.

### Enter

Activates links and buttons, submits forms.

### Space

Activates buttons, toggles checkboxes, scrolls page.

### Arrow Keys

Navigate within composite widgets (menus, tabs, radio buttons, listboxes).

### Escape

Closes modals, dropdowns, dialogs.

## V. Skip Navigation Links

A skip link allows keyboard users to bypass the header/nav and jump directly to main content:

```html
<body>
  <!-- Skip link — first focusable element -->
  <a href="#main-content" class="skip-link">Skip to main content</a>

  <header>
    <nav><!-- Long navigation with many links --></nav>
  </header>

  <main id="main-content" tabindex="-1">
    <!-- Main content -->
  </main>
</body>
```

```css
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  padding: 8px;
  background: black;
  color: white;
  z-index: 100;
}

.skip-link:focus {
  top: 0;  /* Visible when focused */
}
```

## VI. Focus Indicators

Users must be able to see where keyboard focus is:

```css
/* ❌ BAD — removing focus outline without replacement */
:focus {
  outline: none;  /* Users can't see where focus is! */
}

/* ✅ GOOD — visible focus indicator */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}

/* ✅ BETTER — custom focus style that's clearly visible */
:focus-visible {
  outline: 3px solid #4a90d9;
  outline-offset: 3px;
  border-radius: 2px;
}
```

**Key rule:** Never remove focus outlines without providing a visible replacement.

## VII. Complex Widget Patterns

### Modal Dialog

```
Tab behavior:
- Focus moves to the modal when it opens
- Tab cycles within the modal (trapped)
- Escape closes the modal
- Focus returns to the trigger element on close
```

### Dropdown Menu

```
Arrow key behavior:
- Down Arrow: opens menu, moves to next item
- Up Arrow: moves to previous item
- Enter/Space: selects item
- Escape: closes menu, returns focus to trigger
- Home/End: first/last item
```

### Tabs

```
Arrow key behavior:
- Right Arrow: next tab
- Left Arrow: previous tab
- Home: first tab
- End: last tab
- Tab: moves into the active tab panel
```

## VIII. Testing Keyboard Accessibility

1. **Unplug your mouse** — try to complete every task using only the keyboard
2. **Tab through every interactive element** — verify each is reachable and has a visible focus indicator
3. **Test Escape** — does it close modals, dropdowns, and dialogs?
4. **Test Enter/Space** — do buttons and links activate correctly?
5. **Test arrow keys** — do composite widgets (menus, tabs, listboxes) respond correctly?

## IX. Key Takeaways

1. **All interactive elements must be keyboard accessible** — use semantic HTML first
2. **DOM order should match visual order** — avoid positive tabindex
3. **Never remove focus outlines** without a visible replacement (`:focus-visible`)
4. **Skip navigation links** let keyboard users bypass repeated content
5. **Test with keyboard only** — unplug your mouse and try it

## References

- [MDN: Keyboard accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Keyboard-navigable_widgets)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/patterns/)
- [web.dev: Keyboard accessibility](https://web.dev/learn/accessibility/keyboard/)
