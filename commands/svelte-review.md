---
description: Comprehensive Svelte and SvelteKit code review for runes correctness, SSR boundaries, accessibility, and security. Invokes the svelte-reviewer agent.
---

# Svelte Code Review

This command invokes the **svelte-reviewer** agent for comprehensive Svelte/SvelteKit-specific code review.

## What This Command Does

1. **Identify Svelte Changes**: Find modified `.svelte` and route files via `git diff`
2. **Run Svelte Check**: Execute `svelte-check --tsconfig ./tsconfig.json`
3. **Security Scan**: Check for @html XSS, secrets in client files, CSRF issues
4. **Reactivity Review**: Verify $state/$derived/$effect usage is correct
5. **SSR Review**: Analyze server/client boundary correctness
6. **Generate Report**: Categorize issues by severity

## When to Use

Use `/svelte-review` when:
- After writing or modifying Svelte components
- Before committing Svelte/SvelteKit changes
- Reviewing pull requests with Svelte code
- Onboarding to a new SvelteKit project
- Learning Svelte 5 runes patterns

## Review Categories

### CRITICAL (Must Fix)
- `@html` with untrusted user input (XSS)
- Secrets in `+page.ts` or other client-loadable files
- Non-reactive state mutations (won't update UI)
- `$effect` without cleanup (memory leaks)
- Missing CSRF protection on form actions

### HIGH (Should Fix)
- Browser-only APIs at module scope (SSR crash)
- `+page.ts` doing server-only work (database, secrets)
- Hydration mismatch (server/client render difference)
- Missing ARIA attributes on interactive elements
- Missing keyboard event handlers
- `$effect` used for derived data (use `$derived`)

### MEDIUM (Consider)
- Prop drilling instead of context
- Oversized components handling too many concerns
- Missing TypeScript on props
- Wrong file placement (+page.ts vs +page.server.ts)
- Unnecessary client JS on static pages

## Automated Checks Run

```bash
# Type checking
svelte-check --tsconfig ./tsconfig.json

# Linting (if configured)
npx eslint --plugin svelte src/

# Unit tests
npx vitest run

# E2E tests (if configured)
npx playwright test
```

## Example Usage

```text
User: /svelte-review

Agent:
# Svelte Code Review Report

## Files Reviewed
- src/routes/products/+page.svelte (modified)
- src/routes/products/+page.server.ts (modified)
- src/lib/components/ProductCard.svelte (modified)

## Static Analysis Results
✓ svelte-check: No errors
✓ eslint: 1 warning

## Issues Found

[CRITICAL] @html XSS Vulnerability
File: src/lib/components/ProductCard.svelte:12
Issue: User-provided content rendered with @html
```svelte
{@html product.description}
```
Fix: Use auto-escaped interpolation or sanitize
```svelte
{product.description}
```

[HIGH] $effect Used for Derived Data
File: src/routes/products/+page.svelte:8
Issue: $effect used to compute filtered list
```svelte
$effect(() => {
  filtered = items.filter(i => i.active);
});
```
Fix: Use $derived
```svelte
let filtered = $derived(items.filter(i => i.active));
```

## Summary
- CRITICAL: 1
- HIGH: 1
- MEDIUM: 0

Recommendation: FAIL: Block merge until CRITICAL issue is fixed
```

## Approval Criteria

| Status | Condition |
|--------|-----------|
| Approve | No CRITICAL or HIGH issues |
| Warning | Only MEDIUM issues (merge with caution) |
| Block | CRITICAL or HIGH issues found |

## Related

- Agent: `agents/svelte-reviewer.md`
- Skills: `skills/svelte-patterns/`
- Commands: `/svelte-test`, `/svelte-build`
