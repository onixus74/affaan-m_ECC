---
description: Svelte TDD workflow with Vitest and Testing Library
agent: tdd-guide
subtask: true
---

# Svelte Test Command

Implement using Svelte TDD methodology with Vitest: $ARGUMENTS

## Your Task

Apply test-driven development with Svelte best practices:

1. **Define component** - Props, events, slots
2. **Write Vitest tests** - Using @testing-library/svelte
3. **Implement minimal code** - Pass the tests
4. **Verify** - Run full suite

## TDD Cycle for Svelte

### Step 1: Define Component Interface
```svelte
<script lang="ts">
  import type { Snippet } from 'svelte';

  interface Props {
    count?: number;
    onchange?: (value: number) => void;
    children: Snippet;
  }

  let { count = 0, onchange, children }: Props = $props();
</script>
```

### Step 2: Write Tests
```typescript
import { render, fireEvent } from '@testing-library/svelte';
import { describe, it, expect } from 'vitest';
import Counter from './Counter.svelte';

describe('Counter', () => {
  it('renders initial count', () => {
    const { getByText } = render(Counter, { props: { count: 5 } });
    expect(getByText('5')).toBeInTheDocument();
  });

  it('increments on click', async () => {
    const { getByRole, getByText } = render(Counter);
    await fireEvent.click(getByRole('button'));
    expect(getByText('1')).toBeInTheDocument();
  });
});
```

### Step 3: Run Tests (RED)
```bash
npm run test
```

### Step 4: Implement (GREEN)
```svelte
<script lang="ts">
  let count = $state(0);
</script>

<button onclick={() => count++}>
  {count}
</button>
```

## Svelte Testing Commands

```bash
# Run all tests
npm run test

# Run in watch mode
npm run test -- --watch

# Run with coverage
npm run test -- --coverage

# Run specific test file
npm run test -- Counter.test.ts
```

## Test File Organization

```
src/
├── lib/
│   ├── components/
│   │   ├── Counter/
│   │   │   ├── Counter.svelte
│   │   │   └── Counter.test.ts
│   │   └── Button/
│   │       ├── Button.svelte
│   │       └── Button.test.ts
│   └── utils/
│       ├── format.ts
│       └── format.test.ts
└── routes/
    └── +page.svelte
```

---

**TIP**: Use `@testing-library/svelte` for user-centric testing. Avoid testing implementation details.
