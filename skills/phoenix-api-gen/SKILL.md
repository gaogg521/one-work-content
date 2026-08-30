---
name: phoenix-api-gen
description: 从 OpenAPI 规范或自然语言描述生成完整的 Phoenix JSON API。自动创建 contexts、Ecto schemas、migrations、controllers、JSON views、router、ExUnit 测试与 factories、auth plugs 及租户隔离(tenant scoping)。适用于构建 Phoenix REST API、添加 CRUD 端点(endpoints)、资源脚手架(scaffolding)或转换 OpenAPI YAML。触发词：Phoenix API、OpenAPI、Ecto、CRUD、REST API、Elixir
tags:
- API
- Schema
- 开发
---

# Phoenix API Generator

## Workflow

### From OpenAPI YAML

1. 解析 OpenAPI spec — 提取 paths、schemas、request/response bodies。
2. 将每个 schema 映射到 Ecto schema + migration。
3. 将每个 path 映射到 controller action；按 resource context 分组。
4. 从 `securitySchemes` 生成 auth plugs。
5. 生成覆盖 happy path + validation errors 的 ExUnit 测试。

### From Natural Language

1. 从描述中提取 resources、fields、types 和 relationships。
2. 推断 context 边界（将相关 resources 分组）。
3. 生成 schemas、migrations、controllers、views、router 和 tests。
4. 在写入文件前请用户确认。

## File Generation Order

1. Migrations（timestamps 前缀：`YYYYMMDDHHMMSS`）
2. Ecto schemas + changesets
3. Context modules（CRUD functions）
4. Controllers + FallbackController
5. JSON renderers（Phoenix 1.7+ `*JSON` modules，或旧版本的 `*View`）
6. Router scope + pipelines
7. Auth plugs
8. Tests + factories

## Phoenix Conventions

参见 [references/phoenix-conventions.md](references/phoenix-conventions.md) 了解 project structure、naming、context patterns。

关键规则：
- 每个 bounded domain 一个 context（例如 `Accounts`、`Billing`、`Notifications`）。
- Context 是 public API — controllers 从不直接调用 Repo。
- Schemas 位于 contexts 下：`MyApp.Accounts.User`。
- Controllers 委托给 contexts；返回 `{:ok, resource}` 或 `{:error, changeset}`。
- 使用 `FallbackController` 和 `action_fallback/1` 处理 error tuples。

## Ecto Patterns

参见 [references/ecto-patterns.md](references/ecto-patterns.md) 了解 schema、changeset、migration 细节。

关键规则：
- 始终使用 `timestamps(type: :utc_datetime_usec)`。
- Binary IDs：`@primary_key {:id, :binary_id, autogenerate: true}` + `@foreign_key_type :binary_id`。
- 当 create/update 字段不同时，分离 `create_changeset/2` 和 `update_changeset/2`。
- 在 changesets 中验证 required fields、formats 和 constraints — 不在 controllers 中验证。

## Multi-Tenancy

在每个 tenant-scoped 表中添加 `tenant_id :binary_id`。模式：

```elixir
# 在 context 中
def list_resources(tenant_id) do
  Resource
  |> where(tenant_id: ^tenant_id)
  |> Repo.all()
end

# 在 plug 中 — 从 conn 提取 tenant 并 assign
defmodule MyAppWeb.Plugs.SetTenant do
  import Plug.Conn
  def init(opts), do: opts
  def call(conn, _opts) do
    tenant_id = get_req_header(conn, "x-tenant-id") |> List.first()
    assign(conn, :tenant_id, tenant_id)
  end
end
```

始终在 `[:tenant_id, <resource_id 或 lookup field>]` 上添加 composite index。

## Auth Plugs

### API Key

```elixir
defmodule MyAppWeb.Plugs.ApiKeyAuth do
  import Plug.Conn
  def init(opts), do: opts
  def call(conn, _opts) do
    with [key] <- get_req_header(conn, "x-api-key"),
         {:ok, account} <- Accounts.authenticate_api_key(key) do
      assign(conn, :current_account, account)
    else
      _ -> conn |> send_resp(401, "Unauthorized") |> halt()
    end
  end
end
```

### Bearer Token

```elixir
defmodule MyAppWeb.Plugs.BearerAuth do
  import Plug.Conn
  def init(opts), do: opts
  def call(conn, _opts) do
    with ["Bearer " <> token] <- get_req_header(conn, "authorization"),
         {:ok, claims} <- MyApp.Token.verify(token) do
      assign(conn, :current_user, claims)
    else
      _ -> conn |> send_resp(401, "Unauthorized") |> halt()
    end
  end
end
```

## Router Structure

```elixir
scope "/api/v1", MyAppWeb do
  pipe_through [:api, :authenticated]

  resources "/users", UserController, except: [:new, :edit]
  resources "/teams", TeamController, except: [:new, :edit] do
    resources "/members", MemberController, only: [:index, :create, :delete]
  end
end
```

## Test Generation

参见 [references/test-patterns.md](references/test-patterns.md) 了解 ExUnit、Mox、factory patterns。

关键规则：
- 在所有不共享 state 的测试上使用 `async: true`。
- 使用 `Ecto.Adapters.SQL.Sandbox` 进行 DB isolation。
- Factory module 使用 `ex_machina` 或手写的 `build/1`、`insert/1`。
- 分别测试 contexts 和 controllers。
- 对于 controllers：测试 status codes、response body shape 和 error cases。
- 使用 `Mox` mock 外部服务 — 定义 behaviours，在测试中设置 expectations。

### Controller Test Template

```elixir
defmodule MyAppWeb.UserControllerTest do
  use MyAppWeb.ConnCase, async: true

  import MyApp.Factory

  setup %{conn: conn} do
    user = insert(:user)
    conn = put_req_header(conn, "authorization", "Bearer #{token_for(user)}")
    {:ok, conn: conn, user: user}
  end

  describe "index" do
    test "lists users", %{conn: conn} do
      conn = get(conn, ~p"/api/v1/users")
      assert %{"data" => users} = json_response(conn, 200)
      assert is_list(users)
    end
  end

  describe "create" do
    test "returns 201 with valid params", %{conn: conn} do
      params = params_for(:user)
      conn = post(conn, ~p"/api/v1/users", user: params)
      assert %{"data" => %{"id" => _}} = json_response(conn, 201)
    end

    test "returns 422 with invalid params", %{conn: conn} do
      conn = post(conn, ~p"/api/v1/users", user: %{})
      assert json_response(conn, 422)["errors"] != %{}
    end
  end
end
```

## JSON Renderer (Phoenix 1.7+)

```elixir
defmodule MyAppWeb.UserJSON do
  def index(%{users: users}), do: %{data: for(u <- users, do: data(u))}
  def show(%{user: user}), do: %{data: data(user)}

  defp data(user) do
    %{
      id: user.id,
      email: user.email,
      inserted_at: user.inserted_at
    }
  end
end
```

## Checklist Before Writing

- [ ] Migrations 使用 `timestamps(type: :utc_datetime_usec)`
- [ ] 如果项目使用 UUIDs，配置 Binary IDs
- [ ] 在需要的地方应用 Tenant scoping
- [ ] Auth plug 接入 router pipeline
- [ ] FallbackController 处理 `{:error, changeset}` 和 `{:error, :not_found}`
- [ ] Tests 覆盖 200、201、404、422 status codes
- [ ] 为每个 schema 定义 Factory
