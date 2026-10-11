# Screen Reader Accessibility

## I. What Is a Screen Reader?

A screen reader is **assistive technology that converts on-screen content into speech or Braille output**. It's the primary way blind and visually impaired users interact with web applications. Screen readers navigate using a combination of keyboard input and semantic information from the HTML and ARIA.

## II. How Screen Readers Navigate

Screen readers expose different "views" of the page:

| Navigation Method | Description | Shortcut (NVDA/JAWS) |
|-------------------|-------------|---------------------|
| **Headings** | Jump between H1–H6 | `H` |
| **Links** | Jump between links | `K` |
| **Form fields** | Jump between inputs | `F` |
| **Buttons** | Jump between buttons | `B` |
| **Landmarks** | Jump between page regions (nav, main, aside) | `D` |
| **Lists** | Navigate list items | `L` |
| **Tables** | Navigate table cells | `T` |
| **Graphics** | Jump between images | `G` |

**Key insight:** If your page uses semantic HTML, screen readers can navigate it efficiently. If you use `<div>` for everything, users must read through the entire page sequentially.

## III. Semantic HTML Is the Foundation

Semantic HTML provides screen readers with information about the structure and purpose of elements:

| Semantic Element | Screen Reader Announces | Purpose |
|-----------------|------------------------|---------|
| `<nav>` | "Navigation landmark" | Page navigation |
| `<main>` | "Main landmark" | Primary content |
| `<header>` | "Banner" | Page header |
| `<footer>` | "Contentinfo" | Page footer |
| `<article>` | "Article" | Self-contained content |
| `<aside>` | "Complementary" | Related content |
| `<h1>`–`<h6>` | "Heading level 1"–"6" | Document structure |
| `<button>` | "Button" | Action |
| `<a href>` | "Link" | Navigation |
| `<ul>`/`<ol>` | "List" | List of items |
| `<table>` | "Table with X rows and Y columns" | Tabular data |
| `<label>` | Labels form controls | Input identification |

```html
<!-- ❌ BAD — screen reader says "group of nothing" -->
<div class="nav">
  <div class="nav-item"><div onclick="go('/home')">Home</div></div>
</div>

<!-- ✅ GOOD — screen reader says "Navigation landmark, Link: Home" -->
<nav aria-label="Main navigation">
  <ul>
    <li><a href="/home">Home</a></li>
  </ul>
</nav>
```

## IV. Alt Text for Images

Alt text describes images for screen reader users:

```html
<!-- ✅ GOOD — meaningful alt text -->
<img src="chart.png" alt="Q4 revenue increased 15% over Q3">

<!-- ✅ GOOD — decorative image (empty alt) -->
<img src="divider.png" alt="">

<!-- ❌ BAD — missing alt (screen reader reads filename) -->
<img src="chart.png">

<!-- ❌ BAD — unhelpful alt text -->
<img src="chart.png" alt="chart image">
```

### Alt Text Rules

| Image Type | Alt Text |
|------------|----------|
| **Informative** | Describe the content/purpose |
| **Decorative** | Empty `alt=""` |
| **Functional** (icon button) | Describe the action ("Close menu", "Search") |
| **Complex** (chart, diagram) | Brief alt + long description nearby |
| **Text** | Use the text itself as alt |

## V. ARIA (Accessible Rich Internet Applications)

ARIA supplements HTML when semantic elements aren't sufficient — primarily for custom widgets and dynamic content.

### The First Rule of ARIA

> **If you can use a native HTML element with the correct semantics and behavior, do that.** ARIA should be a last resort.

```html
<!-- ❌ BAD — ARIA used unnecessarily -->
<div role="button" tabindex="0" onclick="submit()">Submit</div>

<!-- ✅ GOOD — native button -->
<button type="button" onclick="submit()">Submit</button>
```

### When ARIA Is Necessary

| Scenario | ARIA Usage |
|----------|-----------|
| **Custom widget** (combobox, tree, slider) | `role`, `aria-expanded`, `aria-selected`, etc. |
| **Dynamic content** | `aria-live` for announcements |
| **Custom checkbox/toggle** | `role="checkbox"` + `aria-checked` |
| **Progress indicator** | `role="progressbar"` + `aria-valuenow` |
| **Icon-only button** | `aria-label` for accessible name |

### ARIA Live Regions

Announce dynamic content changes to screen readers:

```html
<!-- Polite: announce after current announcement -->
<div aria-live="polite">
  3 items in cart
</div>

<!-- Assertive: interrupt current announcement -->
<div aria-live="assertive" role="alert">
  Error: Payment failed
</div>
```

### Common ARIA Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `aria-label` | Accessible name (when no visible label) | `aria-label="Close menu"` |
| `aria-labelledby` | Reference element as label | `aria-labelledby="heading-id"` |
| `aria-describedby` | Additional description | `aria-describedby="help-text"` |
| `aria-expanded` | Widget open/closed state | `aria-expanded="true"` |
| `aria-hidden` | Hide from screen readers | `aria-hidden="true"` (decorative icons) |
| `aria-live` | Dynamic content announcements | `aria-live="polite"` |
| `aria-current` | Current item in set | `aria-current="page"` |
| `aria-disabled` | Disabled state | `aria-disabled="true"` |

## VI. Form Accessibility

```html
<!-- ❌ BAD — placeholder is not a label -->
<input type="email" placeholder="Email address">

<!-- ✅ GOOD — explicit label linked to input -->
<label for="email">Email address</label>
<input type="email" id="email" name="email">

<!-- ✅ GOOD — visually hidden label (if no visible label needed) -->
<label for="email" class="sr-only">Email address</label>
<input type="email" id="email" name="email">

<!-- ✅ GOOD — aria-label for icon-only inputs -->
<input type="search" aria-label="Search products">
```

### Screen Reader Hidden Text

```css
/* Visually hidden but available to screen readers */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

## VII. Testing with Screen Readers

| Screen Reader | Browser | Platform |
|---------------|---------|----------|
| **NVDA** | Firefox, Chrome | Windows (free) |
| **JAWS** | Chrome, Firefox | Windows (paid) |
| **VoiceOver** | Safari | macOS, iOS (built-in) |
| **TalkBack** | Chrome | Android (built-in) |

### Quick Test (macOS)

1. Enable VoiceOver: `Cmd + F5`
2. Navigate: `VO + Arrow keys` (VO = Control + Option)
3. Interact with element: `VO + Shift + Down Arrow`
4. Stop: `Cmd + F5` again

## VIII. Common Screen Reader Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Reads filename for images | Missing `alt` attribute | Add descriptive alt text |
| Reads "button" for no reason | `aria-hidden` not set on decorative element | Add `aria-hidden="true"` |
| Skips content | Content hidden with `display:none` | Use `.sr-only` if content should be read |
| Reads empty links | Links with no text or aria-label | Add text content or `aria-label` |
| No announcement on state change | Missing `aria-live` | Add `aria-live="polite"` or `aria-live="assertive"` |

## IX. Key Takeaways

1. **Semantic HTML is the foundation** — ARIA is a supplement, not a replacement
2. **Alt text describes image content**, not appearance
3. **ARIA live regions** announce dynamic changes
4. **Test with real screen readers** — NVDA (free) and VoiceOver (built-in)
5. **First rule of ARIA:** use native HTML if possible

## References

- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- [WebAIM: Screen Reader Testing](https://webaim.org/projects/screenreadersurvey10/)
- [MDN: ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA)
