---
name: drizzle
description: Drizzle ORM 模式和数据库指南。在处理数据库模式（src/database/schemas/*）、定义表、创建迁移或数据库模型代码时使用。在 Drizzle 模式定义、数据库迁移或 ORM 使用问题上触发。
---

# Drizzle ORM 模式风格指南

## 配置

- 配置：`drizzle.config.ts`
- 模式：`src/database/schemas/`
- 迁移：`src/database/migrations/`
- 方言：`postgresql`，启用 `strict: true`

## 辅助函数

位置：`src/database/schemas/_helpers.ts`

- `timestamptz(name)`：带时区的时间戳
- `createdAt()`、`updatedAt()`、`accessedAt()`：标准时间戳列
- `timestamps`：包含所有三个的对象，便于展开

## 命名约定

- **表**：复数 snake_case（`users`、`session_groups`）
- **列**：snake_case（`user_id`、`created_at`）

## 列定义

### 主键

```typescript
id: text('id')
  .primaryKey()
  .$defaultFn(() => idGenerator('agents'))
  .notNull(),
```

ID 前缀使实体类型可区分。对于内部表，使用 `uuid`。

### 外键

```typescript
userId: text('user_id')
  .references(() => users.id, { onDelete: 'cascade' })
  .notNull(),
```

### 时间戳

```typescript
...timestamps,  // 从 _helpers.ts 展开
```

### 索引

```typescript
// 返回数组（对象样式已弃用）
(t) => [uniqueIndex('client_id_user_id_unique').on(t.clientId, t.userId)],
```

## 类型推断

```typescript
export const insertAgentSchema = createInsertSchema(agents);
export type NewAgent = typeof agents.$inferInsert;
export type AgentItem = typeof agents.$inferSelect;
```

## 示例模式

```typescript
export const agents = pgTable(
  'agents',
  {
    id: text('id')
      .primaryKey()
      .$defaultFn(() => idGenerator('agents'))
      .notNull(),
    slug: varchar('slug', { length: 100 })
      .$defaultFn(() => randomSlug(4))
      .unique(),
    userId: text('user_id')
      .references(() => users.id, { onDelete: 'cascade' })
      .notNull(),
    clientId: text('client_id'),
    chatConfig: jsonb('chat_config').$type<LobeAgentChatConfig>(),
    ...timestamps,
  },
  (t) => [uniqueIndex('client_id_user_id_unique').on(t.clientId, t.userId)],
);
```

## 常见模式

### 连接表（多对多）

```typescript
export const agentsKnowledgeBases = pgTable(
  'agents_knowledge_bases',
  {
    agentId: text('agent_id')
      .references(() => agents.id, { onDelete: 'cascade' })
      .notNull(),
    knowledgeBaseId: text('knowledge_base_id')
      .references(() => knowledgeBases.id, { onDelete: 'cascade' })
      .notNull(),
    userId: text('user_id')
      .references(() => users.id, { onDelete: 'cascade' })
      .notNull(),
    enabled: boolean('enabled').default(true),
    ...timestamps,
  },
  (t) => [primaryKey({ columns: [t.agentId, t.knowledgeBaseId] })],
);
```

## 查询风格

**始终使用 `db.select()` 构建器 API。永远不要使用 `db.query.*` 关系 API**（`findMany`、`findFirst`、`with:`）。

关系 API 生成带有 `json_build_array` 的复杂横向连接，脆弱且难以调试。

### 选择单行

```typescript
// ✅ 好
const [result] = await this.db
  .select()
  .from(agents)
  .where(eq(agents.id, id))
  .limit(1);
return result;

// ❌ 坏：关系 API
return this.db.query.agents.findFirst({
  where: eq(agents.id, id),
});
```

### 带 JOIN 的选择

```typescript
// ✅ 好：显式选择 + leftJoin
const rows = await this.db
  .select({
    runId: agentEvalRunTopics.runId,
    score: agentEvalRunTopics.score,
    testCase: agentEvalTestCases,
    topic: topics,
  })
  .from(agentEvalRunTopics)
  .leftJoin(agentEvalTestCases, eq(agentEvalRunTopics.testCaseId, agentEvalTestCases.id))
  .leftJoin(topics, eq(agentEvalRunTopics.topicId, topics.id))
  .where(eq(agentEvalRunTopics.runId, runId))
  .orderBy(asc(agentEvalRunTopics.createdAt));

// ❌ 坏：使用 `with:` 的关系 API
return this.db.query.agentEvalRunTopics.findMany({
  where: eq(agentEvalRunTopics.runId, runId),
  with: { testCase: true, topic: true },
});
```

### 带聚合的选择

```typescript
// ✅ 好：select + leftJoin + groupBy
const rows = await this.db
  .select({
    id: agentEvalDatasets.id,
    name: agentEvalDatasets.name,
    testCaseCount: count(agentEvalTestCases.id).as('testCaseCount'),
  })
  .from(agentEvalDatasets)
  .leftJoin(agentEvalTestCases, eq(agentEvalDatasets.id, agentEvalTestCases.datasetId))
  .groupBy(agentEvalDatasets.id);
```

### 一对多（单独查询）

当您需要父记录及其子记录时，使用两个查询而不是关系 `with:`：

```typescript
// ✅ 好：两个简单查询
const [dataset] = await this.db
  .select()
  .from(agentEvalDatasets)
  .where(eq(agentEvalDatasets.id, id))
  .limit(1);

if (!dataset) return undefined;

const testCases = await this.db
  .select()
  .from(agentEvalTestCases)
  .where(eq(agentEvalTestCases.datasetId, id))
  .orderBy(asc(agentEvalTestCases.sortOrder));

return { ...dataset, testCases };
```

## 数据库迁移

有关详细的迁移指南，请参阅 `db-migrations` 技能。
