---
description: Fix SvelteKit/Vite build and compilation errors
agent: svelte-build-resolver
subtask: true
---

# Svelte Build Command

Fix SvelteKit and Vite build errors: $ARGUMENTS

## Your Task

1. **Run build**: `npm run build` or `pnpm build`
2. **Run svelte-check**: `svelte-check --tsconfig ./tsconfig.json`
3. **Fix errors** one by one
4. **Verify fixes** don't introduce new errors

## Common Svelte Errors

### TypeScript Errors
```
Type 'X' is not assignable to type 'Y'
```
**Fix**: Add proper types or fix the type definition

### Svelte Compilation Errors
```
Expected a valid expression
```
**Fix**: Check syntax in template, runes usage

### Import Errors
```
Cannot find module '$lib/...'
```
**Fix**: Check path aliases in `svelte.config.js`

### SSR Errors
```
document is not defined
```
**Fix**: Guard with `browser` check or move to `onMount`

## Fix Order

1. **Import/module errors** - Fix paths and aliases
2. **TypeScript errors** - Fix type definitions
3. **Svelte compilation** - Fix template/runes syntax
4. **SSR errors** - Guard browser-only code

## Build Commands

```bash
# Build for production
npm run build

# Type check
svelte-check --tsconfig ./tsconfig.json

# Dev server
npm run dev

# Preview production build
npm run preview

# Lint
npm run lint
```

## Verification

After fixes:
```bash
npm run build    # Should succeed
npm run lint     # Should pass
npm run test     # Tests should pass
```

---

**IMPORTANT**: Fix errors only. No refactoring, no improvements. Get the build green with minimal changes.
