---
description: Comprehensive Elixir code review for OTP correctness, Ecto patterns, Phoenix conventions, and security. Invokes the elixir-reviewer agent.
---

# Elixir Code Review

This command invokes the **elixir-reviewer** agent for comprehensive Elixir-specific code review.

## What This Command Does

1. **Identify Elixir Changes**: Find modified `.ex` and `.exs` files via `git diff`
2. **Run Static Analysis**: Execute `mix credo --strict`, `mix format --check-formatted`
3. **Security Scan**: Check for Ecto query injection, secrets in config, unsafe LiveView rendering
4. **OTP Review**: Analyze GenServer patterns, supervision trees, process safety
5. **Code Quality**: Verify pattern matching, pipe operator usage, error handling
6. **Generate Report**: Categorize issues by severity

## When to Use

Use `/elixir-review` when:
- After writing or modifying Elixir code
- Before committing Elixir changes
- Reviewing pull requests with Elixir code
- Onboarding to a new Elixir/Phoenix codebase
- Learning idiomatic Elixir patterns

## Review Categories

### CRITICAL (Must Fix)
- Ecto query injection via string interpolation
- Hardcoded secrets in config files
- GenServer blocking on external calls
- Process leaks without supervision
- Unsafe `@html` in LiveView with user input
- Missing CSRF protection

### HIGH (Should Fix)
- Ignored return values from Repo operations
- Bare `raise` instead of typed exceptions
- `with` without `else` clause
- Non-idiomatic nested conditionals
- Missing `@spec` on public functions
- Wrong supervision strategy

### MEDIUM (Consider)
- N+1 queries missing preloads
- Missing pagination on large queries
- Missing `@doc` on public functions
- Code not formatted with `mix format`
- Using Task instead of Task.Supervisor

## Automated Checks Run

```bash
# Formatting
mix format --check-formatted

# Static analysis
mix credo --strict

# Type checking (if configured)
mix dialyzer

# Security scanning (Phoenix)
mix sobelow --config

# Tests
mix test
```

## Example Usage

```text
User: /elixir-review

Agent:
# Elixir Code Review Report

## Files Reviewed
- lib/my_app/accounts.ex (modified)
- lib/my_app_web/controllers/user_controller.ex (modified)

## Static Analysis Results
✓ mix format: No issues
✓ mix credo: 2 warnings

## Issues Found

[CRITICAL] Ecto Query Injection
File: lib/my_app/accounts.ex:28
Issue: String interpolation in Ecto query
```elixir
from(u in User, where: u.email == "#{email}")
```
Fix: Use parameterized ^value
```elixir
from(u in User, where: u.email == ^email)
```

[HIGH] Ignored Return Value
File: lib/my_app/accounts.ex:42
Issue: Repo.insert result not handled
```elixir
Repo.insert(changeset)
```
Fix: Handle both outcomes
```elixir
case Repo.insert(changeset) do
  {:ok, user} -> {:ok, user}
  {:error, changeset} -> {:error, changeset}
end
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

- Agent: `agents/elixir-reviewer.md`
- Skills: `skills/elixir-patterns/`
- Commands: `/elixir-test`, `/elixir-build`
