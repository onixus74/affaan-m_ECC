---
description: Fix Elixir build errors, mix compile warnings, dependency issues, and dialyzer warnings incrementally. Invokes the elixir-build-resolver agent.
---

# Elixir Build and Fix

This command invokes the **elixir-build-resolver** agent to incrementally fix Elixir build errors with minimal changes.

## What This Command Does

1. **Run Diagnostics**: Execute `mix compile`, `mix deps.get`, `mix format --check-formatted`
2. **Parse Errors**: Group by file and sort by severity
3. **Fix Incrementally**: One error at a time
4. **Verify Each Fix**: Re-run compile after each change
5. **Report Summary**: Show what was fixed and what remains

## When to Use

Use `/elixir-build` when:
- `mix compile` fails with errors
- `mix deps.get` reports dependency conflicts
- `mix dialyzer` shows type warnings
- `mix format --check-formatted` reports unformatted code
- After pulling changes that break the build

## Diagnostic Commands Run

```bash
# Primary compile check
mix compile

# Dependency check
mix deps.get
mix deps.compile

# Format check
mix format --check-formatted

# Static analysis
mix credo --strict

# Type checking (if configured)
mix dialyzer
```

## Example Session

```text
User: /elixir-build

Agent:
# Elixir Build Resolution

## Initial Diagnostics

```bash
$ mix compile
== Compilation error in file lib/my_app/accounts.ex ==
** (CompileError) undefined function get_user/1
== Compilation error in file lib/my_app_web/controllers/user_controller.ex ==
** (CompileError) function index/2 undefined
```

Errors found: 2

## Fix 1: Undefined Function

File: lib/my_app/accounts.ex:15
Error: undefined function get_user/1
Cause: Missing alias

```elixir
# Added alias
alias MyApp.Users

# Changed
get_user(id)
# To
Users.get_user(id)
```

```bash
$ mix compile
# 1 error remaining
```

## Fix 2: Missing Function Clause

File: lib/my_app_web/controllers/user_controller.ex:8
Error: function index/2 undefined
Cause: Function not defined

```elixir
# Added missing function
def index(conn, _params) do
  users = Accounts.list_users()
  json(conn, %{data: users})
end
```

```bash
$ mix compile
# Compilation successful!
```

## Final Verification

```bash
$ mix format --check-formatted
# All files formatted

$ mix test
All 45 tests passed.
```

## Summary

| Metric | Count |
|--------|-------|
| Build errors fixed | 2 |
| Files modified | 2 |
| Remaining issues | 0 |

Build Status: SUCCESS
```

## Common Errors Fixed

| Error | Typical Fix |
|-------|-------------|
| `undefined function` | Add import, alias, or define the function |
| `no match of right hand value` | Fix pattern match or use case/if |
| `could not compile dependency` | Pin version in mix.exs, clean deps |
| `deps resolution failed` | Unlock and re-resolve dependencies |
| `dialyzer: no local return` | Fix @spec or function return type |
| `unused variable` | Prefix with `_` or remove |
| `unbound variable` | Fix scope or initialize |

## Fix Strategy

1. **Compile errors first** - Code must compile
2. **Dependency issues second** - All deps must resolve
3. **Format issues third** - Code must be formatted
4. **One fix at a time** - Verify each change
5. **Minimal changes** - Don't refactor, just fix

## Stop Conditions

The agent will stop and report if:
- Same error persists after 3 attempts
- Fix introduces more errors
- Requires architectural changes
- Dependency conflict requires manual version negotiation

## Related

- Agent: `agents/elixir-build-resolver.md`
- Skills: `skills/elixir-patterns/`
- Commands: `/elixir-test`, `/elixir-review`
