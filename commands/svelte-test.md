---
description: Enforce TDD workflow for Svelte. Write Vitest tests first, then implement components. Verify 80%+ coverage.
---

# Svelte TDD Command

This command enforces test-driven development methodology for Svelte components using Vitest and Testing Library.

## What This Command Does

1. **Define Component Props**: Scaffold component with typed props first
2. **Write Component Tests**: Create test cases using @testing-library/svelte (RED)
3. **Run Tests**: Verify tests fail for the right reason
4. **Implement Component**: Write minimal code to pass (GREEN)
5. **Refactor**: Improve while keeping tests green
6. **Check Coverage**: Ensure 80%+ coverage

## When to Use

Use `/svelte-test` when:
- Creating new Svelte components
- Adding test coverage to existing components
- Fixing bugs in components (write failing test first)
- Building complex interactive components
- Learning TDD in Svelte

## TDD Cycle

```
RED     → Write failing component test
GREEN   → Implement minimal component
REFACTOR → Improve component, tests stay green
REPEAT  → Next test case
```

## Example Session

````markdown
User: /svelte-test I need a SearchInput component with debounced onChange

Agent:
# TDD Session: SearchInput Component

## Step 1: Define Component Props

```svelte
<!-- src/lib/components/SearchInput.svelte -->
<script lang="ts">
  interface Props {
    placeholder?: string;
    debounceMs?: number;
    onSearch?: (query: string) => void;
  }

  let {
    placeholder = 'Search...',
    debounceMs = 300,
    onSearch,
  }: Props = $props();

  // Not implemented yet
</script>

<input {placeholder} />
```

## Step 2: Write Tests (RED)

```typescript
// src/lib/components/SearchInput.test.ts
import { render, fireEvent } from '@testing-library/svelte';
import { vi } from 'vitest';
import SearchInput from './SearchInput.svelte';

describe('SearchInput', () => {
  it('renders with placeholder text', () => {
    const { getByPlaceholderText } = render(SearchInput, {
      props: { placeholder: 'Find products...' },
    });
    expect(getByPlaceholderText('Find products...')).toBeTruthy();
  });

  it('calls onSearch after debounce', async () => {
    vi.useFakeTimers();
    const onSearch = vi.fn();

    const { getByPlaceholderText } = render(SearchInput, {
      props: { onSearch, debounceMs: 300 },
    });

    const input = getByPlaceholderText('Search...');
    await fireEvent.input(input, { target: { value: 'shoes' } });

    expect(onSearch).not.toHaveBeenCalled();

    vi.advanceTimersByTime(300);
    expect(onSearch).toHaveBeenCalledWith('shoes');

    vi.useRealTimers();
  });

  it('debounces rapid input', async () => {
    vi.useFakeTimers();
    const onSearch = vi.fn();

    const { getByPlaceholderText } = render(SearchInput, {
      props: { onSearch, debounceMs: 300 },
    });

    const input = getByPlaceholderText('Search...');
    await fireEvent.input(input, { target: { value: 'sh' } });
    await fireEvent.input(input, { target: { value: 'shoe' } });
    await fireEvent.input(input, { target: { value: 'shoes' } });

    vi.advanceTimersByTime(300);
    expect(onSearch).toHaveBeenCalledTimes(1);
    expect(onSearch).toHaveBeenCalledWith('shoes');

    vi.useRealTimers();
  });
});
```

## Step 3: Run Tests - Verify FAIL

```bash
$ npx vitest run SearchInput

FAIL  src/lib/components/SearchInput.test.ts
  ✕ calls onSearch after debounce (expected onSearch to be called)
```

✓ Tests fail as expected.

## Step 4: Implement Minimal Code (GREEN)

```svelte
<!-- src/lib/components/SearchInput.svelte -->
<script lang="ts">
  interface Props {
    placeholder?: string;
    debounceMs?: number;
    onSearch?: (query: string) => void;
  }

  let {
    placeholder = 'Search...',
    debounceMs = 300,
    onSearch,
  }: Props = $props();

  let value = $state('');
  let timer: ReturnType<typeof setTimeout>;

  function handleInput(event: Event) {
    const target = event.target as HTMLInputElement;
    value = target.value;
    clearTimeout(timer);
    timer = setTimeout(() => {
      onSearch?.(value);
    }, debounceMs);
  }
</script>

<input {placeholder} {value} oninput={handleInput} />
```

## Step 5: Run Tests - Verify PASS

```bash
$ npx vitest run SearchInput

PASS  src/lib/components/SearchInput.test.ts
  ✓ renders with placeholder text
  ✓ calls onSearch after debounce
  ✓ debounces rapid input

Tests: 3 passed
```

✓ All tests passing!

## Step 6: Check Coverage

```bash
$ npx vitest run --coverage

% Coverage report
  SearchInput.svelte   | 100 | 100 | 100 | 100
```

✓ Coverage: 100%

## TDD Complete!
````

## Test Patterns

### Component Rendering
```typescript
import { render } from '@testing-library/svelte';

const { getByText, getByRole, container } = render(Component, {
  props: { title: 'Hello' },
});
```

### User Interactions
```typescript
import { fireEvent } from '@testing-library/svelte';

await fireEvent.click(getByRole('button'));
await fireEvent.input(input, { target: { value: 'test' } });
```

### Testing Runes
```typescript
// For reactive state in .svelte.ts files
import { get } from 'svelte/store';
// Test class-based state modules directly
const counter = new Counter();
counter.increment();
expect(counter.count).toBe(1);
```

### Snapshot Testing
```typescript
expect(container).toMatchSnapshot();
```

## Coverage Commands

```bash
npx vitest run --coverage
npx vitest run --coverage --reporter=verbose
```

## Coverage Targets

| Code Type | Target |
|-----------|--------|
| Utility functions | 90%+ |
| Custom hooks/stores | 85%+ |
| Presentational components | 80%+ |
| Page routes | Via E2E tests |

## TDD Best Practices

**DO:**
- Write test FIRST, before implementation
- Use `getByRole` and `getByText` over CSS selectors
- Use `data-testid` as last resort
- Test user-visible behavior, not implementation
- Use `vi.useFakeTimers()` for debounce/timeout tests

**DON'T:**
- Test internal state directly
- Use `container.querySelector` for simple cases
- Snapshot test complex components (brittle)
- Mock Svelte internals
- Skip `act()` warnings

## Related

- Agent: `agents/svelte-reviewer.md`
- Skills: `skills/svelte-patterns/`, `skills/tdd-workflow/`
- Commands: `/svelte-review`, `/svelte-build`
