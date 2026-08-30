---
name: database-migrations
model: standard
description: 安全、零停机的数据库迁移策略，覆盖生产系统 schema 演进、回滚(rollback)规划、数据迁移、工具选型及反模式规避。触发词：数据库迁移(database migration)、schema 演进(schema evolution)、零停机(zero downtime)、回滚(rollback)、数据迁移(data migration)。
tags:
- Schema
- 数据库
---

# Database Migration Patterns

## Schema Evolution Strategies

| Strategy | Risk | Downtime | Best For |
|----------|------|----------|----------|
| **Additive-Only** | Very Low | None | 需要向后兼容保证的 API |
| **Expand-Contract** | Low | None | 重命名、重构、类型变更 |
| **Parallel Change** | Low | None | 关键表上的高风险变更 |
| **Lazy Migration** | Medium | None | 大表，批量迁移太慢的场景 |
| **Big Bang** | High | Yes | 仅用于 dev/staging 或小数据集 |

**默认使用 Additive-Only。** 仅在必须修改或移除已有结构时，才升级到 Expand-Contract。

---

## Zero-Downtime Patterns

每个生产环境迁移都必须避免锁表或破坏正在运行的 application code。

| Operation | Pattern | Key Constraint |
|-----------|---------|----------------|
| **Add column** | Nullable first | 绝不要在大表上直接添加无默认值的 `NOT NULL` |
| **Rename column** | Expand-contract | 添加新列 → dual-write → backfill → 切换读取 → 删除旧列 |
| **Drop column** | Deprecate first | 停止读取 → 停止写入 → 部署 → 删除 |
| **Change type** | Parallel column | 添加新类型列 → dual-write + cast → 切换 → 删除旧列 |
| **Add index** | Concurrent | `CREATE INDEX CONCURRENTLY` —— 不要在事务中包裹 |
| **Split table** | Extract + FK | 创建新表 → backfill → 添加 FK → 更新查询 → 删除旧列 |
| **Change constraint** | Two-phase | 先添加 `NOT VALID` → 再单独执行 `VALIDATE CONSTRAINT` |
| **Add enum value** | Append only | 绝不要删除或重命名已有值 |

---

## Migration Tools

| Tool | Ecosystem | Style | Key Strength |
|------|-----------|-------|-------------|
| **Prisma Migrate** | TypeScript/Node | Declarative (schema diff) | ORM 集成、shadow DB |
| **Knex** | JavaScript/Node | Imperative (up/down) | 轻量、灵活 |
| **Drizzle Kit** | TypeScript/Node | Declarative (schema diff) | Type-safe、类 SQL |
| **Alembic** | Python | Imperative (upgrade/downgrade) | 细粒度控制、autogenerate |
| **Django Migrations** | Python/Django | Declarative (model diff) | 自动检测 |
| **Flyway** | JVM / CLI | SQL file versioning | 简单、支持广泛的数据库 |
| **golang-migrate** | Go / CLI | SQL (up/down files) | 极简、可嵌入 |
| **Atlas** | Go / CLI | Declarative (HCL/SQL diff) | Schema-as-code、linting、CI |

根据你的 ORM 和部署 pipeline 选择工具。简单 schema 优先使用 declarative，需要精细数据操作时使用 imperative。

---

## Rollback Strategies

| Approach | When to Use |
|----------|-------------|
| **Reversible (up + down)** | 仅 schema 变更、早期阶段产品 |
| **Forward-only (corrective migration)** | 破坏性数据变更、大规模生产环境 |
| **Hybrid** | Schema 可回滚，数据变更只能向前 |

### Data Preservation

1. **Soft-delete columns** —— 删除前先重命名为带 `_deprecated` 后缀的列
2. **Snapshot tables** —— `CREATE TABLE _backup_<table>_<date> AS SELECT * FROM <table>`
3. **Point-in-time recovery** —— 确保 WAL archiving 覆盖迁移窗口
4. **Logical backups** —— 迁移前对受影响表执行 `pg_dump`

### Blue-Green Database

```
1. 将 primary 复制到 secondary (green)
2. 在 green 上应用 migration
3. 针对 green 运行 validation suite
4. 将流量切换到 green
5. 保留 blue 作为回滚目标 (N 小时)
6. 在 confidence window 后下线 blue
```

---

## Data Migration Patterns

### Backfill Strategies

| Strategy | Best For |
|----------|----------|
| **Inline backfill** | 小表 (< 100K 行) |
| **Batched backfill** | 中等表 (100K–10M 行) |
| **Background job** | 大表 (10M+ 行) |
| **Lazy backfill** | 不需要立即保持一致性的场景 |

### Batch Processing

```sql
DO $$
DECLARE
  batch_size INT := 1000;
  rows_updated INT;
BEGIN
  LOOP
    UPDATE my_table
    SET new_col = compute_value(old_col)
    WHERE id IN (
      SELECT id FROM my_table
      WHERE new_col IS NULL
      LIMIT batch_size
      FOR UPDATE SKIP LOCKED
    );
    GET DIAGNOSTICS rows_updated = ROW_COUNT;
    EXIT WHEN rows_updated = 0;
    PERFORM pg_sleep(0.1);  -- 节流以降低锁压力
    COMMIT;
  END LOOP;
END $$;
```

### Dual-Write Period

对于 expand-contract 和 parallel change：

1. **Dual-write** —— application 同时写入旧和新列/表
2. **Backfill** —— 用历史数据填充新结构
3. **Verify** —— 校验一致性 (行数、checksums)
4. **Cut over** —— 将读取切换到新结构，停止写入旧结构
5. **Cleanup** —— 在 cool-down period 后删除旧结构

---

## Testing Migrations

### Test Against Production-Like Data

- 绝不要仅在空数据或 synthetic data 上测试
- 使用 anonymized production snapshots
- 匹配数据量 —— 在 1K 行上能工作的迁移可能在 10M 行上锁表
- 复现 edge cases：NULLs、empty strings、max-length、unicode

### Migration CI Pipeline

```yaml
- name: Test migrations
  steps:
    - run: docker compose up -d db
    - run: npm run migrate:up        # 应用所有
    - run: npm run migrate:down      # 回滚所有
    - run: npm run migrate:up        # 重新应用 (idempotency)
    - run: npm run test:integration  # 验证 app
    - run: npm run migrate:status    # 无 pending
```

每个迁移 PR 必须通过：up → down → up → tests。

---

## Migration Checklist

### Pre-Migration

- [ ] 已在生产级数据量上测试
- [ ] Rollback 已编写并测试
- [ ] 已创建受影响表的备份
- [ ] App code 兼容新旧 schema
- [ ] 已在 staging 上 benchmark 执行时间
- [ ] 已分析锁影响
- [ ] 已部署 replication lag 监控

### During Migration

- [ ] 监控 lock waits 和 active queries
- [ ] 监控 replication lag
- [ ] 关注 error rate 是否飙升
- [ ] 准备好 rollback 命令

### Post-Migration

- [ ] Schema 符合预期状态
- [ ] Integration tests 在迁移后的 DB 上通过
- [ ] Data integrity 已验证 (行数、checksums)
- [ ] ORM schema / type definitions 已更新
- [ ] Deprecated structures 在 cool-down 后已清理
- [ ] Migration 已记录在团队 runbook 中

---

## NEVER Do

1. **NEVER** 直接在 production 中运行未测试的 migrations
2. **NEVER** 在未先移除所有 application 引用并部署的情况下删除列
3. **NEVER** 在单条语句中给大表添加无默认值的 `NOT NULL`
4. **NEVER** 在同一份 migration 文件中混合 schema DDL 和数据变更
5. **NEVER** 在 live system 中重命名列时跳过 dual-write 阶段
6. **NEVER** 假设 migrations 是瞬时的 —— 始终在 production-scale 数据上 benchmark
7. **NEVER** 为了"加速"迁移而在 production 中禁用 foreign key checks
8. **NEVER** 在 migration 完成前部署依赖于 schema 变更的 application code
