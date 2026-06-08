---
description: Svelte/SvelteKit code review for runes, reactivity, and SSR
agent: svelte-reviewer
subtask: true
---

# Svelte Review Command

Review Svelte/SvelteKit code for reactivity, SSR safety, and best practices: $ARGUMENTS

## Your Task

1. **Analyze Svelte code** for runes and reactivity patterns
2. **Check SSR boundaries** - server vs client code
3. **Review accessibility** - ARIA, keyboard navigation
4. **Verify performance** - lazy loading, code splitting

## Review Checklist

### Svelte 5 Runes
- [ ] `$state()` used correctly for reactive state
- [ ] `$derived()` for computed values
- [ ] `$effect()` cleanup functions present
- [ ] `$props()` properly typed

### Reactivity
- [ ] No stale reactive references
- [ ] Reactive statements trigger correctly
- [ ] Stores used appropriately
- [ ] Bindings are two-way where needed

### SSR / SvelteKit
- [ ] Browser-only code guarded (`browser` from `$app/environment`)
- [ ] `onMount` used for client-only logic
- [ ] Load functions typed correctly
- [ ] Form actions validated server-side

### Accessibility
- [ ] Semantic HTML elements used
- [ ] ARIA attributes correct
- [ ] Keyboard navigation works
- [ ] Focus management in modals

## Report Format

### Reactivity Issues
- [file:line] Issue description
  Suggestion: How to fix

### SSR Issues
- [file:line] Issue description
  Suggestion: How to fix

### Accessibility Issues
- [file:line] Issue description
  Suggestion: How to fix

### Performance Issues
- [file:line] Issue description
  Suggestion: How to fix

---

**TIP**: Run `svelte-check --tsconfig ./tsconfig.json` for type checking.
