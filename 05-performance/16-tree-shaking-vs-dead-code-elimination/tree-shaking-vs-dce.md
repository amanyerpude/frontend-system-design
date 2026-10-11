# Tree Shaking vs Dead Code Elimination

## Overview

**Tree shaking** and **dead code elimination (DCE)** are related but distinct techniques for reducing JavaScript bundle size. Tree shaking is a specific form of DCE that operates on ES modules. Understanding the difference helps configure build tools correctly and avoid shipping unnecessary code.

## Dead Code Elimination (DCE)

DCE is the **general process of removing code that is never executed** or whose results are never used. It's an optimization performed by compilers and minifiers.

### What DCE Removes

| Type | Example |
|------|---------|
| **Unreachable code** | Code after `return`, `throw`, `break` |
| **Unused variables** | Variables declared but never read |
| **Unused functions** | Functions declared but never called |
| **Unused imports** | Imported modules that are never referenced |
| **Constant-folded branches** | `if (false) { ... }` or `if (process.env.NODE_ENV === 'production') { devTools }` |
| **Side-effect-free unused exports** | Exports never imported by the application |

### DCE in Minifiers

Terser and other minifiers perform DCE as part of minification:

```javascript
// Before DCE
function unusedFunction() {
  return 'never called'
}

const unusedVariable = 42

function activeFunction() {
  return 'called'
}

// After DCE
function activeFunction() {
  return 'called'
}
```

### Constant Folding

DCE can evaluate constant expressions at build time:

```javascript
// Before
if (process.env.NODE_ENV === 'production') {
  console.log('prod')
} else {
  console.log('dev')
}

// After (with NODE_ENV=production)
console.log('prod')
```

This is why `process.env.NODE_ENV` checks work — the bundler replaces the constant and removes the dead branch.

## Tree Shaking

Tree shaking is a **specific form of DCE that operates on ES module imports and exports**. It uses static analysis to determine which exports from a module are actually used, then removes unused exports from the bundle.

### How Tree Shaking Works

```
1. Module Analysis
   - Parse ES module imports/exports
   - Build dependency graph

2. Usage Analysis
   - Track which exports are imported
   - Follow references through the code

3. Dead Code Elimination
   - Remove unused exports
   - Remove unused module-level code
   - Remove now-unused functions

4. Bundle Output
   - Include only code that is actually used
```

### ES Modules Enable Tree Shaking

Tree shaking works because ES modules have **static structure** — imports and exports are determined at parse time, not runtime:

```javascript
// ES Module — tree-shakable (static structure)
import { formatDate } from './utils'
export function formatDate(date) { /* ... */ }
export function formatCurrency(amount) { /* ... */ }

// CommonJS — NOT tree-shakable (dynamic structure)
const utils = require('./utils')  // Can't statically analyze
module.exports = { formatDate, formatCurrency }
```

### Tree Shaking Example

```typescript
// utils.ts — Module with multiple exports
export function formatDate(date: Date): string {
  return date.toLocaleDateString()
}

export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency', currency: 'USD'
  }).format(amount)
}

export function formatRelativeTime(date: Date): string {
  const rtf = new Intl.RelativeTimeFormat('en', { numeric: 'auto' })
  const diff = date.getTime() - Date.now()
  return rtf.format(Math.round(diff / 86400000), 'day')
}

// app.ts — Only imports formatDate
import { formatDate } from './utils'

console.log(formatDate(new Date()))

// After tree shaking:
// ✅ formatDate: included (used)
// ❌ formatCurrency: removed (unused)
// ❌ formatRelativeTime: removed (unused)
```

### Side Effects and Tree Shaking

Tree shaking **cannot remove code with side effects**. A side effect is any code that modifies state outside its local scope:

```typescript
// ✅ Tree-shakable — no side effects
export function pureFunction(x: number): number {
  return x * 2
}

// ❌ NOT tree-shakable — has side effects
console.log('Module loaded')                    // Global side effect
window.myGlobal = value                         // Global mutation
document.title = 'My App'                       // DOM mutation
registerPlugin(myPlugin)                        // Registration side effect
```

### Controlling Side Effects in `package.json`

Libraries can declare which files have no side effects:

```json
{
  "name": "my-library",
  "sideEffects": false  // All files are tree-shakable
}

// Or specify which files have side effects:
{
  "sideEffects": [
    "*.css",
    "*.scss",
    "src/setup.ts"
  ]
}
```

## Key Differences

| Aspect | Dead Code Elimination | Tree Shaking |
|--------|----------------------|--------------|
| **Scope** | All code in a file | ES module imports/exports |
| **When** | During minification (Terser) | During bundling (Rollup, webpack) |
| **Mechanism** | Removes unreachable/unused code within a file | Removes unused exports across module boundaries |
| **Module format** | Works with CommonJS and ES modules | ES modules only |
| **Granularity** | Per-file, within a file | Cross-module |
| **Enabled by** | Minifier settings | Bundler with ES module support |
| **Requires** | Nothing special | ES modules + `sideEffects: false` |

## Bundler Comparison

| Bundler | Tree Shaking Support | Notes |
|---------|---------------------|-------|
| **Rollup** | Excellent | First bundler to implement tree shaking; default behavior |
| **Vite** | Excellent | Uses Rollup in production; ESBuild for dev |
| **webpack** | Good (4+) | Requires `mode: production`; uses `sideEffects` in `package.json` |
| **esbuild** | Good | Fast; may not catch all tree-shakable code |
| **Parcel** | Good | Automatic; uses `sideEffects` detection |

### Rollup Configuration

```javascript
// rollup.config.js
export default {
  input: 'src/index.js',
  output: {
    file: 'dist/bundle.js',
    format: 'es',
  },
  // Tree shaking is enabled by default
  treeshake: {
    moduleSideEffects: false,
    propertyReadSideEffects: false,
    tryCatchDeoptimization: false,
  },
}
```

### Webpack Configuration

```javascript
// webpack.config.js
module.exports = {
  mode: 'production',  // Enables tree shaking
  optimization: {
    usedExports: true,     // Mark unused exports
    minimize: true,        // Enable Terser for DCE
    sideEffects: true,     // Respect sideEffects in package.json
  },
}
```

## What Tree Shaking Cannot Remove

| Category | Reason |
|----------|--------|
| **Code with side effects** | Modifies global state |
| **CommonJS imports** | Dynamic module structure |
| **Dynamic imports** | Can't statically analyze `import(variable)` |
| **Re-exported values used elsewhere** | Used by another module |
| **CSS imports in JS** | Side effect (adds stylesheet to page) |
| **Polyfills** | Often have side effects (modify global prototypes) |
| **Decorators (legacy)** | May have side effects |

## Practical Performance Impact

### Lodash Example

```javascript
// ❌ BAD — imports entire lodash (~70KB minified)
import _ from 'lodash'
_.debounce(fn, 300)

// ❌ BETTER — named import, but lodash is CommonJS
import { debounce } from 'lodash'

// ✅ BEST — ES module import, tree-shakable
import debounce from 'lodash-es/debounce'

// ✅ BEST — lodash-es, tree-shakable by bundler
import { debounce } from 'lodash-es'
```

### Bundle Size Comparison

```
Full lodash import:        70KB minified
Named lodash import:       70KB minified (no tree shaking)
lodash-es named import:    ~2KB minified (tree-shaken)
```

## Best Practices

1. **Use ES modules** — `import`/`export` instead of `require`/`module.exports`
2. **Mark side effects** — Use `sideEffects: false` in `package.json` for libraries
3. **Import selectively** — `import { fn } from 'lib'` instead of `import lib from 'lib'`
4. **Use tree-shakable alternatives** — `lodash-es` over `lodash`, `date-fns` over `moment`
5. **Check bundle size** — Use `webpack-bundle-analyzer` or `source-map-explorer` to verify tree shaking worked
6. **Avoid CommonJS** — In your own code, always use ES modules
7. **Be careful with CSS-in-JS** — CSS imports in JS are side effects

## References

- [Rollup: Tree Shaking](https://rollupjs.org/guide/en/#tree-shaking)
- [webpack: Tree Shaking](https://webpack.js.org/guides/tree-shaking/)
- [Web.dev: Reduce JavaScript payloads with tree shaking](https://web.dev/learn/performance/serialization/tree-shaking)
- [Lodash ES](https://github.com/lodash/lodash/wiki/FP-Guide)
