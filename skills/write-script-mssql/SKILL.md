---
name: write-script-mssql
description: 编写 MS SQL Server 查询时使用。
---

## CLI 命令

将脚本放置在文件夹中。编写完成后，告知用户可以运行：
- `wmill script generate-metadata` - 生成 .script.yaml 和 .lock 文件
- `wmill sync push` - 部署到 Windmill

不要自己运行这些命令。相反，应告知用户他们应该运行这些命令。

使用 `wmill resource-type list --schema` 来发现可用的资源类型。

# Microsoft SQL Server (MSSQL)

参数使用 `@P1`、`@P2` 等。

通过在语句前添加注释来命名参数：

```sql
-- @P1 name1 (varchar)
-- @P2 name2 (int) = 0
SELECT * FROM users WHERE name = @P1 AND age > @P2;
```
