---
name: oceanbase-sql-optimization
description: OceanBase 数据库（MySQL 和 Oracle 模式）的 SQL 优化最佳实践。涵盖查询优化、索引使用、执行计划分析、慢查询调优和性能优化技术。在 SQL 优化、查询性能、索引设计、执行计划、慢查询、数据库性能场景下激活。
license: MIT
metadata:
  audience: developers, database-administrators
  domain: database-optimization
  compatibility: mysql-mode, oracle-mode
---

# OceanBase SQL 优化技能

**OceanBase 分布式数据库（MySQL 和 Oracle 模式）SQL 查询优化专家。**

提供编写高效 SQL 查询、设计有效索引、分析执行计划和调优 OceanBase 慢查询的最佳实践。本技能涵盖 MySQL 模式和 Oracle 模式。

---

## 模式特定语法指南

虽然 OceanBase 对 MySQL 和 Oracle 模式使用相同的底层优化器，但存在重要的语法差异：

| 方面 | MySQL 模式 | Oracle 模式 |
|--------|-----------|-------------|
| **EXPLAIN 语法** | `EXPLAIN [INTO table_name] [SET statement_id = string]` | `EXPLAIN [INTO table_name] [SET STATEMENT_ID = 'string']` |
| **系统视图** | `oceanbase.GV$OB_SQL_AUDIT` | `GV$OB_SQL_AUDIT`（同名，不同模式访问） |
| **数据类型** | `BIGINT`, `VARCHAR` | `NUMBER`, `VARCHAR2` |
| **时间函数** | `TIME_TO_USEC()`, `usec_to_time()` | `TO_DATE()`, `TO_CHAR()` |
| **模式访问** | `database.table` | `schema.table` |
| **Limit 语法** | `LIMIT n` | `FETCH FIRST n ROWS ONLY` |
| **字符串连接** | `CONCAT()` | `\|\|` 操作符 |

**优化原则：**
- ✅ **相同**：执行计划操作符、连接算法、分区剪枝逻辑
- ✅ **相同**：索引设计原则、查询重写策略
- ❌ **不同**：语法、系统视图访问、数据类型名称、函数名称

本技能中的示例标有模式指示器：**MySQL 模式：** 或 **Oracle 模式：**

---

## 核心优化原则

### 1. 理解执行计划

**优化前始终分析执行计划：**

**MySQL 模式：**
```sql
obclient [SALES_DB]> EXPLAIN SELECT * FROM order_table WHERE order_date >= '2024-01-01';
```

**Oracle 模式：**
```sql
obclient [SALES_DB]> EXPLAIN SELECT * FROM order_table WHERE order_date >= DATE '2024-01-01';
```

查询结果如下：

```
+------------------------------------------------------------------------------------------------+
| Query Plan                                                                                     |
+------------------------------------------------------------------------------------------------+
| ===========================================                                                    |
| |ID|OPERATOR              |NAME      |EST. ROWS|COST |                                        |
| -------------------------------------------                                                    |
| |0 |TABLE SCAN            |order_table|1000    |1000 |                                        |
| |1 | TABLE GET            |order_table|1000    |1000 |                                        |
+===========================================                                                      |
| Outputs & filters:                                                                             |
| -------------------------------------                                                          |
|   0 - output([order_table.order_id], [order_table.customer_id], [order_table.order_date]),   |
|       filter([order_table.order_date >= '2024-01-01']),                                        |
|       access([order_table.order_id], [order_table.customer_id], [order_table.order_date]),    |
|       partitions(p0)                                                                            |
+------------------------------------------------------------------------------------------------+
```

**关键检查指标：**
- **EST. ROWS**：估计处理的行数（越低越好）
- **EST.TIME(us)**：估计执行时间（微秒，越低越好）
- **OPERATOR**：查找 `TABLE SCAN`（全表扫描，不好）vs `TABLE GET`（索引查找，好）
- **Partitions**：检查分区剪枝是否生效（例如 `partitions(p1)` 表示只访问一个分区）
- **is_index_back**：`false` 表示无索引回表，`true` 表示需要索引回表

### 2. 索引设计最佳实践

**在频繁查询的列上创建索引（两种模式语法相同）：**

```sql
-- ✅ 好：在 WHERE 子句列上创建索引
obclient [SALES_DB]> CREATE INDEX idx_order_date ON order_table(order_date);

-- ✅ 好：多列查询的复合索引（列顺序很重要）
obclient [SALES_DB]> CREATE INDEX idx_customer_date ON order_table(customer_id, order_date);

-- ✅ 好：唯一约束的唯一索引
obclient [SALES_DB]> CREATE UNIQUE INDEX idx_order_number ON order_table(order_number);

-- ✅ 好：使用 STORING 子句的覆盖索引
obclient [SALES_DB]> CREATE INDEX idx_customer_date ON order_table(customer_id, order_date)
                     STORING (total_amount, status);

-- ❌ 不好：在很少查询的列上创建索引
obclient [SALES_DB]> CREATE INDEX idx_notes ON order_table(notes);
```

**OceanBase 索引类型：**
- **UNIQUE 索引**：确保唯一性，快速查找
- **LOCAL 索引**：分区本地索引（分区表的默认）
- **GLOBAL 索引**：跨分区索引
- **函数索引**：表达式上的索引（例如 `CREATE INDEX idx_expr ON t1((c1 + c2))`）

**索引选择规则：**
- 索引 WHERE 子句中使用的列
- 索引 JOIN 条件中使用的列
- 为多列过滤器考虑复合索引（列顺序很重要 - 最具选择性的在前）
- 使用 STORING 子句创建覆盖索引（避免索引回表）
- 避免过度索引（每个索引都会增加写入开销）
- 对于分区表，优先使用 LOCAL 索引，除非跨分区查询很常见

### 3. 查询编写最佳实践

**使用具体的列列表而不是 SELECT *：**

```sql
-- ✅ 好：只选择需要的列
obclient [SALES_DB]> SELECT order_id, customer_id, total_amount
                     FROM order_table
                     WHERE order_date >= '2024-01-01';

-- ❌ 不好：选择所有列
obclient [SALES_DB]> SELECT * FROM order_table WHERE order_date >= '2024-01-01';
```

**使用适当的 WHERE 条件：**

```sql
-- ✅ 好：在 WHERE 中使用索引列
obclient [SALES_DB]> SELECT * FROM order_table WHERE order_id = 12345;

-- ✅ 好：在索引列上使用范围查询
obclient [SALES_DB]> SELECT * FROM order_table
                     WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';

-- ❌ 不好：在索引列上使用函数会阻止索引使用
obclient [SALES_DB]> SELECT * FROM order_table WHERE YEAR(order_date) = 2024;

-- ✅ 好：重写以使用索引
obclient [SALES_DB]> SELECT * FROM order_table
                     WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01';
```

---

## 分区剪枝优化

分区剪枝避免访问不相关的分区，显著提高 SQL 执行效率。始终检查执行计划中的 `partitions` 字段以验证分区剪枝是否生效。

### Hash 分区

**确保分区键在 WHERE 子句中：**

```sql
-- ✅ 好：WHERE 子句中的分区键启用分区剪枝
obclient [SALES_DB]> CREATE TABLE order_table (
                     order_id INT,
                     customer_id INT,
                     order_date DATE
                     ) PARTITION BY HASH(order_id) PARTITIONS 5;

obclient [SALES_DB]> SELECT * FROM order_table WHERE order_id = 12345;
```

查询结果如下：

```
+------------------------------------------------------------------------------------------------+
| Query Plan                                                                                     |
+------------------------------------------------------------------------------------------------+
| =================================================                                              |
| |ID|OPERATOR  |NAME       |EST. ROWS|EST.TIME(us)|                                            |
| -------------------------------------------------                                              |
| |0 |TABLE SCAN|order_table|990      |383         |                                            |
| =================================================                                              |
| Outputs & filters:                                                                             |
| -------------------------------------                                                          |
|   0 - output([order_table.order_id], [order_table.customer_id], [order_table.order_date]),  |
|       filter(nil),                                                                             |
|       access([order_table.order_id], [order_table.customer_id], [order_table.order_date]),   |
|       partitions(p1)  -- 只访问一个分区！                                         |
+------------------------------------------------------------------------------------------------+
```

```sql
-- ❌ 不好：缺少分区键阻止剪枝（扫描所有分区）
obclient [SALES_DB]> SELECT * FROM order_table WHERE customer_id = 1001;
-- 检查执行计划：partitions(p0, p1, p2, p3, p4) - 扫描所有分区
```

### Range 分区

**使用匹配分区边界的范围条件：**

```sql
-- ✅ 好：范围查询匹配分区边界
obclient [SALES_DB]> CREATE TABLE order_table (
                     order_id INT,
                     customer_id INT,
                     order_date DATE
                     ) PARTITION BY RANGE(order_date) (
                     PARTITION p0 VALUES LESS THAN('2024-01-01'),
                     PARTITION p1 VALUES LESS THAN('2024-02-01'),
                     PARTITION p2 VALUES LESS THAN('2024-03-01')
                     );

obclient [SALES_DB]> SELECT * FROM order_table
                     WHERE order_date >= '2024-01-01' AND order_date < '2024-02-01';
```

查询结果如下：

```
+------------------------------------------------------------------------------------------------+
| Query Plan                                                                                     |
+------------------------------------------------------------------------------------------------+
| =================================================                                              |
| |ID|OPERATOR  |NAME       |EST. ROWS|EST.TIME(us)|                                            |
| -------------------------------------------------                                              |
| |0 |TABLE SCAN|order_table|1        |46          |                                            |
| =================================================                                              |
| Outputs & filters:                                                                             |
| -------------------------------------                                                          |
|   0 - output([order_table.order_id], [order_table.customer_id], [order_table.order_date]),  |
|       filter([order_table.order_date >= '2024-01-01'], [order_table.order_date < '2024-02-01']),
|       access([order_table.order_id], [order_table.customer_id], [order_table.order_date]),   |
|       partitions(p1)  -- 只访问分区 p1！                                        |
+------------------------------------------------------------------------------------------------+
```

```sql
-- ❌ 不好：函数阻止分区剪枝
obclient [SALES_DB]> SELECT * FROM order_table WHERE MONTH(order_date) = 1;
-- 检查执行计划：partitions(p0, p1, p2) - 扫描所有分区

-- ✅ 好：重写以使用分区剪枝
obclient [SALES_DB]> SELECT * FROM order_table
                     WHERE order_date >= '2024-01-01' AND order_date < '2024-02-01';
```

**分区剪枝规则：**
- 对于 Hash/List 分区：分区键必须在 WHERE 子句中使用等值或 IN 条件
- 对于 Range 分区：使用匹配分区边界的范围条件
- 避免在 WHERE 子句中对分区键使用函数
- 对于表达式作为分区键，表达式必须整体出现在 WHERE 子句中

---

## 连接优化

OceanBase 支持三种连接算法：**Nested Loop Join (NLJ)**、**Hash Join (HJ)** 和 **Merge Join (MJ)**。优化器根据统计信息和成本估算自动选择最佳算法。

### 连接算法

**1. Nested Loop Join (NLJ)**
- 最适合：小外表，带索引的内表
- 使用时机：连接条件在内表上有索引
- 提示：`/*+ USE_NL(table1, table2) */`

```sql
-- ✅ 好：带索引连接列的 NLJ
obclient [SALES_DB]> CREATE INDEX idx_customer_id ON order_table(customer_id);

obclient [SALES_DB]> EXPLAIN SELECT /*+USE_NL(c, o)*/ o.order_id, c.customer_name
                     FROM customer_table c
                     INNER JOIN order_table o ON c.customer_id = o.customer_id
                     WHERE c.customer_type = 'VIP';
```

查询结果如下：

```
+------------------------------------------------------------------------------------------------+
| Query Plan                                                                                     |
+------------------------------------------------------------------------------------------------+
| ===========================================                                                    |
| |ID|OPERATOR        |NAME  |EST. ROWS|EST.TIME(us)|                                           |
| -------------------------------------------                                                    |
| |0 |NESTED-LOOP JOIN|      |990      |37346       |                                           |
| |1 | TABLE SCAN     |c     |999      |669         |                                           |
| |2 | TABLE GET      |o     |1        |36          |  -- 索引查找！                         |
| ===========================================                                                    |
| Outputs & filters:                                                                             |
| -------------------------------------                                                          |
|   0 - output([o.order_id], [c.customer_name]), filter(nil),                                   |
|       conds(nil), nl_params_([c.customer_id])                                                 |
|   1 - output([c.customer_id], [c.customer_name]), filter([c.customer_type = 'VIP']),         |
|       access([c.customer_id], [c.customer_name]), partitions(p0)                             |
|   2 - output([o.order_id]), filter(nil),                                                      |
|       access([o.order_id]), partitions(p0),                                                    |
|       range_key([o.customer_id]), range([? = o.customer_id])  -- 使用了索引！                  |
+------------------------------------------------------------------------------------------------+
```

**2. Hash Join (HJ)**
- 最适合：大数据集，等值连接条件
- 使用时机：两个表都很大且连接条件是等值
- 提示：`/*+ USE_HASH(table1, table2) */`

```sql
-- ✅ 好：大表连接的 Hash Join
obclient [SALES_DB]> EXPLAIN SELECT /*+USE_HASH(o, c)*/ o.order_id, c.customer_name
                     FROM order_table o
                     INNER JOIN customer_table c ON o.customer_id = c.customer_id;
```

查询结果如下：

```
+------------------------------------------------------------------------------------------------+
| Query Plan                                                                                     |
+------------------------------------------------------------------------------------------------+
| =======================================                                                        |
| |ID|OPERATOR   |NAME|EST. ROWS|EST.TIME(us)|                                                   |
| ---------------------------------------                                                        |
| |0 |HASH JOIN  |    |98010000 |66774608    |                                                   |
| |1 | TABLE SCAN|o   |100000   |68478       |                                                   |
| |2 | TABLE SCAN|c   |100000   |68478       |                                                   |
| =======================================                                                        |
| Outputs & filters:                                                                             |
| -------------------------------------                                                          |
|   0 - output([o.order_id], [c.customer_name]), filter(nil),                                    |
|       equal_conds([o.customer_id = c.customer_id]), other_conds(nil)                           |
+------------------------------------------------------------------------------------------------+
```

**3. Merge Join (MJ)**
- 最适合：已排序数据，需要有序结果
- 使用时机：两个输入都已排序或可以高效排序
- 提示：`/*+ USE_MERGE(table1, table2) */`

```sql
-- ✅ 好：数据已排序时使用 Merge Join
obclient [SALES_DB]> EXPLAIN SELECT /*+USE_MERGE(o, c)*/ o.order_id, c.customer_name
                     FROM order_table o
                     INNER JOIN customer_table c ON o.customer_id = c.customer_id
                     ORDER BY o.customer_id;
```

### 连接顺序

**使用 LEADING 提示控制连接顺序：**

```sql
-- ✅ 好：显式控制连接顺序
obclient [SALES_DB]> SELECT /*+LEADING(c, o)*/ o.order_id, c.customer_name
                     FROM customer_table c
                     INNER JOIN order_table o ON c.customer_id = o.customer_id
                     WHERE c.customer_type = 'VIP'
                     AND o.order_date >= '2024-01-01';

-- ❌ 不好：让优化器选择次优顺序
obclient [SALES_DB]> SELECT o.order_id, c.customer_name
                     FROM order_table o
                     INNER JOIN customer_table c ON o.customer_id = c.customer_id
                     WHERE c.customer_type = 'VIP';
```

### 连接列上的索引

**始终为外键列创建索引：**

```sql
-- ✅ 好：连接列上的索引支持高效连接
obclient [SALES_DB]> CREATE INDEX idx_customer_id ON order_table(customer_id);

-- 然后连接查询可以使用索引（NLJ 带 TABLE GET）
obclient [SALES_DB]> SELECT o.*, c.customer_name
                     FROM order_table o
                     INNER JOIN customer_table c ON o.customer_id = c.customer_id;
```

---

## 聚合优化

### 使用适当的聚合函数

```sql
-- ✅ 好：COUNT(*) 已优化
obclient [SALES_DB]> SELECT COUNT(*) FROM order_table WHERE order_date >= '2024-01-01';

-- ✅ 好：需要非 NULL 计数时使用 COUNT(column)
obclient [SALES_DB]> SELECT COUNT(customer_id) FROM order_table;

-- ❌ 不好：大数据集上的 COUNT(DISTINCT) 可能很慢
obclient [SALES_DB]> SELECT COUNT(DISTINCT customer_id) FROM order_table;

-- ✅ 好：对前 N 使用 GROUP BY 与 LIMIT（MySQL）或 FETCH FIRST（Oracle）
-- MySQL 模式：
obclient [SALES_DB]> SELECT customer_id, SUM(total_amount) AS total
                     FROM order_table
                     GROUP BY customer_id
                     ORDER BY total DESC
                     LIMIT 10;

-- Oracle 模式：
obclient [SALES_DB]> SELECT customer_id, SUM(total_amount) AS total
                     FROM order_table
                     GROUP BY customer_id
                     ORDER BY total DESC
                     FETCH FIRST 10 ROWS ONLY;
```

---

## 子查询优化

### 将相关子查询转换为 JOIN

```sql
-- ❌ 不好：相关子查询（为每行执行）
obclient [SALES_DB]> SELECT o.order_id, o.total_amount
                     FROM order_table o
                     WHERE o.total_amount > (
                         SELECT AVG(total_amount)
                         FROM order_table
                         WHERE customer_id = o.customer_id
                     );

-- ✅ 好：转换为 JOIN
obclient [SALES_DB]> SELECT o.order_id, o.total_amount
                     FROM order_table o
                     INNER JOIN (
                         SELECT customer_id, AVG(total_amount) as avg_amount
                         FROM order_table
                         GROUP BY customer_id
                     ) avg_orders ON o.customer_id = avg_orders.customer_id
                     WHERE o.total_amount > avg_orders.avg_amount;
```

### 对大数据集使用 EXISTS 代替 IN

**两种模式语法相同：**

```sql
-- ✅ 好：EXISTS 在第一个匹配处停止
obclient [SALES_DB]> SELECT * FROM order_table o
                     WHERE EXISTS (
                         SELECT 1 FROM customer_table c
                         WHERE c.customer_id = o.customer_id
                         AND c.customer_type = 'VIP'
                     );

-- ⚠️ 可以：IN 工作但可能对大列表较慢
obclient [SALES_DB]> SELECT * FROM order_table
                     WHERE customer_id IN (SELECT customer_id FROM customer_table WHERE customer_type = 'VIP');
```

---

## 慢查询调优

### 启用 SQL 审计

**配置 SQL 审计设置：**

```sql
-- 启用 SQL 审计
obclient> ALTER SYSTEM SET enable_sql_audit = true;

-- 设置 SQL 审计内存百分比（默认：3%，范围：[0,80]）
obclient> SET GLOBAL ob_sql_audit_percentage = 3;
```

### 识别慢查询

**按 CPU 使用率查询 TOP SQL：**

**MySQL 模式：** 使用 `oceanbase.GV$OB_SQL_AUDIT` 与 `TIME_TO_USEC(NOW())`
**Oracle 模式：** 使用 `GV$OB_SQL_AUDIT` 与 `SYSDATE - INTERVAL '30' MINUTE`

```sql
-- 查找最近 30 分钟内按 CPU 使用率的 TOP SQL
obclient> SELECT sql_id, COUNT(*) AS executions, SUM(execute_time) AS tot_cpu_time,
                AVG(execute_time) AS avg_cpu_time,
                SUM(execute_time)/(30*60*1000*1000) AS cpu_cnt, query_sql
         FROM oceanbase.GV$OB_SQL_AUDIT  -- MySQL: 添加 'oceanbase.' 前缀
         WHERE tenant_id = 'mysql001'  -- MySQL: 字符串，Oracle: 数字
           AND request_time BETWEEN (TIME_TO_USEC(NOW())-30*60*1000*1000) AND TIME_TO_USEC(NOW())  -- MySQL
           -- Oracle: BETWEEN (SYSDATE - INTERVAL '30' MINUTE) AND SYSDATE
           AND is_executor_rpc = 0
         GROUP BY sql_id
         HAVING COUNT(*) > 1
         ORDER BY cpu_cnt DESC
         LIMIT 10;  -- MySQL: LIMIT，Oracle: FETCH FIRST 10 ROWS ONLY
```

**按执行时间查询慢查询：**

**MySQL 模式：** 使用 `usec_to_time()` 函数
**Oracle 模式：** 使用 `TO_CHAR()` 函数

```sql
-- 查找执行时间 > 100ms 的查询
obclient> SELECT request_id,
                usec_to_time(request_time) AS request_time,  -- MySQL
                -- TO_CHAR(request_time, 'YYYY-MM-DD HH24:MI:SS.FF') AS request_time,  -- Oracle
                elapsed_time, queue_time, execute_time, flt_trace_id, query_sql
         FROM oceanbase.v$ob_sql_audit  -- MySQL: 添加 'oceanbase.' 前缀
         WHERE elapsed_time > 100000  -- > 100ms（微秒）
         ORDER BY elapsed_time DESC
         LIMIT 10;  -- MySQL: LIMIT，Oracle: FETCH FIRST 10 ROWS ONLY
```

### 分析查询性能

**获取详细执行统计：**

**MySQL 模式：** 使用 `oceanbase.gv$ob_sql_audit`
**Oracle 模式：** 使用 `GV$OB_SQL_AUDIT`（无模式前缀）

```sql
-- 获取特定 SQL 的详细统计
obclient> SELECT sql_id, query_sql, executions, elapsed_time, execute_time,
                queue_time, return_rows, affected_rows, partition_cnt,
                table_scan, is_hit_plan, plan_id
         FROM oceanbase.gv$ob_sql_audit  -- MySQL: 添加 'oceanbase.' 前缀
         WHERE sql_id = 'your_sql_id'
         ORDER BY elapsed_time DESC;
```

**获取慢查询的执行计划：**

**MySQL 模式：** 使用 `oceanbase.GV$OB_PLAN_CACHE_PLAN_STAT`
**Oracle 模式：** 使用 `GV$OB_PLAN_CACHE_PLAN_STAT`（无模式前缀）

```sql
-- 从计划缓存获取执行计划
obclient> SELECT tenant_id, svr_ip, svr_port, sql_id, plan_id,
                last_active_time, first_load_time, outline_data
         FROM oceanbase.GV$OB_PLAN_CACHE_PLAN_STAT  -- MySQL: 添加 'oceanbase.' 前缀
         WHERE tenant_id = 1002 AND sql_id = 'your_sql_id'
           AND svr_ip = 'xxx.xxx.xxx.xxx' AND svr_port = 35046;

-- 获取计划详情
obclient> SELECT operator, name, rows, cost
         FROM oceanbase.GV$OB_PLAN_CACHE_PLAN_EXPLAIN  -- MySQL: 添加 'oceanbase.' 前缀
         WHERE tenant_id = 1002 AND plan_id = 741
           AND svr_ip = 'xxx.xxx.xxx.xxx' AND svr_port = 35046;
```

### 查找当前运行的慢查询

**MySQL 模式：**
```sql
obclient> SELECT user, tenant, sql_id, concat(time, 's') AS time, info,
                svr_ip, svr_port, trace_id
         FROM oceanbase.GV$OB_PROCESSLIST
         WHERE state = 'ACTIVE' ORDER BY time DESC LIMIT 10;
```

**Oracle 模式：**
```sql
obclient> SELECT user_name, tenant_name, sql_id, time || 's' AS time, info,
                svr_ip, svr_port, trace_id
         FROM GV$OB_PROCESSLIST
         WHERE state = 'ACTIVE' ORDER BY time DESC FETCH FIRST 10 ROWS ONLY;
```

---

## 真实优化案例

### 案例：类型转换阻止索引使用

**问题：** SQL 执行时间为 2 秒，CPU 使用率超过服务器资源的 70%。

**根本原因分析：**

1. **识别 TOP SQL：**
   ```sql
   obclient> SELECT sql_id, COUNT(*) AS executions, AVG(execute_time) AS avg_cpu_time
            FROM oceanbase.GV$OB_SQL_AUDIT
            WHERE tenant_id = 'mysql001'
              AND request_time BETWEEN (TIME_TO_USEC(NOW())-30*60*1000*1000) AND TIME_TO_USEC(NOW())
            GROUP BY sql_id
            ORDER BY SUM(execute_time) DESC
            LIMIT 1;
   ```

2. **分析执行计划：**
   ```sql
   obclient> EXPLAIN SELECT ... FROM v_tt01 WHERE COL001 IN (20017476);
   ```

   执行计划显示：
   - `TBL3(IDX_TBL3_COL170)` 索引扫描成本高
   - `range(MIN,MIN ; MAX,MAX)always true` - 索引未用于匹配
   - `filter([20017476 = cast(cast(TBL3.COL170, VARCHAR2(20 BYTE)), NUMBER)])` - 发生类型转换

3. **根本原因：**
   - 视图 `V_TT01` 对 `COL001` 有 `TO_NUMBER` 转换
   - 表 `TBL3.COL170` 是 `VARCHAR2(20)` 但与 `NUMBER` 比较
   - 类型转换阻止了索引使用

**优化步骤：**

1. **移除不必要的 TO_NUMBER 转换：**
   ```sql
   -- 之前：视图有 TO_NUMBER 转换
   CREATE VIEW V_TT01_OLD AS
   SELECT To_number("tt01"."col001") AS "COL001", ...

   -- 之后：移除 TO_NUMBER（COL001 已经是 NUMBER）
   CREATE VIEW V_TT01 AS
   SELECT "tt01"."col001" AS "COL001", ...
   ```
   结果：SQL RT 从 2s 降至 150ms

2. **统一数据类型：**
   ```sql
   -- 之前：TBL3.COL170 是 VARCHAR2(20)
   CREATE TABLE TBL3 (
     COL170 VARCHAR2(20) NOT NULL,
     ...
   );

   -- 之后：改为 NUMBER(20) 以匹配连接条件
   ALTER TABLE TBL3 MODIFY COL170 NUMBER(20) NOT NULL;
   ```
   结果：SQL RT 进一步降至 20ms

3. **分区和表组优化：**
   ```sql
   -- 在连接键上添加 hash 分区
   ALTER TABLE TBL1 PARTITION BY HASH(COL001) PARTITIONS 8;
   ALTER TABLE TBL2 PARTITION BY HASH(COL004) PARTITIONS 8;
   ALTER TABLE TBL3 PARTITION BY HASH(COL170) PARTITIONS 8;
   ALTER TABLE TT01 PARTITION BY HASH(COL001) PARTITIONS 8;

   -- 创建表组以避免跨分区连接
   CREATE TABLEGROUP order_tg PARTITION BY HASH(COL001) PARTITIONS 8;
   ALTER TABLE TBL1 SET TABLEGROUP order_tg;
   ALTER TABLE TBL2 SET TABLEGROUP order_tg;
   ALTER TABLE TBL3 SET TABLEGROUP order_tg;
   ALTER TABLE TT01 SET TABLEGROUP order_tg;
   ```
   结果：SQL RT 进一步降至 4ms

**最终执行计划：** 显示 `PX PARTITION ITERATOR`（分区剪枝生效）和带索引 `INDEX_TBL3_COL170` 的 `TABLE SCAN`（索引高效使用）

**关键经验：**
- 始终检查过滤器中的类型转换执行计划
- 确保连接列数据类型匹配
- 对分布式查询使用分区剪枝和表组
- 移除视图中不必要的类型转换

---

## 常见反模式

### ❌ 避免这些模式

**1. N+1 查询问题：**
```sql
-- ❌ 不好：循环中的多个查询
-- 相反，使用 JOIN 或批量查询
```

**2. 生产环境使用 SELECT *：**
```sql
-- ❌ 不好：SELECT * 返回不必要的数据
SELECT * FROM order_table;

-- ✅ 好：只选择需要的列
SELECT order_id, customer_id, total_amount FROM order_table;
```

**3. 在索引列上使用函数：**
```sql
-- ❌ 不好：阻止索引使用
WHERE UPPER(customer_name) = 'JOHN'

-- ✅ 好：存储规范化数据或使用函数索引
WHERE customer_name = 'JOHN'
```

**4. 隐式类型转换：**
```sql
-- ❌ 不好：字符串到数字转换
WHERE order_id = '12345'

-- ✅ 好：匹配数据类型
WHERE order_id = 12345
```

---

## 性能监控

### 关键监控指标

```sql
-- 检查表统计
obclient [SALES_DB]> ANALYZE TABLE order_table;

-- 查看表大小和行数
obclient [SALES_DB]> SELECT
                         table_name,
                         table_rows,
                         data_length,
                         index_length
                     FROM information_schema.tables
                     WHERE table_schema = 'SALES_DB';

-- 检查索引使用
obclient [SALES_DB]> SHOW INDEX FROM order_table;
```

---

## 优化检查清单

在将 SQL 查询部署到生产环境之前：

- [ ] 已审查执行计划 (EXPLAIN)
- [ ] 已创建适当索引
- [ ] 已启用分区剪枝（如果分区）
- [ ] SELECT 列限制为所需字段
- [ ] WHERE 条件使用索引列
- [ ] JOIN 条件已索引
- [ ] 子查询已优化（优先使用 JOIN）
- [ ] 聚合使用高效函数
- [ ] WHERE 中没有在索引列上使用函数
- [ ] 比较时数据类型匹配
- [ ] 已使用生产级数据量测试查询

---

## 快速参考

### 执行计划操作符

| 操作符 | 含义 | 优化 |
|----------|---------|--------------|
| TABLE SCAN / TABLE FULL SCAN | 全表扫描 | 添加索引或使用分区键 |
| TABLE RANGE SCAN | 表范围扫描 | ✅ 好，但考虑索引 |
| TABLE GET | 通过主键直接行访问 | ✅ 最佳 |
| HASH JOIN | Hash 连接算法 | ✅ 适合大表等值连接 |
| NESTED-LOOP JOIN | 嵌套循环连接 | ✅ 适合小外表带索引内表 |
| MERGE JOIN | Merge 连接算法 | ✅ 适合已排序数据 |
| SORT | 排序操作 | 添加 ORDER BY 索引或使用排序索引 |
| AGGREGATE | 聚合 | 考虑物化视图 |
| PX PARTITION ITERATOR | 并行分区迭代器 | ✅ 好，分区剪枝生效 |
| EXCHANGE OUT DISTR | 分布式交换 | 表示分布式执行 |

### 索引类型

- **Primary Key**：自动索引，唯一
- **Unique Index**：确保唯一性，快速查找
- **Composite Index**：多列，顺序很重要
- **Covering Index**：包含所有查询列
