---
name: tidb-sql
description: 为TiDB编写、审查和调整SQL，正确处理TiDB与MySQL的差异（VECTOR类型+向量索引/函数、全文搜索、AUTO_RANDOM、乐观/悲观事务、外键、视图、DDL限制以及不支持的MySQL功能如存储过程/触发器/事件/GEOMETRY/SPATIAL）。用于生成必须在TiDB上运行的SQL、将MySQL SQL迁移到TiDB或调试TiDB SQL兼容性错误时。
---

# TiDB SQL（MySQL兼容聚焦）

目标：默认生成在TiDB上正确运行的SQL，并避免"在MySQL上工作但在TiDB上中断"的构造。

## 工作流程（每次使用）

1. 识别目标引擎和版本：
   - 运行`SELECT VERSION();`
   - 如果结果包含`TiDB`，将其视为TiDB并解析版本（需要用于Vector/外键等功能门控）。
   - 如果连接到TiDB Cloud，确保客户端启用SSL证书+身份验证（参见`skills/tidb-sql/references/tidb-cloud-ssl.md`）。
2. 如果请求依赖于它们，询问2个快速能力问题：
   - "你有TiFlash吗？"（向量索引需要）
   - "这是在支持全文搜索的区域的TiDB Cloud Starter/Essential吗？"（可用性有限）
3. 使用TiDB安全默认值生成SQL：
   - 避免不支持的MySQL功能（存储过程/触发器/事件/UDF/GEOMETRY/SPATIAL等）
   - 将视图视为只读
   - 将主键更改视为迁移/重建工作
4. 如果用户提供MySQL SQL，进行兼容性检查：
   - 用TiDB替代方案替换不支持的功能
   - 明确说明行为差异和版本先决条件
5. 如果SQL运行缓慢或意外失败，使用TiDB原生诊断：
   - 使用`EXPLAIN FORMAT = "tidb_json"`获取结构化计划和操作符树。
   - 使用`EXPLAIN ANALYZE`比较`estRows`与`actRows`（它执行查询）。
   - 如果计划看起来不正确，考虑`ANALYZE TABLE ...`刷新统计信息。

## 高信号差异（牢记）

- **向量**：TiDB支持`VECTOR`/`VECTOR(D)`类型和向量函数/索引；MySQL不支持。
- **无GEOMETRY/SPATIAL**：避免`GEOMETRY`、空间函数和`SPATIAL`索引。
- **无存储过程/函数/触发器/事件**：将逻辑移至应用层或外部调度器。
- **全文搜索（TiDB功能）**：可用时使用TiDB全文搜索SQL；不要假设MySQL`FULLTEXT`到处都能工作。
- **视图是只读的**：不能对视图执行`UPDATE/INSERT/DELETE`。
- **外键**：TiDB v6.6.0+支持；否则，不要依赖FK强制。
- **主键更改受限**：假设主键更改需要"创建新表+回填+交换"。
- **AUTO_RANDOM**：适当时优先使用`AUTO_RANDOM`而不是`AUTO_INCREMENT`以避免写入热点。
- **事务**：TiDB支持悲观和乐观模式；在应用逻辑中处理乐观`COMMIT`失败。

## 使用这些参考（在此技能内）

- `skills/tidb-sql/references/vector.md` - VECTOR类型、函数、向量索引DDL和查询模式。
- `skills/tidb-sql/references/full-text-search.md` - 全文搜索SQL模式和可用性注意事项。
- `skills/tidb-sql/references/auto-random.md` - `AUTO_RANDOM`规则、DDL模式和限制。
- `skills/tidb-sql/references/transactions.md` - 悲观与乐观模式以及会话/全局参数。
- `skills/tidb-sql/references/mysql-compatibility-notes.md` - 其他常见的"MySQL vs TiDB"差异。
- `skills/tidb-sql/references/explain.md` - EXPLAIN/EXPLAIN ANALYZE用法、tidb_json和dot格式。
- `skills/tidb-sql/references/flashback.md` - FLASHBACK TABLE/DATABASE和FLASHBACK CLUSTER恢复手册。
- `skills/tidb-sql/references/tidb-cloud-ssl.md` - TiDB Cloud SSL验证要求和客户端标志。
