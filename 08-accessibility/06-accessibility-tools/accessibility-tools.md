# Accessibility Tools

## I. Tool Categories

Accessibility testing uses a spectrum of tools, from automated scanners to manual assistive technology testing. No single tool catches all issues — a combination is required.

```
Automated ←──────────────────────────────→ Manual
(Quick, broad)                            (Slow, deep)
~30% of issues                           ~70% of issues

axe DevTools, Lighthouse                 Screen readers, keyboard-only
pa11y, eslint-plugin-jsx-a11y           manual inspection
```

## II. Browser DevTools (Built-in)

### Chrome DevTools Accessibility Panel

1. Open DevTools → Elements → Accessibility tab
2. **Computed Properties** — shows role, name, value for any element
3. **Contrast** — shows contrast ratio for text elements
4. **Tree View** — shows the accessibility tree (what screen readers see)

### Firefox Accessibility Inspector

1. Open DevTools → Accessibility panel
2. Shows accessibility tree with roles, names, states
3. Highlights contrast issues
4. Shows landmarks and heading structure

## III. Automated Testing Tools

### axe-core (Deque)

The industry-standard automated accessibility engine. Open-source, embeddable in tests.

```typescript
// Jest + axe-core + jest-axe
import { axe, toHaveNoViolations } from 'jest-axe'

expect.extend(toHaveNoViolations)

test('login form has no accessibility violations', async () => {
  const { container } = render(<LoginForm />)
  const results = await axe(container)
  expect(results).toHaveNoViolations()
})
```

### Lighthouse (Chrome DevTools)

Built into Chrome DevTools. Provides an accessibility score with specific issue descriptions.

1. Open DevTools → Lighthouse tab
2. Check "Accessibility"
3. Click "Generate report"

**Limitation:** Only tests what's in the initial DOM; doesn't test dynamic interactions.

### axe DevTools Browser Extension

Real-time accessibility scanning while you browse:

1. Install the axe DevTools extension
2. Click the axe icon on any page
3. See violations with severity, impact, and fix suggestions

### pa11y

Command-line accessibility testing for CI/CD:

```bash
# Install
npm install -g pa11y

# Test a page
pa11y https://example.com

# Test with WCAG 2.1 AA standard
pa11y --standard WCAG2AA https://example.com

# Output as JSON for CI
pa11y --reporter json https://example.com > report.json
```

### ESLint Plugin (jsx-a11y)

Catches accessibility issues during development:

```json
{
  "plugins": ["jsx-a11y"],
  "extends": ["plugin:jsx-a11y/recommended"]
}
```

```bash
npm install --save-dev eslint-plugin-jsx-a11y
```

**Catches:** missing alt text, missing labels, invalid ARIA, non-interactive elements with click handlers.

## IV. Manual Testing Tools

### Screen Readers

| Screen Reader | Browser | Platform | Cost |
|---------------|---------|----------|------|
| **NVDA** | Firefox, Chrome | Windows | Free |
| **JAWS** | Chrome, Firefox | Windows | Paid |
| **VoiceOver** | Safari | macOS, iOS | Free (built-in) |
| **TalkBack** | Chrome | Android | Free (built-in) |

### Keyboard-Only Testing

1. Unplug your mouse
2. Use Tab/Shift+Tab to navigate
3. Use Enter/Space to activate
4. Use Escape to close
5. Use Arrow keys in composite widgets
6. Verify all functionality is accessible

### Color Contrast Checkers

| Tool | Type |
|------|------|
| **WebAIM Contrast Checker** | Web tool |
| **Stark** | Figma/Sketch plugin |
| **Colour Contrast Analyser** | Desktop app (TPGi) |
| **Chrome DevTools** | Built-in |

## V. Testing in CI/CD

### GitHub Actions with pa11y

```yaml
name: Accessibility
on: [pull_request]

jobs:
  a11y:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build && npm run preview &

      - name: Install pa11y
        run: npm install -g pa11y-ci

      - name: Run accessibility tests
        run: |
          npx pa11y-ci --config .pa11yci.json
```

### Jest with jest-axe

```typescript
// setupTests.ts
import { toHaveNoViolations } from 'jest-axe'
expect.extend(toHaveNoViolations)

// component.test.tsx
import { axe } from 'jest-axe'

test('has no accessibility violations', async () => {
  const { container } = render(<MyComponent />)
  const results = await axe(container)
  expect(results).toHaveNoViolations()
})
```

### Playwright with axe

```typescript
import AxeBuilder from '@axe-core/playwright'

test('page has no accessibility violations', async ({ page }) => {
  await page.goto('/')

  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa'])
    .analyze()

  expect(results.violations).toEqual([])
})
```

## VI. Tool Selection Guide

| Need | Tool |
|------|------|
| Quick scan of any page | Lighthouse or axe extension |
| CI/CD automated checks | pa11y-ci or jest-axe |
| Component-level testing | jest-axe or @axe-core/playwright |
| Development-time linting | eslint-plugin-jsx-a11y |
| Manual screen reader testing | NVDA (Windows) or VoiceOver (macOS) |
| Keyboard-only testing | Unplug your mouse |
| Contrast checking | WebAIM or Chrome DevTools |
| Design review | Stark (Figma plugin) |

## VII. Key Takeaways

1. **No single tool catches everything** — combine automated + manual testing
2. **Automated tools catch ~30%** — screen reader and keyboard testing catch the rest
3. **CI integration** — add axe/pa11y to your test suite to prevent regressions
4. **Linting catches issues early** — eslint-plugin-jsx-a11y flags problems during development
5. **Test with real assistive technology** — at least quarterly, use NVDA or VoiceOver

## References

- [axe-core](https://github.com/dequelabs/axe-core)
- [pa11y](https://github.com/pa11y/pa11y)
- [jest-axe](https://github.com/nickcolley/jest-axe)
- [NVDA Screen Reader](https://www.nvaccess.org/)
