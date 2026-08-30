---
name: write-script-mysql
description: 将脚本放置在文件夹中。编写完成后，告知用户可以运行：
---

## CLI 命令

将脚本放置在文件夹中。编写完成后，告知用户可以运行：
- `wmill script generate-metadata` - 生成 .script.yaml 和 .lock 文件
- `wmill sync push` - 部署到 Windmill

不要自己运行这些命令。相反，应告知用户他们应该运行这些命令。

使用 `wmill resource-type list --schema` 来发现可用的资源类型。

# MySQL

参数使用 `?` 占位符。

通过在语句前添加注释来命名参数：

```sql
-- ? name1 (text)
-- ? name2 (int) = 0
SELECT * FROM users WHERE name = ? AND age > ?;
```
