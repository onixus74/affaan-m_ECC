---
description: Fix SvelteKit and Vite build errors, TypeScript issues in .svelte files, and adapter configuration problems. Invokes the svelte-build-resolver agent.
---

# Svelte Build and Fix

This command invokes the **svelte-build-resolver** agent to incrementally fix Svelte/SvelteKit build errors with minimal changes.

## What This Command Does

1. **Run Diagnostics**: Execute `npm run build`, `svelte-check`, `tsc --noEmit`
2. **Parse Errors**: Group by file and sort by severity
3. **Fix Incrementally**: One error at a time
4. **Verify Each Fix**: Re-run build after each change
5. **Report Summary**: Show what was fixed and what remains

## When to Use

Use `/svelte-build` when:
- `npm run build` fails with errors
- `svelte-check` reports TypeScript issues
- Vite bundler errors occur
- SvelteKit adapter configuration problems
- CSS scoping issues in components

## Diagnostic Commands Run

```bash
# Primary build check
npm run build

# Svelte type checking
svelte-check --tsconfig ./tsconfig.json

# TypeScript check
npx tsc --noEmit

# Lint check
npm run lint
```

## Example Session

```text
User: /svelte-build

Agent:
# Svelte Build Resolution

## Initial Diagnostics

```bash
$ npm run build

src/routes/products/+page.svelte:5:16
Error: Cannot use $state outside of .svelte or .svelte.ts files

src/routes/products/+page.server.ts:12:5
Error: Type 'null' is not assignable to type 'PageLoad'

src/lib/components/Card.svelte:8:3
Error: Cannot find module '$lib/utils'
```

Errors found: 3

## Fix 1: Runes Outside .svelte

File: src/routes/products/+page.svelte:5
Error: Cannot use $state outside of .svelte files
Cause: File has wrong extension

```typescript
// Moved reactive state to .svelte.ts
// src/lib/state/products.svelte.ts
export class ProductState {
  items = $state<Product[]>([]);
}
```

```bash
$ npm run build
# 2 errors remaining
```

## Fix 2: Type Error in Server Load

File: src/routes/products/+page.server.ts:12
Error: Type 'null' not assignable to 'PageLoad'

```typescript
// Added null check
const product = await getProduct(params.id);
if (!product) {
  throw error(404, 'Not found');
}
```

```bash
$ npm run build
# 1 error remaining
```

## Fix 3: Missing Module Alias

File: src/lib/components/Card.svelte:8
Error: Cannot find module '$lib/utils'

```typescript
// Fixed import path
import { format } from '$lib/utils';
// Was: import { format } from 'lib/utils'; // Missing $
```

```bash
$ npm run build
# Build successful!
```

## Final Verification

```bash
$ svelte-check --tsconfig ./tsconfig.json
# No errors

$ npx vitest run
# All 32 tests passed
```

## Summary

| Metric | Count |
|--------|-------|
| Build errors fixed | 3 |
| Files modified | 3 |
| Remaining issues | 0 |

Build Status: SUCCESS
```

## Common Errors Fixed

| Error | Typical Fix |
|-------|-------------|
| `Cannot use $state outside .svelte` | Move to `.svelte.ts` or use stores |
| `Cannot find module '$lib'` | Fix path alias in svelte.config.js and tsconfig |
| `Hydration mismatch` | Ensure server/client render same content |
| `Type 'X' not assignable` | Fix type annotation |
| `Unexpected token` in `.svelte` | Add `lang="ts"` to script tag |
| `CSS :global missing` | Add `:global()` wrapper or move styles |
| `Adapter error` | Install and configure correct adapter |

## Fix Strategy

1. **Build errors first** - Code must compile
2. **Type errors second** - Fix TypeScript issues
3. **Lint errors third** - Style and best practices
4. **One fix at a time** - Verify each change
5. **Minimal changes** - Don't refactor, just fix

## Stop Conditions

The agent will stop and report if:
- Same error persists after 3 attempts
- Fix introduces more errors
- Requires architectural changes
- Adapter needs platform-specific configuration

## Related

- Agent: `agents/svelte-build-resolver.md`
- Skills: `skills/svelte-patterns/`
- Commands: `/svelte-test`, `/svelte-review`
