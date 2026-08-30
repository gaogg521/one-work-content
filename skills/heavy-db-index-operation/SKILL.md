---
name: heavy-db-index-operation
description: 使用 Explorer.Migrator.HeavyDbIndexOperation 框架生成后台迁移模块，用于在大表上创建、删除或重命名数据库索引。自动更新 BackgroundMigrations 缓存模块以进行适当的跟踪。这些迁移在后台运行，具有进度跟踪和依赖管理功能。在对大表（logs、internal_transactions、token_transfers、addresses、transactions、blocks 等）执行删除/创建/重命名索引请求时使用此技能，以避免阻塞数据库。
---

## 概述

heavy-db-index-operation 技能帮助你生成迁移模块，以受控、非阻塞的方式在大表上创建、删除或重命名数据库索引。这些操作使用 `Explorer.Migrator.HeavyDbIndexOperation` 行为，并通过 `Explorer.Migrator.MigrationStatus` 进行跟踪。

**此技能生成内容：**
1. 迁移模块文件（create/delete/rename），位于 `apps/explorer/lib/explorer/migrator/heavy_db_index_operation/`
2. 更新 `apps/explorer/lib/explorer/chain/cache/background_migrations.ex`：
   - 用于跟踪完成状态的缓存键
   - 模块别名
   - 缓存填充的 fallback 处理程序

## 何时使用

- 在大表（logs、internal_transactions、token_transfers、addresses、transactions、blocks 等）上创建新索引时
- 作为 schema 优化的一部分删除现有索引时
- 重命名索引时（通常是 create → delete → rename 工作流的最后一步）
- 当索引操作可能需要大量时间且应在后台运行时
- 当你需要跟踪索引创建/删除/重命名的进度时
- 当索引操作需要依赖于其他已完成的迁移时
- 当你想要在 PostgreSQL 上执行 CONCURRENT 索引操作时

## 模块结构

每个 heavy index 操作模块必须实现 `Explorer.Migrator.HeavyDbIndexOperation` 行为，并包含以下回调：

### 必需回调

1. **`migration_name/0`** - 从 `operation_type` 和 `index_name` 自动生成。格式：`heavy_indexes_{operation_type}_{lowercase_index_name}`
2. **`table_name/0`** - 返回表原子（`:logs`、`:internal_transactions`、`:addresses` 等）
3. **`operation_type/0`** - 返回 `:create`、`:drop` 或 `:rename`
4. **`index_name/0`** - 以字符串形式返回索引名称（对于重命名，返回最终/新索引名称）
5. **`dependent_from_migrations/0`** - 返回此迁移所依赖的迁移名称列表（或 `[]`）
6. **`db_index_operation/0`** - 执行实际的索引创建/删除/重命名
7. **`check_db_index_operation_progress/0`** - 检查操作进度
8. **`db_index_operation_status/0`** - 返回操作状态
9. **`restart_db_index_operation/0`** - 在需要时重启操作
10. **`running_other_heavy_migration_exists?/1`** - 检查冲突的迁移
11. **`update_cache/0`** - 迁移完成时更新 BackgroundMigrations 缓存

## 索引定义方法

### 方法 1：使用 `@table_columns`（简单索引）

用于一个或多个列的简单索引：

```elixir
@table_columns ["address_hash", "block_number DESC", "index DESC"]

@impl HeavyDbIndexOperation
def db_index_operation do
  HeavyDbIndexOperationHelper.create_db_index(@index_name, @table_name, @table_columns)
end
```

**何时使用：**
- 简单的多列索引
- 不需要 WHERE 子句
- 可接受标准列顺序
- 最常见的用例

### 方法 2：使用 `@query_string`（复杂索引）

用于更复杂的索引定义：

```elixir
@query_string """
CREATE INDEX #{HeavyDbIndexOperationHelper.add_concurrently_flag?()} IF NOT EXISTS "#{@index_name}"
ON #{@table_name} ((1))
WHERE verified = true;
"""

@impl HeavyDbIndexOperation
def db_index_operation do
  HeavyDbIndexOperationHelper.create_db_index(@query_string)
end
```

**何时使用：**
- 部分索引（带 WHERE 子句）
- 表达式索引（例如，用于存在性检查的 `((1))`）
- 自定义索引类型（GIN、GIST 等）
- 对 SQL 进行细粒度控制

## 命名约定

### 模块名称

- **创建**：`CreateTableNameColumnNameIndex`
  - 示例：`CreateLogsAddressHashBlockNumberDescIndexDescIndex`
  - 示例：`CreateAddressesVerifiedIndex`

- **删除**：`DropTableNameIndexName`
  - 示例：`DropInternalTransactionsCreatedContractAddressHashPartialIndex`
  - 示例：`DropLogsAddressHashIndex`

- **重命名**：`RenameOldIndexNameToNewIndexName` 或 `RenameTableNameIndexDescriptor`
  - 示例：`RenameTransactions2ndCreatedContractAddressHashWithPendingIndexA`

### 索引名称

- 遵循 PostgreSQL 命名：`table_name_column_name_suffix_index`
- 对于部分索引：在名称中包含 `_partial`
- 对于降序列：在名称中包含 `_desc`
- 示例：
  - `logs_address_hash_block_number_DESC_index_DESC_index`
  - `addresses_verified_index`
  - `internal_transactions_created_contract_address_hash_partial_index`

### 文件名称

- 将模块名称转换为 snake_case
- 示例：`CreateLogsAddressHashIndex` → `create_logs_address_hash_index.ex`

## 通过 `dependent_from_migrations/0` 设置依赖

指定必须在此迁移之前完成的迁移：

```elixir
# 无依赖
@impl HeavyDbIndexOperation
def dependent_from_migrations, do: []

# 依赖于另一个迁移
alias Explorer.Migrator.EmptyInternalTransactionsData

@impl HeavyDbIndexOperation
def dependent_from_migrations do
  [EmptyInternalTransactionsData.migration_name()]
end

# 多个依赖
alias Explorer.Migrator.HeavyDbIndexOperation.{
  DropLogsIndexIndex,
  DropLogsBlockNumberAscIndexAscIndex
}

@impl HeavyDbIndexOperation
def dependent_from_migrations do
  [
    DropLogsIndexIndex.migration_name(),
    DropLogsBlockNumberAscIndexAscIndex.migration_name()
  ]
end
```

## 完整示例：创建索引

```elixir
defmodule Explorer.Migrator.HeavyDbIndexOperation.CreateLogsAddressHashBlockNumberDescIndexDescIndex do
  @moduledoc """
  在 `logs` 表上为 (`address_hash`, `block_number DESC`, `index DESC`) 列创建 B-tree 索引 `logs_address_hash_block_number_DESC_index_DESC_index`。
  """

  use Explorer.Migrator.HeavyDbIndexOperation

  require Logger

  alias Explorer.Chain.Cache.BackgroundMigrations
  alias Explorer.Migrator.{HeavyDbIndexOperation, MigrationStatus}
  alias Explorer.Migrator.HeavyDbIndexOperation.Helper, as: HeavyDbIndexOperationHelper

  @table_name :logs
  @index_name "logs_address_hash_block_number_DESC_index_DESC_index"
  @operation_type :create
  @table_columns ["address_hash", "block_number DESC", "index DESC"]

  @impl HeavyDbIndexOperation
  def table_name, do: @table_name

  @impl HeavyDbIndexOperation
  def operation_type, do: @operation_type

  @impl HeavyDbIndexOperation
  def index_name, do: @index_name

  @impl HeavyDbIndexOperation
  def dependent_from_migrations, do: []

  @impl HeavyDbIndexOperation
  def db_index_operation do
    HeavyDbIndexOperationHelper.create_db_index(@index_name, @table_name, @table_columns)
  end

  @impl HeavyDbIndexOperation
  def check_db_index_operation_progress do
    operation = HeavyDbIndexOperationHelper.create_index_query_string(@index_name, @table_name, @table_columns)
    HeavyDbIndexOperationHelper.check_db_index_operation_progress(@index_name, operation)
  end

  @impl HeavyDbIndexOperation
  def db_index_operation_status do
    HeavyDbIndexOperationHelper.db_index_creation_status(@index_name)
  end

  @impl HeavyDbIndexOperation
  def restart_db_index_operation do
    HeavyDbIndexOperationHelper.safely_drop_db_index(@index_name)
  end

  @impl HeavyDbIndexOperation
  def running_other_heavy_migration_exists?(migration_name) do
    MigrationStatus.running_other_heavy_migration_for_table_exists?(@table_name, migration_name)
  end

  @impl HeavyDbIndexOperation
  def update_cache do
    BackgroundMigrations.set_heavy_indexes_create_logs_address_hash_block_number_desc_index_desc_index_finished(
      true
    )
  end
end
```

## 完整示例：删除索引

```elixir
defmodule Explorer.Migrator.HeavyDbIndexOperation.DropInternalTransactionsCreatedContractAddressHashPartialIndex do
  @moduledoc """
  删除 internal_transactions(created_contract_address_hash, block_number DESC, transaction_index DESC, index DESC) 上的索引 "internal_transactions_created_contract_address_hash_partial_index"。
  """

  use Explorer.Migrator.HeavyDbIndexOperation

  alias Explorer.Chain.Cache.BackgroundMigrations
  alias Explorer.Migrator.{EmptyInternalTransactionsData, HeavyDbIndexOperation, MigrationStatus}
  alias Explorer.Migrator.HeavyDbIndexOperation.Helper, as: HeavyDbIndexOperationHelper

  @table_name :internal_transactions
  @index_name "internal_transactions_created_contract_address_hash_partial_index"
  @operation_type :drop

  @impl HeavyDbIndexOperation
  def table_name, do: @table_name

  @impl HeavyDbIndexOperation
  def operation_type, do: @operation_type

  @impl HeavyDbIndexOperation
  def index_name, do: @index_name

  @impl HeavyDbIndexOperation
  def dependent_from_migrations do
    [EmptyInternalTransactionsData.migration_name()]
  end

  @impl HeavyDbIndexOperation
  def db_index_operation do
    HeavyDbIndexOperationHelper.safely_drop_db_index(@index_name)
  end

  @impl HeavyDbIndexOperation
  def check_db_index_operation_progress do
    operation = HeavyDbIndexOperationHelper.drop_index_query_string(@index_name)
    HeavyDbIndexOperationHelper.check_db_index_operation_progress(@index_name, operation)
  end

  @impl HeavyDbIndexOperation
  def db_index_operation_status do
    HeavyDbIndexOperationHelper.db_index_dropping_status(@index_name)
  end

  @impl HeavyDbIndexOperation
  def restart_db_index_operation do
    HeavyDbIndexOperationHelper.safely_drop_db_index(@index_name)
  end

  @impl HeavyDbIndexOperation
  def running_other_heavy_migration_exists?(migration_name) do
    MigrationStatus.running_other_heavy_migration_for_table_exists?(@table_name, migration_name)
  end

  @impl HeavyDbIndexOperation
  def update_cache do
    BackgroundMigrations.set_heavy_indexes_drop_internal_transactions_created_contract_address_hash_partial_index_finished(
      true
    )
  end
end
```

## 支持的表名称

`@table_name` 的有效值（来自行为类型规范）：

- `:addresses`
- `:address_coin_balances`
- `:address_current_token_balances`
- `:address_token_balances`
- `:blocks`
- `:internal_transactions`
- `:logs`
- `:token_transfers`
- `:tokens`
- `:transactions`

## 文件位置

所有生成的模块必须放在：

```
apps/explorer/lib/explorer/migrator/heavy_db_index_operation/
```

## 必需别名

要包含的标准别名：

```elixir
alias Explorer.Chain.Cache.BackgroundMigrations
alias Explorer.Migrator.{HeavyDbIndexOperation, MigrationStatus}
alias Explorer.Migrator.HeavyDbIndexOperation.Helper, as: HeavyDbIndexOperationHelper
```

对于依赖项，添加特定别名：

```elixir
alias Explorer.Migrator.EmptyInternalTransactionsData
```

## 可用辅助函数

### 用于索引创建：

- `HeavyDbIndexOperationHelper.create_db_index/1` - 使用查询字符串
- `HeavyDbIndexOperationHelper.create_db_index/3` - 使用索引名称、表、列
- `HeavyDbIndexOperationHelper.create_index_query_string/3` - 生成查询字符串
- `HeavyDbIndexOperationHelper.db_index_creation_status/1` - 检查创建状态
- `HeavyDbIndexOperationHelper.add_concurrently_flag?/0` - 用于 CONCURRENT 关键字

### 用于索引删除：

- `HeavyDbIndexOperationHelper.safely_drop_db_index/1` - 带安全检查的删除
- `HeavyDbIndexOperationHelper.drop_index_query_string/1` - 生成删除查询
- `HeavyDbIndexOperationHelper.db_index_dropping_status/1` - 检查删除状态

### 通用：

- `HeavyDbIndexOperationHelper.check_db_index_operation_progress/2` - 监控进度

## 缓存失效

删除索引时，你可能需要按模块文档字符串中所示使缓存失效：

```elixir
@impl HeavyDbIndexOperation
def restart_db_index_operation do
  HeavyDbIndexOperationHelper.safely_drop_db_index(@index_name)
  BackgroundMigrations.invalidate_cache(__MODULE__.migration_name())
end
```

## 完整示例：重命名索引

对于重命名操作（通常用于 create + delete 之后交换索引）：

```elixir
defmodule Explorer.Migrator.HeavyDbIndexOperation.RenameTransactions2ndCreatedContractAddressHashWithPendingIndexA do
  @moduledoc """
  将索引 "transactions_2nd_created_contract_address_hash_with_pending_index_a"
  重命名为 "transactions_created_contract_address_hash_with_pending_index_a"。
  """

  use Explorer.Migrator.HeavyDbIndexOperation

  require Logger

  alias Explorer.Chain.Cache.BackgroundMigrations
  alias Explorer.Migrator.{HeavyDbIndexOperation, MigrationStatus}
  alias Explorer.Migrator.HeavyDbIndexOperation.Helper, as: HeavyDbIndexOperationHelper
  alias Explorer.Migrator.HeavyDbIndexOperation.DropTransactionsCreatedContractAddressHashWithPendingIndexA
  alias Explorer.Repo

  @table_name :transactions
  @old_index_name "transactions_2nd_created_contract_address_hash_with_pending_index_a"
  @new_index_name "transactions_created_contract_address_hash_with_pending_index_a"
  @operation_type :rename

  # 注意：migration_name 将是：
  # "heavy_indexes_rename_transactions_created_contract_address_hash_with_pending_index_a"

  @impl HeavyDbIndexOperation
  def table_name, do: @table_name

  @impl HeavyDbIndexOperation
  def operation_type, do: @operation_type

  @impl HeavyDbIndexOperation
  def index_name, do: @new_index_name

  @impl HeavyDbIndexOperation
  def dependent_from_migrations do
    [DropTransactionsCreatedContractAddressHashWithPendingIndexA.migration_name()]
  end

  @impl HeavyDbIndexOperation
  # sobelow_skip ["SQL"]
  def db_index_operation do
    case Repo.query(rename_index_query_string(), [], timeout: :infinity) do
      {:ok, _} ->
        :ok

      {:error, error} ->
        Logger.error("Failed to rename index from #{@old_index_name} to #{@new_index_name}: #{inspect(error)}")
        :error
    end
  end

  @impl HeavyDbIndexOperation
  def check_db_index_operation_progress do
    HeavyDbIndexOperationHelper.check_db_index_operation_progress(@new_index_name, rename_index_query_string())
  end

  @impl HeavyDbIndexOperation
  def db_index_operation_status do
    old_index_status = HeavyDbIndexOperationHelper.db_index_exists_and_valid?(@old_index_name)
    new_index_status = HeavyDbIndexOperationHelper.db_index_exists_and_valid?(@new_index_name)

    cond do
      # 重命名完成：旧索引不存在，新索引存在且有效
      old_index_status == %{exists?: false, valid?: nil} and new_index_status == %{exists?: true, valid?: true} ->
        :completed

      # 重命名未开始：旧索引存在，新索引不存在
      old_index_status == %{exists?: true, valid?: true} and new_index_status == %{exists?: false, valid?: nil} ->
        :not_initialized

      # 未知状态
      true ->
        :unknown
    end
  end

  @impl HeavyDbIndexOperation
  def restart_db_index_operation do
    # 要重启，我们需要重命名回旧名称
    case Repo.query(reverse_rename_index_query_string(), [], timeout: :infinity) do
      {:ok, _} ->
        :ok

      {:error, error} ->
        Logger.error("Failed to reverse rename index from #{@new_index_name} to #{@old_index_name}: #{inspect(error)}")
        :error
    end
  end

  @impl HeavyDbIndexOperation
  def running_other_heavy_migration_exists?(migration_name) do
    MigrationStatus.running_other_heavy_migration_for_table_exists?(@table_name, migration_name)
  end

  @impl HeavyDbIndexOperation
  def update_cache do
    BackgroundMigrations.set_heavy_indexes_rename_transactions_created_contract_address_hash_with_pending_index_a_finished(
      true
    )
  end

  defp rename_index_query_string do
    "ALTER INDEX #{@old_index_name} RENAME TO #{@new_index_name};"
  end

  defp reverse_rename_index_query_string do
    "ALTER INDEX #{@new_index_name} RENAME TO #{@old_index_name};"
  end
end
```

**何时使用重命名操作：**
- 创建新索引并删除旧索引之后
- 将临时索引名称与永久索引名称交换
- 作为索引替换的 create → delete → rename 工作流的一部分

**重命名操作的重要说明：**
- 使用 `@operation_type :rename`（不是 `:create`）
- `index_name/0` 应返回 **新**（最终）索引名称
- Migration name 将是 `heavy_indexes_rename_{new_index_name_lowercase}`
- 示例：对于 `index_name` = "transactions_created_contract_address_hash_with_pending_index_a"，migration name 是 "heavy_indexes_rename_transactions_created_contract_address_hash_with_pending_index_a"

## 更新 BackgroundMigrations 缓存

创建迁移模块后，你必须更新 `apps/explorer/lib/explorer/chain/cache/background_migrations.ex` 中的缓存跟踪：

### 步骤 1：添加缓存键

在模块顶部为每个新迁移添加键：

```elixir
use Explorer.Chain.MapCache,
  name: :background_migrations_status,
  # ... 现有键 ...
  key: :heavy_indexes_create_transactions_2nd_created_contract_address_hash_with_pending_index_a_finished,
  key: :heavy_indexes_drop_transactions_created_contract_address_hash_with_pending_index_a_finished,
  key: :heavy_indexes_rename_transactions_created_contract_address_hash_with_pending_index_a_finished
```

### 步骤 2：添加模块别名

在 `HeavyDbIndexOperation` 别名块中添加别名：

```elixir
alias Explorer.Migrator.HeavyDbIndexOperation.{
  # ... 现有别名 ...
  CreateTransactions2ndCreatedContractAddressHashWithPendingIndexA,
  DropTransactionsCreatedContractAddressHashWithPendingIndexA,
  RenameTransactions2ndCreatedContractAddressHashWithPendingIndexA
}
```

### 步骤 3：添加 Fallback 处理程序

为每个迁移添加 `handle_fallback/1` 函数：

```elixir
defp handle_fallback(:heavy_indexes_create_transactions_2nd_created_contract_address_hash_with_pending_index_a_finished) do
  set_and_return_migration_status(
    CreateTransactions2ndCreatedContractAddressHashWithPendingIndexA,
    &set_heavy_indexes_create_transactions_2nd_created_contract_address_hash_with_pending_index_a_finished/1
  )
end

defp handle_fallback(:heavy_indexes_drop_transactions_created_contract_address_hash_with_pending_index_a_finished) do
  set_and_return_migration_status(
    DropTransactionsCreatedContractAddressHashWithPendingIndexA,
    &set_heavy_indexes_drop_transactions_created_contract_address_hash_with_pending_index_a_finished/1
  )
end

defp handle_fallback(:heavy_indexes_rename_transactions_created_contract_address_hash_with_pending_index_a_finished) do
  set_and_return_migration_status(
    RenameTransactions2ndCreatedContractAddressHashWithPendingIndexA,
    &set_heavy_indexes_rename_transactions_created_contract_address_hash_with_pending_index_a_finished/1
  )
end
```

**缓存键命名约定：**
- 格式：`heavy_indexes_{operation}_{snake_case_index_name}_finished`
- 操作：`create`、`drop`、`rename` 等
- 始终以 `_finished` 结尾

### 步骤 4：添加到应用 Supervisor

将每个迁移模块添加到 `apps/explorer/lib/explorer/application.ex` 中的应用 supervisor：

找到包含其他 heavy DB index 操作的节并添加：

```elixir
configure_mode_dependent_process(
  Explorer.Migrator.HeavyDbIndexOperation.CreateTransactions2ndCreatedContractAddressHashWithPendingIndexA,
  :indexer
),
configure_mode_dependent_process(
  Explorer.Migrator.HeavyDbIndexOperation.DropTransactionsCreatedContractAddressHashWithPendingIndexA,
  :indexer
),
configure_mode_dependent_process(
  Explorer.Migrator.HeavyDbIndexOperation.RenameTransactions2ndCreatedContractAddressHashWithPendingIndexA,
  :indexer
),
```

**重要：** 必须添加这些条目才能在应用程序启动期间启动迁移进程。

## 更新缓存实现

每个迁移模块必须实现 `update_cache/0`：

```elixir
@impl HeavyDbIndexOperation
def update_cache do
  BackgroundMigrations.set_heavy_indexes_create_my_index_finished(true)
end
```

setter 函数名称遵循：`set_heavy_indexes_{operation}_{index_name}_finished/1`

## 新模块检查清单

- [ ] 模块名称遵循 `Create*/Drop*/Rename*` 约定
- [ ] 文件名为模块名称的 snake_case 版本
- [ ] `@moduledoc` 描述索引及其列
- [ ] `use Explorer.Migrator.HeavyDbIndexOperation` 在模块顶部附近声明
- [ ] 所有必需回调已实现
- [ ] `@table_name`、`@index_name`、`@operation_type` 模块属性已定义
- [ ] 索引定义使用 `@table_columns` 或 `@query_string`（或重命名的自定义）
- [ ] 通过 `dependent_from_migrations/0` 指定依赖项
- [ ] 在模块顶部添加适当的别名
- [ ] 文件保存到 `apps/explorer/lib/explorer/migrator/heavy_db_index_operation/`
- [ ] `update_cache/0` 使用正确的 setter 名称实现
- [ ] **BackgroundMigrations 缓存已更新**，包含键、别名和 fallback 处理程序
- [ ] **Application.ex 已更新**，包含 `configure_mode_dependent_process` 条目

## 常见陷阱

❌ **表名称不正确** - 必须是支持的原子之一
❌ **缺少依赖项** - 如果索引依赖于其他迁移，请指定它们
❌ **错误的辅助函数** - 对 `:create` 使用创建辅助函数，对 `:drop` 使用删除辅助函数
❌ **命名不一致** - 索引名称应在语义上与模块名称匹配
❌ **缺少 CONCURRENT** - 在查询字符串中使用 `add_concurrently_flag?()`
❌ **无进度跟踪** - 始终实现 `check_db_index_operation_progress/0`
❌ **忘记缓存更新** - 必须更新 BackgroundMigrations 缓存模块
❌ **缺少 update_cache/0** - 每个模块必须实现此回调

## 索引替换工作流（创建 → 删除 → 重命名）

用新版本替换现有索引时（例如，添加 WHERE 子句）：

1. **创建** 具有临时名称的新索引（例如，`_2nd_` 前缀）
   - 依赖于：表上的最新 heavy DB 操作
2. **删除** 旧索引
   - 依赖于：创建操作完成
3. **重命名** 新索引为旧索引名称
   - 依赖于：删除操作完成

这确保零停机时间 —— 旧索引保持可用直到新索引准备就绪。

## 参考

- 行为定义：[apps/explorer/lib/explorer/migrator/heavy_db_index_operation.ex](../../../apps/explorer/lib/explorer/migrator/heavy_db_index_operation.ex)
- 自述文件：[apps/explorer/lib/explorer/migrator/heavy_db_index_operation/README.md](../../../apps/explorer/lib/explorer/migrator/heavy_db_index_operation/README.md)
- 辅助模块：[apps/explorer/lib/explorer/migrator/heavy_db_index_operation/helper.ex](../../../apps/explorer/lib/explorer/migrator/heavy_db_index_operation/helper.ex)
