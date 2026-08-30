---
name: type-mapping-encyclopedia
description: RDBMS 到 RDBMS 数据类型映射表、RDBMS 到 NoSQL 转换模式、字符集/排序规则转换和特殊类型处理指南。对于涉及'类型映射'、'数据类型转换'、'MySQL PostgreSQL 转换'、'Oracle 迁移'、'字符集转换'、'排序规则'、'AUTO_INCREMENT 序列'、'JSON 类型'等的请求使用此技能。增强 schema-mapper 的类型转换能力。注意：ETL 脚本编写和验证查询超出本技能范围。
---

# 类型映射百科全书 —— 数据类型映射参考

RDBMS 到 RDBMS 数据类型映射、特殊类型转换和字符集处理的综合参考。

## MySQL 到 PostgreSQL 映射

| MySQL | PostgreSQL | 说明 |
|-------|-----------|-------|
| TINYINT | SMALLINT | TINYINT UNSIGNED -> SMALLINT |
| INT | INTEGER | |
| INT UNSIGNED | BIGINT | PostgreSQL 无 UNSIGNED |
| BIGINT | BIGINT | |
| FLOAT | REAL | |
| DOUBLE | DOUBLE PRECISION | |
| DECIMAL(M,N) | NUMERIC(M,N) | 相同 |
| VARCHAR(N) | VARCHAR(N) | |
| CHAR(N) | CHAR(N) | |
| TEXT | TEXT | |
| MEDIUMTEXT | TEXT | PostgreSQL TEXT 无大小限制 |
| LONGTEXT | TEXT | |
| BLOB | BYTEA | |
| LONGBLOB | BYTEA | 或使用大对象 |
| DATE | DATE | |
| DATETIME | TIMESTAMP | MySQL：非 UTC；PG：时区选项 |
| TIMESTAMP | TIMESTAMPTZ | MySQL：自动 UTC 转换 |
| TIME | TIME | |
| YEAR | SMALLINT | |
| ENUM('a','b') | VARCHAR + CHECK | 或 CREATE TYPE |
| SET('a','b') | VARCHAR[] | 或规范化 |
| JSON | JSONB | 推荐 JSONB（可索引） |
| BIT(N) | BIT(N) | |
| BOOLEAN | BOOLEAN | MySQL TINYINT(1) -> BOOLEAN |
| AUTO_INCREMENT | GENERATED ALWAYS AS IDENTITY | 或 SERIAL（旧版） |

### 特殊转换模式

```sql
-- MySQL ENUM -> PostgreSQL
-- 方法 1：CHECK 约束
CREATE TABLE orders (
    status VARCHAR(20) CHECK (status IN ('pending', 'paid', 'shipped'))
);

-- 方法 2：自定义类型
CREATE TYPE order_status AS ENUM ('pending', 'paid', 'shipped');
CREATE TABLE orders (status order_status);

-- MySQL AUTO_INCREMENT -> PostgreSQL IDENTITY
-- MySQL：
CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY);
-- PostgreSQL：
CREATE TABLE users (id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY);

-- MySQL ON UPDATE CURRENT_TIMESTAMP -> PostgreSQL 触发器
CREATE OR REPLACE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN NEW.updated_at = CURRENT_TIMESTAMP; RETURN NEW; END;
$$ LANGUAGE plpgsql;
CREATE TRIGGER update_timestamp BEFORE UPDATE ON users
FOR EACH ROW EXECUTE FUNCTION update_modified_column();
```

## Oracle 到 PostgreSQL 映射

| Oracle | PostgreSQL | 说明 |
|--------|-----------|-------|
| NUMBER | NUMERIC | 需要精度规范 |
| NUMBER(N,0) | INTEGER/BIGINT | 根据大小选择 |
| VARCHAR2(N) | VARCHAR(N) | |
| CHAR(N) | CHAR(N) | |
| CLOB | TEXT | |
| BLOB | BYTEA | |
| DATE | TIMESTAMP | Oracle DATE 包含时间！ |
| TIMESTAMP WITH TIME ZONE | TIMESTAMPTZ | |
| RAW | BYTEA | |
| LONG | TEXT | 已弃用 —— 建议迁移 |
| NVARCHAR2 | VARCHAR | PostgreSQL 默认为 UTF-8 |
| ROWID | N/A | 使用 ctid 或自定义键 |
| SEQUENCE | SEQUENCE | 语法相似 |
| SYSDATE | CURRENT_TIMESTAMP | |
| NVL() | COALESCE() | |
| DECODE() | CASE WHEN | |

## RDBMS 到 MongoDB 转换模式

### 反规范化策略

```
关系型：                      文档型：
+- users -+  +- orders -+      {
| id       |  | id       |        _id: ObjectId,
| name     |  | user_id  |->      name: "John Doe",
| email    |  | total    |        email: "john@test.com",
+----------+  | items[]  |        orders: [
               +----------+          { total: 50000,
                                      items: [
                                        { product: "A", qty: 2 }
                                      ]
                                    }
                                  ]
                                }
```

### 嵌入与引用决策

| 标准 | 嵌入（嵌套） | 引用（独立） |
|-----------|-------------------|---------------------|
| 关系 | 1:1, 1:Few | 1:Many, M:N |
| 读取模式 | 总是一起查询 | 经常独立查询 |
| 更新频率 | 随父级更新 | 独立更新 |
| 数据大小 | < 16MB（文档限制） | 大小无关 |
| 重复 | 可接受 | 需要去重 |

## 字符集 / 排序规则转换

### MySQL 到 PostgreSQL

```sql
-- MySQL 字符集检查
SHOW VARIABLES LIKE 'character_set%';
SELECT character_set_name, collation_name
FROM information_schema.columns WHERE table_name = 'users';

-- PostgreSQL 排序规则
-- MySQL utf8mb4_general_ci -> PostgreSQL ICU 排序规则
CREATE COLLATION korean_ci (
    provider = icu, locale = 'ko-u-ks-level1', deterministic = false
);

-- 不区分大小写的比较
-- MySQL：utf8mb4_general_ci（默认）
-- PostgreSQL：citext 扩展或 LOWER() 索引
CREATE EXTENSION IF NOT EXISTS citext;
ALTER TABLE users ALTER COLUMN email TYPE citext;
```

### 编码转换说明

| 问题 | 症状 | 解决方案 |
|-------|---------|----------|
| MySQL utf8 (3 字节) | 表情符号损坏 | 迁移前转换为 utf8mb4 |
| Latin1 到 UTF8 | CJK 字符损坏 | 通过二进制中间步骤路由 |
| EUC-KR 到 UTF8 | 稀有 CJK 字符丢失 | 使用映射表 |
| CP949 到 UTF8 | 扩展字符需要验证 | 使用 iconv 预验证 |

## 不可逆转换清单

| 转换 | 不可逆原因 | 缓解措施 |
|-----------|--------------------------|-----------|
| ENUM -> VARCHAR | 允许值约束丢失 | 添加 CHECK 约束 |
| UNSIGNED INT -> INT | 负范围扩展 | 用业务规则加强 |
| DATETIME -> TIMESTAMPTZ | 必须添加时区信息 | 明确声明假定 TZ |
| ROWID -> ctid | 依赖物理位置 | 替换为逻辑键 |
| Oracle DATE -> PG DATE | 时间部分丢失 | 改用 TIMESTAMP |
| CLOB -> TEXT | 行为相同 | -- |

## 类型映射验证查询

```sql
-- 源 (MySQL)
SELECT column_name, data_type, column_type,
       character_maximum_length, numeric_precision, numeric_scale,
       is_nullable, column_default, extra
FROM information_schema.columns
WHERE table_schema = 'mydb' AND table_name = 'orders'
ORDER BY ordinal_position;

-- 目标 (PostgreSQL)
SELECT column_name, data_type, udt_name,
       character_maximum_length, numeric_precision, numeric_scale,
       is_nullable, column_default
FROM information_schema.columns
WHERE table_schema = 'public' AND table_name = 'orders'
ORDER BY ordinal_position;
```
