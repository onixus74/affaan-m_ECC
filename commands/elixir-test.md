---
description: Enforce TDD workflow for Elixir. Write ExUnit tests first, then implement. Verify 80%+ coverage with mix test --cover.
---

# Elixir TDD Command

This command enforces test-driven development methodology for Elixir code using ExUnit.

## What This Command Does

1. **Define Function Signatures**: Scaffold module and function heads first
2. **Write ExUnit Tests**: Create comprehensive test cases (RED)
3. **Run Tests**: Verify tests fail for the right reason
4. **Implement Code**: Write minimal code to pass (GREEN)
5. **Refactor**: Improve while keeping tests green
6. **Check Coverage**: Ensure 80%+ coverage

## When to Use

Use `/elixir-test` when:
- Implementing new Elixir modules or functions
- Adding test coverage to existing code
- Fixing bugs (write failing test first)
- Building critical business logic
- Learning TDD workflow in Elixir

## TDD Cycle

```
RED     → Write failing ExUnit test
GREEN   → Implement minimal code to pass
REFACTOR → Improve code, tests stay green
REPEAT  → Next test case
```

## Example Session

````markdown
User: /elixir-test I need a function to validate email addresses

Agent:
# TDD Session: Email Validator

## Step 1: Define Module

```elixir
# lib/my_app/validator/email.ex
defmodule MyApp.Validator.Email do
  @doc "Validates email format. Returns :ok or {:error, reason}"
  @spec validate(String.t()) :: :ok | {:error, atom()}
  def validate(email) do
    raise "not implemented"
  end
end
```

## Step 2: Write Tests (RED)

```elixir
# test/my_app/validator/email_test.exs
defmodule MyApp.Validator.EmailTest do
  use ExUnit.Case, async: true
  alias MyApp.Validator.Email

  describe "validate/1" do
    test "accepts valid emails" do
      assert :ok = Email.validate("user@example.com")
      assert :ok = Email.validate("user+tag@example.com")
      assert :ok = Email.validate("first.last@example.com")
    end

    test "rejects empty string" do
      assert {:error, :empty} = Email.validate("")
    end

    test "rejects missing @" do
      assert {:error, :invalid_format} = Email.validate("userexample.com")
    end

    test "rejects missing domain" do
      assert {:error, :invalid_format} = Email.validate("user@")
    end

    test "rejects missing local part" do
      assert {:error, :invalid_format} = Email.validate("@example.com")
    end
  end
end
```

## Step 3: Run Tests - Verify FAIL

```bash
$ mix test test/my_app/validator/email_test.exs

  1) test validate/1 accepts valid emails (MyApp.Validator.EmailTest)
     ** (RuntimeError) not implemented
```

✓ Tests fail as expected.

## Step 4: Implement Minimal Code (GREEN)

```elixir
# lib/my_app/validator/email.ex
defmodule MyApp.Validator.Email do
  @email_regex ~r/^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$/

  @spec validate(String.t()) :: :ok | {:error, atom()}
  def validate(""), do: {:error, :empty}

  def validate(email) when is_binary(email) do
    if Regex.match?(@email_regex, email) do
      :ok
    else
      {:error, :invalid_format}
    end
  end
end
```

## Step 5: Run Tests - Verify PASS

```bash
$ mix test test/my_app/validator/email_test.exs

  All 5 tests passed.
```

✓ All tests passing!

## Step 6: Check Coverage

```bash
$ mix test --cover

  100.0% | lib/my_app/validator/email.ex
```

✓ Coverage: 100%

## TDD Complete!
````

## Test Patterns

### Describe/Test Structure
```elixir
describe "function_name/arity" do
  test "handles valid input" do
    assert {:ok, result} = Module.function(input)
    assert result == expected
  end

  test "handles invalid input" do
    assert {:error, reason} = Module.function(bad_input)
  end
end
```

### DataCase for Database Tests
```elixir
defmodule MyApp.UserTest do
  use MyApp.DataCase, async: true

  test "creates user with valid attrs" do
    attrs = %{email: "test@example.com", name: "Test"}
    assert {:ok, user} = Accounts.create_user(attrs)
    assert user.email == "test@example.com"
  end
end
```

### Mox for Mocking
```elixir
import Mox

setup :verify_on_exit!

test "sends welcome email on registration" do
  EmailClientMock
  |> expect(:send_welcome, fn email ->
    assert email == "test@example.com"
    {:ok, :sent}
  end)

  assert {:ok, _user} = UserService.register(%{email: "test@example.com"})
end
```

## Coverage Commands

```bash
# Run with coverage
mix test --cover

# Generate HTML report
mix test --cover --export-coverage default
```

## Coverage Targets

| Code Type | Target |
|-----------|--------|
| Business logic (contexts) | 90%+ |
| Public API | 85%+ |
| General code | 80%+ |
| Generated code | Exclude |

## Parameterized Tests (Elixir 1.18+)

Run the same test logic across multiple inputs without writing repeated test blocks:

```elixir
defmodule MyApp.Validator.EmailTest do
  use ExUnit.Case, async: true,
    parameterize: [
      %{email: "user@example.com", valid: true},
      %{email: "user+tag@example.com", valid: true},
      %{email: "", valid: false},
      %{email: "no-at-sign", valid: false},
      %{email: "@nodomain.com", valid: false}
    ]

  test "validates email: {email}", %{email: email, valid: expected} do
    result = MyApp.Validator.Email.validate(email)
    assert match?(:ok, result) == expected or result == :ok == expected
  end
end
```

## TDD Best Practices

**DO:**
- Write test FIRST, before implementation
- Use `async: true` for isolated tests
- Use pattern matching in assertions
- Test edge cases (nil, empty string, large values)
- Use `setup` blocks for common fixtures

**DON'T:**
- Write implementation before tests
- Skip the RED phase
- Use `Code.eval_file` or private function tests
- Ignore flaky tests
- Use `ExUnit.Callbacks` for shared mutable state

## Related

- Agent: `agents/elixir-reviewer.md`
- Skills: `skills/elixir-patterns/`, `skills/tdd-workflow/`
- Commands: `/elixir-review`, `/elixir-build`
