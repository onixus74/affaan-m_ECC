---
description: Fix Elixir/Mix build and compilation errors
agent: elixir-build-resolver
subtask: true
---

# Elixir Build Command

Fix Elixir/Mix build, compilation, and dependency errors: $ARGUMENTS

## Your Task

1. **Run mix compile**: `mix compile`
2. **Run mix format --check**: `mix format --check-formatted`
3. **Fix errors** one by one
4. **Verify fixes** don't introduce new errors

## Common Elixir Errors

### Compilation Errors
```
undefined function foo/0
```
**Fix**: Import or alias the module, define the function

### Type Errors
```
function clause cannot match
```
**Fix**: Add a catch-all clause or fix pattern match

### Dependency Errors
```
** (Mix) Unknown dependency
```
**Fix**: Add to mix.exs deps, run `mix deps.get`

### Warning: Unused Variable
```
variable "x" is unused
```
**Fix**: Prefix with `_` or remove

## Fix Order

1. **Dependency errors** - Fix mix.exs, deps.get
2. **Compilation errors** - Undefined functions/modules
3. **Type/spec errors** - Fix @spec, dialyzer
4. **Warnings** - Unused vars, formatting

## Build Commands

```bash
# Compile project
mix compile

# Get dependencies
mix deps.get

# Clean and rebuild
mix clean && mix compile

# Format check
mix format --check-formatted

# Run credo linter
mix credo --strict

# Run dialyzer
mix dialyzer

# Run tests
mix test
```

## Verification

After fixes:
```bash
mix compile          # Should succeed
mix format --check   # Should be formatted
mix test             # Tests should pass
```

---

**IMPORTANT**: Fix errors only. No refactoring, no improvements. Get the build green with minimal changes.
