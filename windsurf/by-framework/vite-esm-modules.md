---
id: rule.vite-esm
type: rule
title: ES Module Syntax Consistency in Vite Projects
tags: [vite, esm, modules, javascript, configuration, build-errors]
summary: Enforce ES Module syntax in all config files when package.json specifies "type:module" - prevents "module is not defined" and other critical build failures in Vite projects.
version: 1
---

# ES Module Syntax Consistency in Vite Projects

**Critical Rule: When `package.json` contains `"type": "module"`, ALL JavaScript config files MUST use ES Module syntax.**

---

## 🎯 The Problem

**Common Error Message:**
```
ReferenceError: module is not defined in ES module scope
This file is being treated as an ES module because it has a '.js' file extension 
and the nearest package.json contains "type": "module"
```

**What Happened:**
- Your `package.json` has `"type": "module"` (required for Vite)
- But your config files still use CommonJS syntax (`module.exports`)
- Node treats `.js` files as ES modules by default when `type: "module"` is set
- Using CommonJS syntax in an ES module context causes runtime errors

**Impact:** Complete build failure - dev server won't start, production builds fail

---

## ✅ The Rule

### Core Requirement

**ALL these config files MUST use ES Module syntax:**

1. ✅ `vite.config.js` or `vite.config.ts`
2. ✅ `postcss.config.js`
3. ✅ `tailwind.config.js`
4. ✅ `vitest.config.js`
5. ✅ `playwright.config.js`
6. ✅ Any custom build scripts (`*.js` files)

### Correct Syntax (ES Modules)

```javascript
// ✅ CORRECT - ES Module syntax
export default {
  // ... your config
}

// ✅ CORRECT - Named exports
export const config = {
  // ... your config
}

// ✅ CORRECT - Multiple exports
export default defineConfig({
  // ... your config
})

export const someHelper = () => {}
```

### Incorrect Syntax (CommonJS)

```javascript
// ❌ WRONG - CommonJS syntax (causes error)
module.exports = {
  // ... your config
}

// ❌ WRONG - CommonJS require
const plugin = require('some-plugin')

// ❌ WRONG - CommonJS exports
exports.config = {
  // ... your config
}
```

---

## 🔧 How to Fix

### Option 1: Convert to ES Module Syntax (Recommended)

**Find and replace in each config file:**

```bash
# Automated fix - converts module.exports to export default
sed -i 's/module\.exports =/export default/g' vite.config.js
sed -i 's/module\.exports =/export default/g' postcss.config.js
sed -i 's/module\.exports =/export default/g' tailwind.config.js

# Or manually edit each file
```

**Manual conversion example:**

**Before (CommonJS):**
```javascript
// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

**After (ES Module):**
```javascript
// postcss.config.js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

### Option 2: Rename to `.cjs` Extension

**If you want to keep CommonJS syntax:**

```bash
# Rename files to .cjs
mv postcss.config.js postcss.config.cjs
mv tailwind.config.js tailwind.config.cjs

# Keep CommonJS syntax in .cjs files
# Vite will still find and use them
```

**Note:** Most tools auto-detect `.cjs` files. Vite, PostCSS, and Tailwind all support this.

### Option 3: Remove `"type": "module"` (Not Recommended)

```json
// package.json
{
  "name": "my-app",
  // Remove or comment out:
  // "type": "module"
}
```

**⚠️ Warning:** This breaks Vite's expectations and may cause other issues. Only use if you have a specific reason to avoid ES modules.

---

## 🔍 All Affected Config Files

### 1. Vite Configuration

**File:** `vite.config.js` or `vite.config.ts`

**✅ Correct:**
```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000
  }
})
```

**❌ Wrong:**
```javascript
const { defineConfig } = require('vite')
const react = require('@vitejs/plugin-react')

module.exports = defineConfig({
  plugins: [react()],
})
```

### 2. PostCSS Configuration

**File:** `postcss.config.js`

**✅ Correct:**
```javascript
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

**❌ Wrong:**
```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

### 3. Tailwind Configuration

**File:** `tailwind.config.js`

**✅ Correct:**
```javascript
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

**❌ Wrong:**
```javascript
module.exports = {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

### 4. Vitest Configuration

**File:** `vitest.config.js`

**✅ Correct:**
```javascript
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
  },
})
```

### 5. Playwright Configuration

**File:** `playwright.config.js`

**✅ Correct:**
```javascript
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests',
  use: {
    baseURL: 'http://localhost:5173',
  },
})
```

### 6. Custom Build Scripts

**Any `.js` files you create:**

**✅ Correct:**
```javascript
// scripts/build-icons.js
import fs from 'fs'
import path from 'path'

export function buildIcons() {
  // ... your logic
}

buildIcons()
```

**❌ Wrong:**
```javascript
// scripts/build-icons.js
const fs = require('fs')
const path = require('path')

function buildIcons() {
  // ... your logic
}

buildIcons()
```

---

## 🚨 Common Error Scenarios

### Error 1: "module is not defined"

**Full Error:**
```
ReferenceError: module is not defined in ES module scope
This file is being treated as an ES module because it has a '.js' file extension
and the nearest package.json contains "type": "module". To treat it as a 
CommonJS script, rename it to use the '.cjs' extension.
```

**Cause:** Using `module.exports` in a `.js` file when `"type": "module"` is set

**Solution:**
```bash
# Option A: Convert to ESM
sed -i 's/module\.exports =/export default/g' <file>.js

# Option B: Rename to .cjs
mv <file>.js <file>.cjs
```

### Error 2: "require is not defined"

**Full Error:**
```
ReferenceError: require is not defined in ES module scope, you can use import instead
This file is being treated as an ES module because it has a '.js' file extension
and the nearest package.json contains "type": "module".
```

**Cause:** Using `require()` in ES module

**Solution:**
```javascript
// ❌ Before
const react = require('@vitejs/plugin-react')

// ✅ After
import react from '@vitejs/plugin-react'
```

### Error 3: "Cannot use import statement outside a module"

**Full Error:**
```
SyntaxError: Cannot use import statement outside a module
```

**Cause:** Using `import` in CommonJS context (missing `"type": "module"`)

**Solution:**
```json
// Add to package.json
{
  "type": "module"
}
```

### Error 4: "__dirname is not defined"

**Full Error:**
```
ReferenceError: __dirname is not defined in ES module scope
```

**Cause:** ES modules don't have `__dirname` or `__filename`

**Solution:**
```javascript
// ❌ CommonJS way
const path = require('path')
const dir = __dirname

// ✅ ES module way
import { fileURLToPath } from 'url'
import { dirname } from 'path'

const __filename = fileURLToPath(import.meta.url)
const __dirname = dirname(__filename)
```

### Error 5: "exports is not defined"

**Full Error:**
```
ReferenceError: exports is not defined in ES module scope
```

**Cause:** Using `exports.something = ...` in ES module

**Solution:**
```javascript
// ❌ Before
exports.config = { ... }

// ✅ After
export const config = { ... }
```

---

## 📦 Package.json Configuration

### Correct Setup

```json
{
  "name": "my-vite-app",
  "type": "module",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "@vitejs/plugin-react": "^4.2.0"
  }
}
```

**Key points:**
- ✅ `"type": "module"` is present
- ✅ All config files use ES module syntax
- ✅ All custom scripts use `import`/`export`

---

## 🧪 TypeScript Considerations

### TypeScript Config Files

**TypeScript files (`.ts`) always use ES module syntax:**

```typescript
// vite.config.ts - Always correct syntax
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
})
```

**No issues with `.ts` files** - they're always treated as ES modules by default.

### tsconfig.json Settings

**For best compatibility:**

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "bundler",
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src"]
}
```

**Key settings:**
- `"module": "ESNext"` - Use latest ES module syntax
- `"moduleResolution": "bundler"` - Vite-compatible resolution
- `"esModuleInterop": true` - Better interop with CommonJS packages

---

## 🔄 Migration Checklist

**When converting an existing project to ES modules:**

### Step 1: Update package.json
- [ ] Add `"type": "module"` to `package.json`

### Step 2: Convert All Config Files
- [ ] Convert `vite.config.js` (or rename to `.ts`)
- [ ] Convert `postcss.config.js`
- [ ] Convert `tailwind.config.js`
- [ ] Convert `vitest.config.js` (if present)
- [ ] Convert `playwright.config.js` (if present)
- [ ] Convert any custom build scripts

### Step 3: Update Import Statements
- [ ] Replace all `require()` with `import`
- [ ] Replace all `module.exports` with `export default`
- [ ] Replace `exports.foo` with `export const foo`

### Step 4: Fix Node.js Globals (if used)
- [ ] Replace `__dirname` with ES module equivalent
- [ ] Replace `__filename` with ES module equivalent

### Step 5: Test Everything
- [ ] Run `npm run dev` - dev server starts
- [ ] Run `npm run build` - build succeeds
- [ ] Run `npm run preview` - preview works
- [ ] Run tests (if applicable)

### Step 6: Update Documentation
- [ ] Update README if it shows old syntax
- [ ] Update any setup guides

---

## 🧪 Verification Tests

### Test 1: Dev Server Starts

```bash
npm run dev
# Should start without errors
# No "module is not defined" errors
```

### Test 2: Build Succeeds

```bash
npm run build
# Should complete successfully
# Check dist/ folder created
```

### Test 3: All Configs Validated

```bash
# Check each config file syntax
node --check vite.config.js
node --check postcss.config.js
node --check tailwind.config.js
```

### Test 4: No CommonJS Syntax Remaining

```bash
# Search for CommonJS patterns
grep -r "module.exports" . --include="*.js" --exclude-dir=node_modules
grep -r "require(" . --include="*.js" --exclude-dir=node_modules
grep -r "exports\." . --include="*.js" --exclude-dir=node_modules

# Should return no results (or only .cjs files)
```

---

## 💡 Best Practices

### 1. Use TypeScript for Config Files

**Recommended approach:**

```bash
# Rename to .ts
mv vite.config.js vite.config.ts
mv vitest.config.js vitest.config.ts
```

**Benefits:**
- Type safety and autocomplete
- No ESM/CommonJS confusion
- Better IDE support

### 2. Be Consistent

**Pick one approach project-wide:**
- Either use `.js` with ES modules everywhere
- Or use `.cjs` for CommonJS files
- Don't mix unless necessary

### 3. Document Your Choice

**Add to README.md:**

```markdown
## Configuration Files

This project uses ES modules (`"type": "module"`). All JavaScript
config files use `export default` syntax. If you need CommonJS,
use `.cjs` extension.
```

### 4. Use Linting

**ESLint config:**

```javascript
// .eslintrc.cjs
module.exports = {
  env: {
    es2021: true,
    node: true,
  },
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module', // Enforce ES modules
  },
}
```

### 5. Template Starters

**When creating new projects:**

```bash
# Use official Vite templates (already correct)
npm create vite@latest my-app -- --template react
npm create vite@latest my-app -- --template react-ts

# Templates already have correct ESM setup
```

---

## 🚫 Anti-Patterns

### ❌ Don't: Mix ESM and CommonJS in Same File

```javascript
// ❌ BAD - mixing syntaxes
import React from 'react'
const plugin = require('some-plugin') // Don't mix!

module.exports = { ... } // Don't use with import!
```

### ❌ Don't: Use `require()` for Node Built-ins in ESM

```javascript
// ❌ BAD
const fs = require('fs')

// ✅ GOOD
import fs from 'fs'
```

### ❌ Don't: Forget to Update Dynamic Imports

```javascript
// ❌ BAD in ESM context
const module = require(`./modules/${name}`)

// ✅ GOOD
const module = await import(`./modules/${name}.js`)
```

### ❌ Don't: Use `.js` Extension in Imports (Usually)

```javascript
// In Vite projects, extensions are optional:

// ✅ GOOD - Vite resolves automatically
import Button from './components/Button'

// ⚠️ OK but unnecessary
import Button from './components/Button.jsx'
```

---

## 🔗 Monorepo Considerations

**In monorepo setups (packages/apps structure):**

### Each Package Needs Correct Configuration

```
monorepo/
├── package.json (workspace root)
├── apps/
│   ├── web/
│   │   ├── package.json ("type": "module")
│   │   ├── vite.config.js (ESM syntax)
│   │   └── postcss.config.js (ESM syntax)
│   └── mobile/
│       └── ... (might use different module system)
└── packages/
    ├── shared/
    │   ├── package.json (check "type" setting)
    │   └── ... (use consistent syntax)
```

**Key points:**
- Each `package.json` can have different `"type"` setting
- Config files must match their package's `"type"`
- Workspace root `"type"` doesn't affect sub-packages

---

## 📚 Related Arsenal Items

**🔧 Scripts:**
- [React Vite Setup](https://github.com/ChrisTansey007/ai-scripts-arsenal/tree/main/scripts/frontend/react-vite-setup) - Automated setup with ESM fixes

**💭 Prompts:**
- [Modernize React UI](https://github.com/ChrisTansey007/prompt-arsenal/blob/main/development/ui/modernize-react-ui.md) - UI enhancement guide

**🔗 Example:**
- [React Vite Setup Example](https://github.com/ChrisTansey007/arsenal-integration-hub/tree/main/examples/react-vite-setup) - Complete workflow demo

---

## 📖 Further Reading

**Official Documentation:**
- [Vite Configuration](https://vitejs.dev/config/)
- [Node.js ES Modules](https://nodejs.org/api/esm.html)
- [MDN: JavaScript Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)

**Common Tools Documentation:**
- [PostCSS Config](https://github.com/postcss/postcss#usage)
- [Tailwind Config](https://tailwindcss.com/docs/configuration)
- [Vitest Config](https://vitest.dev/config/)
- [Playwright Config](https://playwright.dev/docs/test-configuration)

---

## ✅ Quick Reference

**When you see this error:**
```
ReferenceError: module is not defined in ES module scope
```

**Do this:**
```bash
# Quick fix for all common config files
sed -i 's/module\.exports =/export default/g' vite.config.js
sed -i 's/module\.exports =/export default/g' postcss.config.js
sed -i 's/module\.exports =/export default/g' tailwind.config.js
```

**Then verify:**
```bash
npm run dev  # Should start without errors
```

---

**Result: Consistent ES module syntax, no build failures, Vite works perfectly!** ⚡
