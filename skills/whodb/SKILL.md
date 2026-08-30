---
name: whodb
description: 数据库操作，包括查询、模式探索和分析。在涉及 PostgreSQL、MySQL、MariaDB、SQLite、MongoDB、Redis、Elasticsearch 或 ClickHouse 数据库的任务时激活。
---

# WhoDB 数据库助手

你可以访问 WhoDB 进行数据库操作。使用这些工具和命令帮助用户处理数据库任务。

## MCP 工具（首选）

当 WhoDB MCP 服务器可用时，直接使用这些工具：

### whodb_connections
列出所有可用的数据库连接。
```
无需参数。
返回：带有类型和源（env/saved）的连接名称列表。
```

### whodb_query
对数据库执行 SQL 查询。
```
参数：
- connection: 连接名称（如果只有一个连接则可选）
- query: 要执行的 SQL 查询

示例：whodb_query(connection="mydb", query="SELECT * FROM users LIMIT 10")
```

### whodb_schemas
列出数据库中的所有模式。
```
参数：
- connection: 连接名称（如果只有一个连接则可选）

示例：whodb_schemas(connection="mydb")
```

### whodb_tables
列出模式中的所有表。
```
参数：
- connection: 连接名称（如果只有一个连接则可选）
- schema: 模式名称（可选，如果未指定则使用默认值）

示例：whodb_tables(connection="mydb", schema="public")
```

### whodb_columns
描述表中的列。
```
参数：
- connection: 连接名称（如果只有一个连接则可选）
- table: 表名（必需）
- schema: 模式名称（可选）

示例：whodb_columns(connection="mydb", table="users")
```

## CLI 命令（后备）

如果 MCP 工具不可用，通过 Bash 直接使用 CLI：

### 查询执行
```bash
whodb-cli query "SELECT * FROM users LIMIT 10" --connection mydb --format json
```

### 模式发现
```bash
# 列出模式
whodb-cli schemas --connection mydb --format json

# 列出表
whodb-cli tables --connection mydb --schema public --format json

# 描述列
whodb-cli columns --connection mydb --table users --format json
```

### 连接管理
```bash
# 列出连接
whodb-cli connections list --format json

# 测试连接
whodb-cli connections test mydb

# 添加新连接（交互式）
whodb-cli connections add --name mydb --type Postgres --host localhost --database mydb
```

### 数据导出
```bash
# 导出到 CSV
whodb-cli export --connection mydb --table users --output users.csv

# 导出查询结果
whodb-cli export --connection mydb --query "SELECT * FROM orders" --output orders.xlsx
```

## 工作流示例

### 探索新数据库
1. 列出连接：`whodb_connections`
2. 列出模式：`whodb_schemas(connection="name")`
3. 列出表：`whodb_tables(connection="name", schema="public")`
4. 描述表：`whodb_columns(connection="name", table="users")`
5. 示例数据：`whodb_query(connection="name", query="SELECT * FROM users LIMIT 5")`

### 回答数据问题
1. 首先理解模式 - 检查表结构
2. 编写带有适当过滤器的目标查询
3. 始终对探索性查询使用 LIMIT
4. 以清晰、可读的格式呈现结果

## 最佳实践

- **始终在编写查询之前探索模式**
- **对探索性查询使用 LIMIT** 以避免压倒性输出
- **优先选择特定列** 而不是 SELECT * 以提高清晰度
- **通过 whodb_columns 检查外键** 以了解关系
- **以编程方式解析输出时使用 JSON 格式**（--format json）
- **绝不暴露凭证** - 使用连接名称，而不是连接字符串
