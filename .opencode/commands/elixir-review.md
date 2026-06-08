---
description: Elixir code review for OTP, Phoenix, and idiomatic patterns
agent: elixir-reviewer
subtask: true
---

# Elixir Review Command

Review Elixir code for OTP correctness, Phoenix best practices, and idiomatic patterns: $ARGUMENTS

## Your Task

1. **Analyze Elixir code** for idioms and functional patterns
2. **Check OTP** - GenServer, Supervisor, Application
3. **Review Phoenix** - LiveView, controllers, contexts
4. **Verify Ecto** - schemas, changesets, queries

## Review Checklist

### Idiomatic Elixir
- [ ] Module naming (CamelCase)
- [ ] Function naming (snake_case)
- [ ] Proper use of pipes (`|>`)
- [ ] Pattern matching in function heads

### OTP
- [ ] GenServer callbacks correct
- [ ] Supervisor strategies appropriate
- [ ] Process naming conventions
- [ ] Telemetry instrumentation

### Phoenix
- [ ] LiveView lifecycle correct
- [ ] Context boundaries clean
- [ ] Router organization
- [ ] Proper use of components

### Ecto
- [ ] Changeset validations complete
- [ ] Query composition efficient
- [ ] Schema associations correct
- [ ] Migration safety

## Report Format

### Idiomatic Issues
- [file:line] Issue description
  Suggestion: How to fix

### OTP Issues
- [file:line] Issue description
  Suggestion: How to fix

### Phoenix Issues
- [file:line] Issue description
  Suggestion: How to fix

### Ecto Issues
- [file:line] Issue description
  Suggestion: How to fix

---

**TIP**: Run `mix credo --strict` and `mix dialyzer` for additional automated checks.
