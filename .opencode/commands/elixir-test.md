---
description: Elixir TDD workflow with ExUnit
agent: tdd-guide
subtask: true
---

# Elixir Test Command

Implement using Elixir TDD methodology with ExUnit: $ARGUMENTS

## Your Task

Apply test-driven development with Elixir idioms:

1. **Define modules** - Functions and structs
2. **Write ExUnit tests** - Comprehensive coverage
3. **Implement minimal code** - Pass the tests
4. **Verify** - Run full suite

## TDD Cycle for Elixir

### Step 1: Define Module
```elixir
defmodule MyApp.Calculator do
  @moduledoc "Calculator operations"
end
```

### Step 2: Write Tests
```elixir
defmodule MyApp.CalculatorTest do
  use ExUnit.Case, async: true

  describe "calculate/1" do
    test "returns correct result for valid input" do
      assert Calculator.calculate(%{a: 1, b: 2}) == {:ok, 3}
    end

    test "returns error for invalid input" do
      assert Calculator.calculate(%{a: nil, b: 2}) == {:error, :invalid_input}
    end
  end
end
```

### Step 3: Run Tests (RED)
```bash
mix test
```

### Step 4: Implement (GREEN)
```elixir
defmodule MyApp.Calculator do
  def calculate(%{a: a, b: b}) when is_number(a) and is_number(b) do
    {:ok, a + b}
  end

  def calculate(_), do: {:error, :invalid_input}
end
```

## Elixir Testing Commands

```bash
# Run all tests
mix test

# Run with verbose output
mix test --trace

# Run specific test file
mix test test/my_app/calculator_test.exs

# Run with coverage
mix test --cover

# Run only tagged tests
mix test --only integration
```

## Test File Organization

```
my_app/
├── lib/
│   └── my_app/
│       └── calculator.ex
├── test/
│   ├── my_app/
│   │   └── calculator_test.exs
│   ├── support/
│   │   └── fixtures.ex
│   └── test_helper.exs
└── mix.exs
```

---

**TIP**: Use `@tag :skip` to temporarily skip tests, and `@tag timeout: 10_000` for slow tests.
