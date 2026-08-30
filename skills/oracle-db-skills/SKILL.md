---
name: oracle-db-skills
description: 117 个 Oracle Database 参考指南，涵盖 SQL、PL/SQL、性能调优、安全、ORDS、SQLcl、迁移等。按需加载单个 skill 文件以获取任何 Oracle 主题的专家指导。
version: 1.0.0
repository: https://github.com/krisrice/oracle-db-skills
---

# Oracle DB Skills

103 个 Oracle Database 独立参考指南的集合。每个文件涵盖一个主题，包含解释、实用示例、最佳实践和常见错误。

## 如何使用

1. 使用下方的分类路由表**找到正确的 skill**。
2. **只读取**与用户任务相关的文件 — 不要一次性加载所有文件。
3. **应用指导**来回答问题、生成代码或审查现有工作。

## 分类路由

| 用户询问关于… | 从以下位置读取 |
|------------------|-----------|
| 备份、恢复、RMAN、Data Guard、redo/undo 日志、用户 | `skills/admin/` |
| JDBC、连接池、JSON、XML、spatial、全文搜索、事务 | `skills/appdev/` |
| RAC、CDB/PDB、Exadata、In-Memory、OCI、ATP/ADW | `skills/architecture/` |
| ERD、数据建模、分区、表空间 | `skills/design/` |
| Liquibase、Flyway、在线操作、EBR、utPLSQL、SQL 版本控制 | `skills/devops/` |
| Advanced Queuing、DBMS_SCHEDULER、物化视图、DBLinks、APEX | `skills/features/` |
| 从 PostgreSQL、MySQL、SQL Server、MongoDB 等迁移 | `skills/migrations/` |
| Alert log、ADR、adrci、空间、Top SQL、健康检查 | `skills/monitoring/` |
| ORDS、REST API、OAuth2、AutoREST、PL/SQL 网关 | `skills/ords/` |
| AWR、ASH、执行计划、索引、优化器统计、等待事件、内存 | `skills/performance/` |
| 包、游标、集合、错误处理、单元测试、调试 | `skills/plsql/` |
| 权限、VPD、TDE、加密、审计、网络安全 | `skills/security/` |
| SQL 模式、窗口函数、CTE、动态 SQL、注入 | `skills/sql-dev/` |
| SQLcl 命令、脚本、Liquibase CLI、MCP server、CI/CD | `skills/sqlcl/` |

## Skills 目录

```
skills/
├── admin/          数据库管理（备份、恢复、用户、redo/undo）
├── appdev/         应用开发（JSON、XML、spatial、text、连接池）
├── architecture/   基础设施（RAC、Multitenant、Exadata、In-Memory、OCI）
├── design/         Schema 设计（ERD、建模、分区、表空间）
├── devops/         CI/CD 和 DevOps（迁移、EBR、测试、版本控制）
├── features/       Oracle 特性（AQ、Scheduler、MVs、DBLinks、APEX）
├── migrations/     迁移到其他数据库到 Oracle
├── monitoring/     诊断（alert log、ADR、健康检查、空间、Top SQL）
├── ords/           Oracle REST Data Services
├── performance/    调优（AWR、ASH、索引、优化器、等待事件、内存）
├── plsql/          PL/SQL 开发（包、游标、集合、测试）
├── security/       安全（权限、VPD、TDE、审计、网络）
├── sql-dev/        SQL 开发（调优、模式、动态 SQL、注入）
└── sqlcl/          SQLcl CLI 工具（基础、脚本、Liquibase、MCP server）
```

## 关键起点

- **`skills/sqlcl/sqlcl-mcp-server.md`** — 通过 SQLcl MCP server 将 AI 助手连接到 Oracle
- **`skills/migrations/migration-assessment.md`** — 任何数据库迁移项目的起点
- **`skills/performance/explain-plan.md`** — 所有 SQL 性能工作的基础
- **`skills/plsql/plsql-package-design.md`** — PL/SQL 架构问题的基础
- **`skills/devops/schema-migrations.md`** — CI/CD 管道中的 Liquibase/Flyway with Oracle
