---
name: svelte-patterns
description: Svelte 5 and SvelteKit patterns covering runes reactivity, component composition, SSR, routing, form actions, and accessibility for building modern reactive UIs.
origin: ECC
---

# Svelte 5 & SvelteKit Development Patterns

Modern Svelte 5 patterns using runes reactivity, SvelteKit routing, and best practices for building performant reactive applications.

## When to Activate

- Writing Svelte 5 components with runes ($state, $derived, $effect, $props)
- Building SvelteKit routes, layouts, or server-side logic
- Managing reactivity and state in Svelte applications
- Implementing SSR or CSR boundaries in SvelteKit
- Writing Svelte component tests with Vitest or Playwright
- Reviewing Svelte code for idiomatic patterns and a11y

## Runes Reactivity Model (Svelte 5)

### $state — Reactive State

```svelte
<script lang="ts">
  let count = $state(0);
  let user = $state<{ name: string; email: string } | null>(null);

  interface Task {
    id: string;
    title: string;
    done: boolean;
  }

  let tasks = $state<Task[]>([]);
</script>
```

### $derived — Computed Values

```svelte
<script lang="ts">
  let items = $state<Item[]>([]);
  let query = $state('');

  let filtered = $derived(
    items.filter(item =>
      item.name.toLowerCase().includes(query.toLowerCase())
    )
  );

  let total = $derived(
    items.reduce((sum, item) => sum + item.price, 0)
  );
</script>
```

### $effect — Side Effects

```svelte
<script lang="ts">
  let searchQuery = $state('');
  let results = $state<Result[]>([]);

  $effect(() => {
    const controller = new AbortController();

    if (searchQuery.length > 2) {
      fetch(`/api/search?q=${searchQuery}`, { signal: controller.signal })
        .then(r => r.json())
        .then(data => { results = data.items; });
    }

    return () => controller.abort();
  });
</script>
```

### $props — Component Props

```svelte
<script lang="ts">
  interface ButtonProps {
    variant?: 'primary' | 'secondary';
    disabled?: boolean;
    onclick?: () => void;
    children?: import('svelte').Snippet;
  }

  let {
    variant = 'primary',
    disabled = false,
    onclick,
    children
  }: ButtonProps = $props();
</script>

<button {onclick} {disabled} class="btn btn-{variant}">
  {#if children}
    {@render children()}
  {/if}
</button>
```

### $bindable — Two-Way Binding Props

```svelte
<script lang="ts">
  let value = $bindable('');
  let label = $props().label;
</script>

<label>
  {label}
  <input bind:value />
</label>
```

## Component Patterns

### Snippets for Composition

```svelte
<!-- Card.svelte -->
<script lang="ts">
  import type { Snippet } from 'svelte';

  interface Props {
    heading?: Snippet;
    children: Snippet;
    footer?: Snippet;
  }

  let { heading, children, footer }: Props = $props();
</script>

<div class="card">
  {#if heading}
    <div class="card-header">
      {@render heading()}
    </div>
  {/if}
  <div class="card-body">
    {@render children()}
  </div>
  {#if footer}
    <div class="card-footer">
      {@render footer()}
    </div>
  {/if}
</div>
```

```svelte
<!-- Usage -->
<Card>
  {#snippet heading()}
    <h2>Dashboard</h2>
  {/snippet}
  <p>Main content here</p>
</Card>
```

### Event Handler Pattern

```svelte
<script lang="ts">
  function createFilter(initial: string) {
    let query = $state(initial);
    let debounced = $state(initial);
    let timer: ReturnType<typeof setTimeout>;

    $effect(() => {
      clearTimeout(timer);
      timer = setTimeout(() => { debounced = query; }, 300);
      return () => clearTimeout(timer);
    });

    return {
      get query() { return query; },
      set query(v: string) { query = v; },
      get debounced() { return debounced; },
    };
  }

  let filter = createFilter('');
</script>

<input bind:value={filter.query} placeholder="Search..." />
```

### Store Pattern (Legacy — Prefer Runes)

```svelte
<!-- Use runes instead. This is only for migration reference. -->
<script lang="ts">
  // Legacy Svelte 3/4 store pattern
  import { writable, derived } from 'svelte/store';

  const count = writable(0);
  const doubled = derived(count, $count => $count * 2);

  // Migration: Replace with runes
  let count = $state(0);
  let doubled = $derived(count * 2);
</script>
```

## SvelteKit Routing

### Route Structure

```text
src/routes/
├── +layout.svelte          # Root layout
├── +layout.ts              # Root layout load
├── +page.svelte            # Home page
├── about/
│   └── +page.svelte
├── products/
│   ├── +page.svelte        # Product list
│   ├── +page.ts            # Client-side data load
│   ├── [id]/
│   │   ├── +page.svelte    # Product detail
│   │   └── +page.server.ts # Server-side data load
│   └── new/
│       ├── +page.svelte
│       └── +page.server.ts # Form action
└── api/
    └── health/
        └── +server.ts      # API endpoint
```

### Server Data Loading

```typescript
// src/routes/products/[id]/+page.server.ts
import { error } from '@sveltejs/kit';
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ params, depends }) => {
  depends('product:detail');

  const product = await getProduct(params.id!);

  if (!product) {
    throw error(404, 'Product not found');
  }

  return { product };
};
```

### Form Actions

```typescript
// src/routes/products/new/+page.server.ts
import { fail, redirect } from '@sveltejs/kit';
import type { Actions } from './$types';

export const actions: Actions = {
  create: async ({ request }) => {
    const data = await request.formData();

    const name = data.get('name') as string;
    const price = Number(data.get('price'));

    if (!name || name.length < 1) {
      return fail(400, { name, error: 'Name is required' });
    }

    if (isNaN(price) || price <= 0) {
      return fail(400, { price, error: 'Price must be positive' });
    }

    const product = await createProduct({ name, price });

    throw redirect(303, `/products/${product.id}`);
  }
};
```

### Layout with Shared Data

```typescript
// src/routes/+layout.server.ts
import type { LayoutServerLoad } from './$types';

export const load: LayoutServerLoad = async ({ locals }) => {
  return {
    user: locals.user ?? null,
  };
};
```

```svelte
<!-- src/routes/+layout.svelte -->
<script lang="ts">
  let { data, children } = $props();

  import { Navbar } from '$lib/components';
</script>

{#if data.user}
  <Navbar user={data.user} />
{/if}

<main>
  {@render children()}
</main>
```

### API Endpoints

```typescript
// src/routes/api/products/+server.ts
import { json } from '@sveltejs/kit';
import type { RequestHandler } from './$types';

export const GET: RequestHandler = async ({ url }) => {
  const query = url.searchParams.get('q') ?? '';
  const limit = Number(url.searchParams.get('limit')) || 20;

  const products = await searchProducts(query, limit);

  return json({ data: products });
};

export const POST: RequestHandler = async ({ request }) => {
  const body = await request.json();

  if (!body.name) {
    return json({ error: 'Name is required' }, { status: 400 });
  }

  const product = await createProduct(body);

  return json({ data: product }, { status: 201 });
};
```

## SSR Considerations

### Browser-Only Code

```svelte
<script lang="ts">
  import { browser } from '$app/environment';

  let windowWidth = $state(0);

  $effect(() => {
    if (!browser) return;

    function onResize() {
      windowWidth = window.innerWidth;
    }

    onResize();
    window.addEventListener('resize', onResize);
    return () => window.removeEventListener('resize', onResize);
  });
</script>
```

### Shared Module Constants

```typescript
// $lib/constants.ts — safe for server and client
export const MAX_ITEMS = 100;
export const API_BASE = '/api';

// Avoid: using window, document, localStorage at module scope
```

## Accessibility

### Semantic Markup

```svelte
<nav aria-label="Main navigation">
  <ul>
    {#each items as item}
      <li>
        <a href={item.href} aria-current={item.active ? 'page' : undefined}>
          {item.label}
        </a>
      </li>
    {/each}
  </ul>
</nav>
```

### Keyboard Navigation

```svelte
<script lang="ts">
  let activeIndex = $state(0);

  function handleKeydown(event: KeyboardEvent) {
    switch (event.key) {
      case 'ArrowDown':
        event.preventDefault();
        activeIndex = Math.min(activeIndex + 1, items.length - 1);
        break;
      case 'ArrowUp':
        event.preventDefault();
        activeIndex = Math.max(activeIndex - 1, 0);
        break;
      case 'Enter':
        selectItem(items[activeIndex]);
        break;
      case 'Escape':
        close();
        break;
    }
  }
</script>

<div role="listbox" onkeydown={handleKeydown}>
  {#each items as item, i}
    <div
      role="option"
      aria-selected={i === activeIndex}
      class:selected={i === activeIndex}
      onclick={() => selectItem(item)}
    >
      {item.label}
    </div>
  {/each}
</div>
```

## Error Boundaries

### <svelte:boundary> for Graceful Error Handling

```svelte
<script lang="ts">
  function handleReset() {
    // Reset state that caused the error
  }
</script>

<svelte:boundary onerror={handleReset}>
  <ComplexComponent />

  {#snippet failed(error, reset)}
    <div class="error-fallback">
      <h2>Something went wrong</h2>
      <p>{error.message}</p>
      <button onclick={reset}>Try again</button>
    </div>
  {/snippet}
</svelte:boundary>
```

## Page State

### $app/state (SvelteKit 2.12+)

Prefer `$app/state` over `$app/stores`. The `page` object is now reactive state, not a writable store.

```svelte
<script lang="ts">
  // New: reactive state (SvelteKit 2.12+)
  import { page } from '$app/state';

  let title = $derived(page.data.title);
  let url = $derived(page.url.pathname);

  // Old: store pattern (avoid for new code)
  // import { page } from '$app/stores';
  // let title = $derived($page.data.title);
</script>

<h1>{title}</h1>
<p>Current path: {url}</p>
```

## Attachments

### {@attach} Directive

Use `{@attach}` to declaratively attach behaviors to elements without wrapper components.

```svelte
<script lang="ts">
  // Attachment directives enable reusable DOM behaviors
  // similar to custom directives but more composable
</script>
```

## Anti-Patterns

### Don't Mutate Without $state

```svelte
<!-- Bad: Non-reactive mutation -->
<script lang="ts">
  let items: Item[] = [];
  // This won't trigger reactivity:
  function addItem(item: Item) {
    items.push(item);
  }
</script>

<!-- Good: Reassign via $state -->
<script lang="ts">
  let items = $state<Item[]>([]);

  function addItem(item: Item) {
    items = [...items, item];
  }
</script>
```

### Don't Use @html with Untrusted Input

```svelte
<!-- Bad: XSS risk -->
<p>{@html userInput}</p>

<!-- Good: Escape or sanitize -->
<p>{userInput}</p>
```

### Don't Put Side Effects in Component Body

```svelte
<!-- Bad: Runs on every render -->
<script lang="ts">
  fetch('/api/data').then(r => r.json()).then(setData);
</script>

<!-- Good: Use $effect with cleanup -->
<script lang="ts">
  $effect(() => {
    const controller = new AbortController();
    fetch('/api/data', { signal: controller.signal })
      .then(r => r.json())
      .then(d => { data = d; });
    return () => controller.abort();
  });
</script>
```

### Don't Confuse +page.ts and +page.server.ts

```text
+page.server.ts — Runs on server only. Access database, secrets.
                   Use for secure data fetching and form actions.

+page.ts        — Runs on both server and client. No secrets.
                   Use for client-side data transformation and loading indicators.
```

## Best Practices

- Use `svelte-check` for TypeScript validation in components
- Prefer runes over stores for new code (Svelte 5)
- Use `$app/state` over `$app/stores` for page state (SvelteKit 2.12+)
- Use `<svelte:boundary>` for error boundaries instead of try/catch in components
- Use `+page.server.ts` for secure data; `+page.ts` for client transforms
- Always clean up `$effect` (return cleanup function for timers, listeners, subscriptions)
- Use `data-` attributes for test selectors, not CSS classes
- Keep components small and focused — use snippets for slot-like composition
- Use verified imports (`$lib/`, `$app/`) not relative paths across routes
- Run `pnpm check` or `svelte-check --tsconfig ./tsconfig.json` before commits
- Use `svelte-preprocess` only if needed — Svelte 5 has built-in TypeScript support

## Related Skills

- `frontend-patterns` for general React/UI patterns
- `tdd-workflow` for test-driven development
- `e2e-testing` for Playwright E2E patterns
- `security-review` for XSS and security patterns
