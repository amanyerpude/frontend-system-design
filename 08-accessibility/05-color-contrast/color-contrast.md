# Color Contrast

## I. Why Color Contrast Matters

Color contrast determines **whether text and interactive elements are readable against their background**. Insufficient contrast makes content unreadable for users with low vision, color blindness, or those viewing screens in bright sunlight. It's one of the most commonly flagged accessibility failures.

## II. WCAG Contrast Requirements

### Minimum Contrast Ratios

| Content Type | WCAG AA | WCAG AAA |
|-------------|---------|----------|
| **Normal text** (<18px / <14px bold) | 4.5:1 | 7:1 |
| **Large text** (≥18px / ≥14px bold) | 3:1 | 4.5:1 |
| **UI components** (borders, icons) | 3:1 | 3:1 |
| **Graphical objects** (charts, diagrams) | 3:1 | 3:1 |

### Contrast Ratio Calculation

The contrast ratio is calculated using relative luminance:

```
Contrast Ratio = (L1 + 0.05) / (L2 + 0.05)

Where L1 = lighter color's luminance
      L2 = darker color's luminance
      Luminance ranges from 0 (black) to 1 (white)
```

**Practical reference:**

| Ratio | Description |
|-------|-------------|
| 1:1 | Same color (no contrast) |
| 3:1 | Minimum for large text and UI components |
| 4.5:1 | Minimum for normal text (AA) |
| 7:1 | Enhanced contrast (AAA) |

## III. Common Contrast Failures

| Scenario | Typical Issue | Fix |
|----------|--------------|-----|
| Gray text on white | `#999` on `#fff` = 2.85:1 | Use `#767676` or darker (4.54:1) |
| White text on yellow | `#fff` on `#ff0` = 1.07:1 | Use dark text on yellow backgrounds |
| Placeholder text | Often too light | Ensure `#767676` or darker |
| Disabled buttons | Often low contrast (intentionally) | Still needs 3:1 for text |
| Link on colored background | Depends on exact colors | Test with contrast checker |

### Specific Bad Combinations

```
❌ #aaa on #fff  →  2.32:1  (fails AA)
❌ #ccc on #000  →  1.61:1  (fails AA)
❌ #666 on #999  →  2.42:1  (fails AA)
❌ #ff6600 on #fff → 2.83:1  (fails AA for normal text)

✅ #767676 on #fff  →  4.54:1  (passes AA)
✅ #595959 on #fff  →  7.00:1  (passes AAA)
✅ #fff on #0066cc  →  5.37:1  (passes AA)
```

## IV. Tools for Checking Contrast

| Tool | Type | Usage |
|------|------|-------|
| **Chrome DevTools** | Built-in | Inspect element → Computed → contrast ratio |
| **WebAIM Contrast Checker** | Web tool | [webaim.org/resources/contrastchecker](https://webaim.org/resources/contrastchecker/) |
| **axe DevTools** | Browser extension | Automatic contrast checking |
| **Color Safe** | Web tool | Generate accessible color palettes |
| **Stark** | Figma/Sketch plugin | Contrast checking in design tools |

### Chrome DevTools

1. Open DevTools
2. Inspect any text element
3. Click the color swatch
4. DevTools shows the contrast ratio and pass/fail status

## V. Designing with Contrast in Mind

### Semantic Colors

```css
/* Define accessible color tokens */
:root {
  /* Text colors (all pass AA on white) */
  --text-primary: #1a1a1a;      /* 16.1:1 on #fff */
  --text-secondary: #595959;    /* 7.0:1 on #fff */
  --text-muted: #767676;        /* 4.5:1 on #fff */

  /* Background colors */
  --bg-primary: #ffffff;
  --bg-secondary: #f5f5f5;
  --bg-dark: #1a1a1a;

  /* Interactive colors */
  --link: #0066cc;              /* 5.4:1 on #fff */
  --link-hover: #004499;        /* 8.6:1 on #fff */
  --focus-ring: #005fcc;        /* 5.9:1 on #fff */

  /* Semantic colors (verify against their backgrounds) */
  --success: #1a7a3a;           /* 4.5:1 on #fff */
  --warning: #8a6d00;           /* 4.5:1 on #fff */
  --error: #cc0000;             /* 5.9:1 on #fff */
}
```

### Dark Mode Contrast

Dark mode requires recalculating contrast ratios — colors that work on white may not work on dark backgrounds:

```css
/* Light mode */
:root {
  --text-primary: #1a1a1a;      /* 16.1:1 on #fff ✓ */
  --bg-primary: #ffffff;
}

/* Dark mode — recalculate */
:root[data-theme="dark"] {
  --text-primary: #e0e0e0;      /* 13.9:1 on #121212 ✓ */
  --bg-primary: #121212;
}
```

## VI. Color as the Only Indicator

Never use color as the **only** means of conveying information:

```html
<!-- ❌ BAD — color-only indicator -->
<span style="color: red;">Error</span>
<span style="color: green;">Success</span>

<!-- ✅ GOOD — color + text -->
<span class="error" role="alert">✕ Error: Invalid email</span>
<span class="success">✓ Success: Form submitted</span>

<!-- ✅ GOOD — color + icon -->
<span class="error"><svg aria-hidden="true">✕</svg> Error</span>
<span class="success"><svg aria-hidden="true">✓</svg> Success</span>
```

### Other Color-Only Patterns to Avoid

| Pattern | Problem | Fix |
|---------|---------|-----|
| Red/green for required/optional fields | Color blind users can't distinguish | Add asterisk or text label |
| Colored dots for status | Color only indicator | Add text label or icon |
| Red/green charts | Color blind users can't read | Use patterns + colors |

## VII. Testing Contrast Automatically

### CI Integration with axe-core

```typescript
import { AxePuppeteer } from '@axe-core/puppeteer'

test('page has no contrast violations', async () => {
  const results = await new AxePuppeteer(page)
    .withRules(['color-contrast'])
    .analyze()

  expect(results.violations).toEqual([])
})
```

### Design Token Validation

```typescript
// Test that all color tokens meet contrast requirements
import { getContrastRatio } from './color-utils'

describe('Color tokens', () => {
  it('primary text meets AA on white background', () => {
    const ratio = getContrastRatio('#1a1a1a', '#ffffff')
    expect(ratio).toBeGreaterThanOrEqual(4.5)
  })

  it('secondary text meets AA on white background', () => {
    const ratio = getContrastRatio('#595959', '#ffffff')
    expect(ratio).toBeGreaterThanOrEqual(4.5)
  })
})
```

## VIII. Key Takeaways

1. **4.5:1 minimum** for normal text; 3:1 for large text and UI components
2. **Test with tools** — Chrome DevTools, axe, WebAIM checker
3. **Never use color as the only indicator** — add text, icons, or patterns
4. **Recalculate for dark mode** — different backgrounds change contrast ratios
5. **Design tokens** should be validated for contrast in CI

## References

- [WCAG 1.4.3: Contrast (Minimum)](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [web.dev: Color and contrast](https://web.dev/learn/accessibility/color-contrast/)
