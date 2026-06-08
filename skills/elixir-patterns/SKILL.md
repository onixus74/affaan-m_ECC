---
name: elixir-patterns
description: Idiomatic Elixir and Phoenix patterns covering OTP, GenServer, Ecto, LiveView, supervision, pattern matching, and Mix conventions for building fault-tolerant applications.
origin: ECC
---

# Elixir & Phoenix Development Patterns

Idiomatic Elixir patterns, OTP conventions, and Phoenix/LiveView best practices for building fault-tolerant, concurrent applications.

## When to Activate

- Writing Elixir code (GenServer, Supervisor, Agent, Task)
- Working with Ecto schemas, changesets, queries, or migrations
- Building Phoenix controllers, LiveView, or LiveComponents
- Designing OTP supervision trees
- Writing ExUnit tests
- Debugging Mix build or dependency issues
- Reviewing Elixir code for idiomatic patterns

## Core Principles

### 1. Immutability and Functional Purity

Elixir data is immutable. Every transformation returns a new data structure.

```elixir
# Good: Pipe operator chains transformations
user
|> Map.put(:name, "Alice")
|> Map.put(:active, true)

# Bad: Re-binding with intermediate variables
user1 = Map.put(user, :name, "Alice")
user2 = Map.put(user1, :active, true)
```

### 2. Pattern Matching Over Conditionals

Prefer pattern matching in function heads and `case`/`with` over nested `if`/`cond`.

```elixir
# Good: Pattern matching in function heads
def handle_event({:user_joined, user}, state) do
  {:ok, add_user(state, user)}
end

def handle_event({:user_left, user_id}, state) do
  {:ok, remove_user(state, user_id)}
end

# Bad: Nested conditionals
def handle_event(event, state) do
  if elem(event, 0) == :user_joined do
    {:ok, add_user(state, elem(event, 1))}
  else
    if elem(event, 0) == :user_left do
      {:ok, remove_user(state, elem(event, 1))}
    end
  end
end
```

### 3. Let It Crash

Design for failure recovery through supervision trees rather than defensive programming.

```elixir
# Good: Supervisor restarts the GenServer on failure
children = [
  {MyApp.Worker, arg}
]

Supervisor.init(children, strategy: :one_for_one)

# Bad: Defensive try/catch everywhere hiding failures
def handle_call(:process, _from, state) do
  try do
    result = risky_operation(state)
    {:reply, {:ok, result}, state}
  rescue
    _ -> {:reply, {:error, :something_failed}, state}
  end
end
```

## OTP Patterns

### GenServer Best Practices

```elixir
defmodule MyApp.UserSession do
  use GenServer

  # Client API
  def start_link(opts) do
    name = Keyword.get(opts, :name, __MODULE__)
    GenServer.start_link(__MODULE__, opts, name: name)
  end

  def get_session(pid, user_id) do
    GenServer.call(pid, {:get_session, user_id})
  end

  def put_session(pid, user_id, data) do
    GenServer.cast(pid, {:put_session, user_id, data})
  end

  # Server callbacks
  @impl true
  def init(opts) do
    initial_state = %{
      sessions: %{},
      ttl: Keyword.get(opts, :ttl, 3600)
    }
    schedule_cleanup()
    {:ok, initial_state}
  end

  @impl true
  def handle_call({:get_session, user_id}, _from, state) do
    result = Map.get(state.sessions, user_id)
    {:reply, result, state}
  end

  @impl true
  def handle_cast({:put_session, user_id, data}, state) do
    new_sessions = Map.put(state.sessions, user_id, data)
    {:noreply, %{state | sessions: new_sessions}}
  end

  @impl true
  def handle_info(:cleanup, state) do
    new_sessions = expire_old_sessions(state.sessions, state.ttl)
    schedule_cleanup()
    {:noreply, %{state | sessions: new_sessions}}
  end

  defp schedule_cleanup do
    Process.send_after(self(), :cleanup, to_timeout(minute: 5))
  end
end
```

### Supervision Trees

```elixir
defmodule MyApp.Application do
  use Application

  @impl true
  def start(_type, _args) do
    children = [
      MyApp.Repo,
      {Phoenix.PubSub, name: MyApp.PubSub},
      {MyApp.UserSession, name: MyApp.UserSession},
      {Task.Supervisor, name: MyApp.TaskSupervisor}
    ]

    opts = [strategy: :one_for_one, name: MyApp.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```

### Task for Concurrent Work

```elixir
# Good: Supervised tasks with proper error handling
def fetch_all_users(user_ids) do
  tasks =
    Enum.map(user_ids, fn id ->
      Task.Supervisor.async_nolink(MyApp.TaskSupervisor, fn ->
        fetch_user(id)
      end)
    end)

  tasks
  |> Task.yield_many(timeout: 5000)
  |> Enum.map(fn {task, result} ->
    case result do
      {:ok, {:ok, user}} -> {:ok, user}
      {:ok, {:error, reason}} -> {:error, reason}
      {:exit, reason} -> {:error, {:task_died, reason}}
      nil ->
        Task.shutdown(task, :brutal_kill)
        {:error, :timeout}
    end
  end)
end
```

## Ecto Patterns

### Schema and Changeset

```elixir
defmodule MyApp.Accounts.User do
  use Ecto.Schema
  import Ecto.Changeset

  schema "users" do
    field :email, :string
    field :name, :string
    field :role, :string, default: "member"
    has_many :orders, MyApp.Orders.Order

    timestamps()
  end

  @required_fields ~w(email name)a
  @optional_fields ~w(role)a

  def changeset(user, attrs) do
    user
    |> cast(attrs, @required_fields ++ @optional_fields)
    |> validate_required(@required_fields)
    |> validate_format(:email, ~r/^[^\s]+@[^\s]+$/)
    |> validate_length(:name, min: 1, max: 200)
    |> unique_constraint(:email)
  end

  def registration_changeset(user, attrs) do
    user
    |> changeset(attrs)
    |> validate_password(:password)
  end
end
```

### Simple Lookups with all_by

```elixir
# Good: all_by for attribute-based lookups (Ecto 3.13+)
Repo.all_by(User, email: "alice@example.com")
Repo.all_by(User, role: "admin")

# Equivalent to:
Repo.all(from(u in User, where: u.email == ^email)
```

### Composable Queries

```elixir
defmodule MyApp.Orders.Order do
  import Ecto.Query

  def by_status(query \\ __MODULE__, status) do
    where(query, [o], o.status == ^status)
  end

  def with_user(query \\ __MODULE__) do
    preload(query, [:user])
  end

  def recent(query \\ __MODULE__, limit \\ 10) do
    query
    |> order_by([o], desc: o.inserted_at)
    |> limit(^limit)
  end
end

# Usage
Order
|> Order.by_status("active")
|> Order.with_user()
|> Order.recent(20)
|> Repo.all()
```

### Safe Query Construction

```elixir
# Good: Parameterized queries prevent injection
def search_users(term) do
  pattern = "%#{term}%"

  from(u in User,
    where: ilike(u.name, ^pattern) or ilike(u.email, ^pattern),
    limit: 20
  )
  |> Repo.all()
end

# Bad: String interpolation in queries
def search_users(term) do
  from(u in User, where: ilike(u.name, "%#{term}%"))
  |> Repo.all()
end
```

### Transactions

Prefer `Repo.transact/2` (Ecto 3.13+) over `Repo.transaction/2`. Use `Ecto.Multi` only when you need named steps and rollback granularity.

```elixir
# Good: Simple transaction with transact/2
def create_user_with_profile(attrs) do
  Repo.transact(fn repo ->
    with {:ok, user} <- repo.insert(User.changeset(%User{}, attrs)),
         {:ok, profile} <- repo.insert(Profile.changeset(%Profile{}, Map.put(attrs, :user_id, user.id))) do
      {:ok, %{user: user, profile: profile}}
    end
  end)
end

# Good: Complex multi-step with named rollbacks using Multi
def transfer(from_id, to_id, amount) do
  Multi.new()
  |> Multi.run(:debit, fn repo, _ ->
    debit_account(repo, from_id, amount)
  end)
  |> Multi.run(:credit, fn repo, _ ->
    credit_account(repo, to_id, amount)
  end)
  |> Multi.insert(:log, fn %{debit: debit, credit: credit} ->
    create_transfer_log(debit, credit, amount)
  end)
  |> Repo.transact()
  |> case do
    {:ok, result} -> {:ok, result}
    {:error, step, changeset, _} -> {:error, {step, changeset}}
  end
end
```

## Phoenix Patterns

### Controller Conventions

```elixir
defmodule MyAppWeb.UserController do
  use MyAppWeb, :controller

  plug :authenticate when action in [:create, :update, :delete]

  def index(conn, params) do
    users =
      User
      |> User.by_status(params["status"])
      |> Repo.paginate(params)

    json(conn, %{data: users})
  end

  def show(conn, %{"id" => id}) do
    case Repo.get(User, id) do
      nil -> conn |> put_status(404) |> json(%{error: "not found"})
      user -> json(conn, %{data: user})
    end
  end
end
```

### LiveView Best Practices

```elixir
defmodule MyAppWeb.ProductLive.Show do
  use MyAppWeb, :live_view

  @impl true
  def mount(%{"id" => id}, _session, socket) do
    if connected?(socket) do
      Phoenix.PubSub.subscribe(MyApp.PubSub, "product:#{id}")
    end

    product = Catalog.get_product!(id)

    {:ok,
     socket
     |> assign(:product, product)
     |> assign(:quantity, 1)}
  end

  @impl true
  def handle_event("add_to_cart", %{"quantity" => qty}, socket) do
    case Cart.add_item(socket.assigns.current_user, socket.assigns.product, qty) do
      {:ok, _item} ->
        {:noreply,
         socket
         |> put_flash(:info, "Added to cart")
         |> push_navigate(to: ~p"/cart")}

      {:error, changeset} ->
        {:noreply, assign(socket, :changeset, changeset)}
    end
  end

  @impl true
  def handle_info({:product_updated, product}, socket) do
    {:noreply, assign(socket, :product, product)}
  end
end
```

### Prefer Function Components Over LiveComponents

Since LiveView 1.1, comprehensions with `:key` have built-in change tracking. This eliminates the main performance reason for using LiveComponents.

```heex
<!-- Good: Efficient rendering with :key (LiveView 1.1+) -->
<ul>
  <li :for={item <- @items} :key={item.id}>
    <%= item.name %> — <%= format_price(item.price) %>
  </li>
</ul>
```

Use **function components** (via `Phoenix.Component`) for most encapsulation. They are simpler, testable, and have no lifecycle overhead.

```elixir
defmodule MyAppWeb.ProductComponents do
  use Phoenix.Component

  attr :product, :map, required: true
  attr :on_add, :any, required: true

  def product_card(assigns) do
    ~H"""
    <div class="product-card">
      <h3>{@product.name}</h3>
      <p>{@product.description}</p>
      <button phx-click="add_to_cart" phx-value-id={@product.id}>
        Add to Cart
      </button>
    </div>
    """
  end
end
```

### When to Use LiveComponent

Only use `Phoenix.LiveComponent` when you need **both event handling AND independent state** within the component. Do not use LiveComponents for code organization alone.

```elixir
# Good use of LiveComponent: independent state + event handling
defmodule MyAppWeb.ProductLive.CartComponent do
  use MyAppWeb, :live_component

  @impl true
  def update(assigns, socket) do
    {:ok, assign(socket, items: Cart.list_items(assigns.cart_id))}
  end

  @impl true
  def render(assigns) do
    ~H"""
    <div class="cart">
      <h2>Cart ({length(@items)})</h2>
      <div :for={item <- @items} :key={item.id} class="cart-item">
        <span>{item.product.name}</span>
        <span>{item.quantity}</span>
        <button phx-click="remove" phx-value-id={item.id} phx-target={@myself}>
          Remove
        </button>
      </div>
    </div>
    """
  end

  @impl true
  def handle_event("remove", %{"id" => id}, socket) do
    Cart.remove_item(id)
    {:noreply, assign(socket, :items, Cart.list_items(socket.assigns.cart_id))}
  end
end
```

### LiveView 1.1+ Features

```heex
<!-- Portal: teleport content outside component hierarchy -->
<.portal target="#modal-root">
  <div class="modal">Rendered in #modal-root</div>
</.portal>
```

## Error Handling Patterns

### With Statement for Pipelined Error Handling

```elixir
# Good: Clean error flow with `with`
def process_order(attrs) do
  with {:ok, changeset} <- validate_order(attrs),
       {:ok, order} <- create_order(changeset),
       {:ok, _payment} <- charge_payment(order),
       {:ok, _email} <- send_confirmation(order) do
    {:ok, order}
  else
    {:error, %Ecto.Changeset{} = changeset} ->
      {:error, {:validation, changeset}}

    {:error, :payment_failed} ->
      {:error, :payment_declined}

    {:error, reason} ->
      {:error, reason}
  end
end
```

### Custom Exceptions

```elixir
defmodule MyApp.NotFoundError do
  defexception [:message, :resource, :id]

  @impl true
  def exception(opts) do
    resource = Keyword.fetch!(opts, :resource)
    id = Keyword.fetch!(opts, :id)
    msg = "#{resource} with id #{id} not found"
    %__MODULE__{message: msg, resource: resource, id: id}
  end
end
```

## Testing Patterns

### ExUnit with Mox

```elixir
defmodule MyApp.UserServiceTest do
  use MyApp.DataCase, async: true
  import Mox

  alias MyApp.UserService
  alias MyApp.Mocks.{EmailClient, PaymentGateway}

  setup :verify_on_exit!

  describe "register_user/1" do
    test "creates user and sends welcome email" do
      EmailClient
      |> expect(:send_welcome, fn email ->
        assert email == "test@example.com"
        {:ok, :sent}
      end)

      assert {:ok, user} = UserService.register_user(%{
        email: "test@example.com",
        name: "Test"
      })

      assert user.email == "test@example.com"
    end

    test "returns error for duplicate email" do
      insert(:user, email: "taken@example.com")

      assert {:error, changeset} = UserService.register_user(%{
        email: "taken@example.com",
        name: "Dup"
      })

      assert "has already been taken" in errors_on(changeset).email
    end
  end
end
```

### Parameterized Tests (Elixir 1.18+)

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

  test "validates email format", %{email: email, valid: expected} do
    result = MyApp.Validator.Email.validate(email)
    assert match?({:ok, _}, result) == expected or result == :ok == expected
  end
end
```

### DataCase Helper

```elixir
defmodule MyApp.DataCase do
  use ExUnit.CaseTemplate

  using do
    quote do
      alias MyApp.Repo
      import Ecto
      import Ecto.Changeset
      import Ecto.Query
      import MyApp.Factory
      import MyApp.DataCase
    end
  end

  setup tags do
    pid = Ecto.Adapters.SQL.Sandbox.start_owner!(MyApp.Repo, shared: not tags[:async])

    on_exit(fn -> Ecto.Adapters.SQL.Sandbox.stop_owner(pid) end)

    :ok
  end
end
```

## Anti-Patterns

### Don't Ignore Return Values

```elixir
# Bad: Ignoring {:ok, _} or {:error, _}
Repo.insert(changeset)
send_email(user.email)

# Good: Handle all outcomes
case Repo.insert(changeset) do
  {:ok, user} -> send_email(user.email)
  {:error, changeset} -> {:error, changeset}
end
```

### Don't Use Process Dictionary

```elixir
# Bad: Hidden mutable state
Process.put(:current_user, user)

# Good: Explicit state passing
defmodule MyApp.Context do
  defstruct [:current_user, :tenant]
end
```

### Don't Block in GenServer

```elixir
# Bad: Synchronous external call in GenServer
def handle_call(:fetch_data, _from, state) do
  # Blocks the GenServer
  data = HTTPoison.get!("https://api.example.com/data")
  {:reply, data, state}
end

# Good: Use Task for external calls
def handle_call(:fetch_data, _from, state) do
  task = Task.Supervisor.async_nolink(MyApp.TaskSupervisor, fn ->
    HTTPoison.get("https://api.example.com/data")
  end)
  {:reply, task, state}
end
```

### Don't Use String Interpolation in Queries

```elixir
# Bad: SQL injection risk
query = "SELECT * FROM users WHERE email = '#{email}'"

# Good: Parameterized via Ecto
from(u in User, where: u.email == ^email)
```

## Best Practices

- Use `mix format` before every commit; use `mix format --migrate` to auto-migrate deprecated constructs
- Enable `mix credo --strict` for static analysis
- Use Dialyzer with `@spec` annotations for public functions
- Prefer `Repo.transact/2` over `Repo.transaction/2` (Ecto 3.13+)
- Use built-in `JSON` module instead of Jason (Elixir 1.18+)
- Use `to_timeout/1` with `Duration` for timeouts instead of `:timer` module (Elixir 1.17+)
- Prefer function components over LiveComponents; only use LiveComponent for event handling + independent state
- Use `:key` on comprehensions for efficient change tracking (LiveView 1.1+)
- Keep GenServer state as a map with documented keys
- Prefer `with` over nested `case` for multi-step flows
- Use `@doc` and `@moduledoc` for all public modules and functions
- Organize contexts by domain, not by type
- Use verified routes (`~p"/path"`) in Phoenix
- Test async by default (`use ExUnit.Case, async: true`)
- Use `Ecto.Adapters.SQL.Sandbox` for test isolation
- Use `Repo.all_by/3` for simple attribute-based lookups (Ecto 3.13+)

## Related Skills

- `tdd-workflow` for test-driven development
- `security-review` for security patterns
- `api-design` for REST API conventions
- `backend-patterns` for general server-side patterns
