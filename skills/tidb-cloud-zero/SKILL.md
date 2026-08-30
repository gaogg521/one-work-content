---
name: tidb-cloud-zero
description: 为技术预览版中的代理工作流创建临时的TiDB Cloud Zero数据库。
metadata:
  version: 0.0.0
  homepage: /
---

# 创建一次性TiDB Cloud Zero数据库（技术预览版）

> **浏览器UI说明：** 本网站没有基于浏览器的SQL编辑器。要运行SQL，请使用API + CLI，或登录TiDB Cloud控制台（需要登录）。

使用本指南为代理工作流创建一次性TiDB Cloud Zero数据库。

## 确定性SQL冒烟测试（API + CLI）

```bash
# 1) 配置实例
curl -s -X POST https://zero.tidbapi.com/v1alpha1/instances \
  -H "Content-Type: application/json" \
  -d '{"tag":"sql-smoke-test"}' \
  | tee tidb-zero.json

# 2) 提取连接字符串
jq -r '.instance.connectionString // .connectionString' tidb-zero.json

# 3) 运行SQL查询（将<connectionString>替换为步骤2的输出）
mysql "<connectionString>" -e "SELECT 1 AS health_check, 2 AS example_value;"
```

预期输出包含一行，`health_check=1`和`example_value=2`。

## 端点

- 方法：`POST`
- URL：`https://zero.tidbapi.com/v1alpha1/instances`
- Content-Type：`application/json`
- **技术预览版：** 当前API路径为`/v1alpha1/instances`，此路径可能在后续版本中更改。

## 请求体

- **可选：** `tag`（用于追踪和分组运行的调用者标识符）。

```json
{
  "tag": "support-bot"
}
```

## 快速开始

```bash
curl -X POST https://zero.tidbapi.com/v1alpha1/instances \
  -H "Content-Type: application/json" \
  -d '{
    "tag": "agent-run"
  }'
```

## 响应

API返回连接详情和过期时间。

- **当前响应结构：** 顶层`instance`。
- **`instance.connection`字段：** `host`、`port`、`username`、`password`。
- **使用这些字段：** `instance.connectionString`用于直接URI连接，`instance.expiresAt`用于过期时间。
- **代理说明：** 配置成功后，将实例详情保存到本地文件（例如`tidb-cloud-zero.json`），并提醒用户安全存储该文件，因为它包含敏感凭证。
- **计划更新：** 我们将在后续版本中提供`claimUrl`。用户将能够在`instance.expiresAt`之前登录TiDB Cloud并认领临时数据库，将其转换为正式的TiDB Cloud Starter数据库。

```json
{
  "instance": {
    "connection": {
      "host": "<HOST>",
      "port": 4000,
      "username": "<USERNAME>",
      "password": "<PASSWORD>"
    },
    "connectionString": "mysql://<USERNAME>:<PASSWORD>@<HOST>:4000",
    "expiresAt": "<ISO_TIMESTAMP>"
  }
}
```

## 使用连接字符串

收到响应后，使用`instance.connectionString`通过MySQL兼容客户端或驱动程序连接。

## 引导式快速体验

配置成功后，你应该询问用户：

- **你想让我创建一个示例表并插入演示数据以便立即查询吗？**

如果用户同意，运行如下的小型引导SQL流程：

```sql
CREATE TABLE IF NOT EXISTS quickstart_notes (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(100) NOT NULL,
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO quickstart_notes (title, content) VALUES
  ('welcome', 'TiDB Cloud Zero quickstart row'),
  ('query-demo', 'Run SELECT * FROM quickstart_notes; to verify data');

SELECT * FROM quickstart_notes ORDER BY id;
```

### 通过CLI连接

```bash
mysql --connect-timeout=10 --protocol=TCP -h '<HOST>' -P 4000 -u '<USERNAME>' -p'<PASSWORD>'
```

### 在Node.js中连接（`mysql2`）

```js
import mysql from "mysql2/promise";

const response = await createDatabase(); // 你的API调用结果
const connectionUrl = new URL(response.instance.connectionString);
connectionUrl.pathname = "/<DATABASE>";
connectionUrl.searchParams.set("ssl", JSON.stringify({ rejectUnauthorized: true }));

const connection = await mysql.createConnection(connectionUrl.toString());
const [rows] = await connection.query("SELECT NOW() AS now_time");
console.log(rows);
await connection.end();
```

## 资源

- TiDB SQL技能：https://skills.sh/pingcap/agent-rules/tidb-sql
- PyTiDB技能：https://skills.sh/pingcap/agent-rules/pytidb（使用此技能通过pytidb从Python连接TiDB，定义表，并构建搜索/AI功能。）
- TiDB Cloud文档：https://docs.pingcap.com/tidbcloud/
- TiDB Cloud网站：https://tidbcloud.com/
