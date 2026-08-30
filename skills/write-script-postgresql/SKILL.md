---
name: write-script-postgresql
description: 编写 PostgreSQL 查询时使用。
---

## CLI 命令

将脚本放置在文件夹中。编写完成后，告知用户可以运行：
- `wmill script generate-metadata` - 生成 .script.yaml 和 .lock 文件
- `wmill sync push` - 部署到 Windmill

不要自己运行这些命令。相反，应告知用户他们应该运行这些命令。

使用 `wmill resource-type list --schema` 来发现可用的资源类型。

# PostgreSQL

参数在语句中直接使用 `$1::{type}`、`$2::{type}` 等获取。

通过在脚本开头添加注释来命名参数（不指定类型）：

```sql
-- $1 name1
-- $2 name2 = default_value
SELECT * FROM users WHERE name = $1::TEXT AND age > $2::INT;
```
