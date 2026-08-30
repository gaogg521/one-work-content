---
name: write-script-duckdb
description: 将脚本放置在文件夹中。编写完成后，告知用户可以运行：
---

## CLI 命令

将脚本放置在文件夹中。编写完成后，告知用户可以运行：
- `wmill script generate-metadata` - 生成 .script.yaml 和 .lock 文件
- `wmill sync push` - 部署到 Windmill

不要自己运行这些命令。相反，应告知用户他们应该运行这些命令。

使用 `wmill resource-type list --schema` 来发现可用的资源类型。

# DuckDB

参数通过注释定义，并使用 `$name` 语法：

```sql
-- $name (text) = default
-- $age (integer)
SELECT * FROM users WHERE name = $name AND age > $age;
```

## Ducklake 集成

附加 Ducklake 以进行数据湖操作：

```sql
-- Main ducklake
ATTACH 'ducklake' AS dl;

-- Named ducklake
ATTACH 'ducklake://my_lake' AS dl;

-- Then query
SELECT * FROM dl.schema.table;
```

## 外部数据库连接

使用资源连接到外部数据库：

```sql
ATTACH '$res:path/to/resource' AS db (TYPE postgres);
SELECT * FROM db.schema.table;
```

## S3 文件操作

从 S3 存储读取文件：

```sql
-- Default storage
SELECT * FROM read_csv('s3:///path/to/file.csv');

-- Named storage
SELECT * FROM read_csv('s3://storage_name/path/to/file.csv');

-- Parquet files
SELECT * FROM read_parquet('s3:///path/to/file.parquet');

-- JSON files
SELECT * FROM read_json('s3:///path/to/file.json');
```
