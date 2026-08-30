---
name: neural-memory
description: 带有 spreading activation 的关联记忆，用于持久、智能的回忆。 在以下情况下主动使用： (1) 你需要跨 sessions 记住 facts、decisions、errors 或 context (2) 用户问 \"do you remember...\" 或引用过去的对话 (3) 开始新任务时 —— 从记忆中注入相关 context (4) 做出决定或遇到错误后 —— 存储以供将来参考 (5) 用户问 \"why did X happen?\" —— 通过记忆追踪 causal chains 零 LLM 依赖。带有 Hebbian learning、memory decay、contradiction detection 和 temporal reasoning 的 neural graph。
homepage: https://github.com/nhadaututtheky/neural-memory
metadata:
  openclaw:
    emoji: brain
install:
- bins:
  - nmem
  id: pip
  kind: node
  label: pip install neural-memory
  package: neural-memory
os:
- darwin
- linux
- win32
primaryEnv: NEURALMEMORY_BRAIN
requires:
bins:
- python3
env:
- NEURALMEMORY_BRAIN
tags:
- AI
- 可观测性
---

# NeuralMemory —— AI Agents 的关联记忆

一个受生物学启发的记忆系统，使用 spreading activation 而非 keyword/vector search。Memories 形成一个 neural graph，其中 neurons 通过 20 种 typed synapses 连接。频繁共同访问的记忆会加强它们的连接（Hebbian learning）。陈旧的记忆自然 decay。Contradictions 被自动检测。

**为什么不只是 vector search？** Vector search 找到与你的查询相似的 documents。NeuralMemory 通过 graph traversal 找到 *概念相关* 的记忆 —— 即使没有 keyword 或 embedding 重叠。"What decision did we make about auth?" 同时激活 time + entity + concept neurons 并找到交集。

## 设置

### 1. 安装 NeuralMemory

```bash
pip install neural-memory
nmem init
```

这会创建带有默认 brain 的 `~/.neuralmemory/` 并自动配置 MCP。

### 2. 为 OpenClaw 配置 MCP

添加到你的 OpenClaw MCP 配置（`~/.openclaw/mcp.json` 或项目 `openclaw.json`）：

```json
{
  "mcpServers": {
    "neural-memory": {
      "command": "python3",
      "args": ["-m", "neural_memory.mcp"],
      "env": {
        "NEURALMEMORY_BRAIN": "default"
      }
    }
  }
}
```

### 3. 验证

```bash
nmem stats
```

你应该看到 brain 统计信息（neurons、synapses、fibers）。

## 工具参考

### 核心记忆工具

| 工具 | 用途 | 何时使用 |
|------|---------|-------------|
| `nmem_remember` | 存储记忆 | 在 decisions、errors、facts、insights、user preferences 之后 |
| `nmem_recall` | 查询记忆 | 在任务之前，当用户引用 past context 时，"do you remember..." |
| `nmem_context` | 获取最近的记忆 | 在 session 开始时，注入新鲜 context |
| `nmem_todo` | 带 30 天过期的快速 TODO | 任务跟踪 |

### 智能工具

| 工具 | 用途 | 何时使用 |
|------|---------|-------------|
| `nmem_auto` | 从文本自动提取记忆 | 在重要对话之后 —— 自动捕获 decisions、errors、TODOs |
| `nmem_recall` (depth=3) | 深度关联回忆 | 需要跨领域连接的复杂问题 |
| `nmem_habits` | 工作流模式建议 | 当用户重复类似的 action sequences 时 |

### 管理工具

| 工具 | 用途 | 何时使用 |
|------|---------|-------------|
| `nmem_health` | Brain health 诊断 | 定期检查，在分享 brain 之前 |
| `nmem_stats` | Brain 统计 | 记忆数量的快速概览 |
| `nmem_version` | Brain 快照和回滚 | 在 risky 操作之前，版本检查点 |
| `nmem_transplant` | 在 brains 之间转移记忆 | 跨项目知识共享 |

## 工作流

### Session 开始时
1. 调用 `nmem_context` 将最近的记忆注入你的意识
2. 如果用户提到特定 topic，调用 `nmem_recall` 并传入该 topic

### 对话期间
3. 当做出决定时：`nmem_remember` 并设置 type="decision"
4. 当发生错误时：`nmem_remember` 并设置 type="error"
5. 当用户陈述偏好时：`nmem_remember` 并设置 type="preference"
6. 当被问及过去事件时：使用适当的 depth 调用 `nmem_recall`

### Session 结束时
7. 对重要的对话片段调用 `nmem_auto` 并设置 action="process"
8. 这会 auto-extract facts、decisions、errors 和 TODOs

## 示例

### 记住一个决定
```
nmem_remember(
  content="Use PostgreSQL for production, SQLite for development",
  type="decision",
  tags=["database", "infrastructure"],
  priority=8
)
```

### 使用 spreading activation 回忆
```
nmem_recall(
  query="database configuration for production",
  depth=1,
  max_tokens=500
)
```
通过 graph traversal 而非 keyword matching 返回找到的记忆。Related memories（例如 "deploy uses Docker with pg_dump backups"）即使没有共享 keywords 也会浮现。

### 追踪 causal chains
```
nmem_recall(
  query="why did the deployment fail last week?",
  depth=2
)
```
沿着 CAUSED_BY 和 LEADS_TO synapses 追踪 cause-and-effect chains。

### 从对话中自动捕获
```
nmem_auto(
  action="process",
  text="We decided to switch from REST to GraphQL because the frontend needs flexible queries. The migration will take 2 sprints. TODO: update API docs."
)
```
自动提取：1 个 decision、1 个 fact、1 个 TODO。

## 关键特性

- **零 LLM 依赖** —— 纯算法：regex、graph traversal、Hebbian learning
- **Spreading activation** —— 通过 neural graph 进行关联回忆，而非 keyword/vector search
- **20 种 synapse 类型** —— Temporal (BEFORE/AFTER)、causal (CAUSED_BY/LEADS_TO)、semantic (IS_A/HAS_PROPERTY)、emotional (FELT/EVOKES)、conflict (CONTRADICTS)
- **记忆生命周期** —— Short-term → Working → Episodic → Semantic，带有 Ebbinghaus decay
- **Contradiction detection** —— 自动检测冲突记忆，deprioritizes 过时的记忆
- **Hebbian learning** —— "Neurons that fire together wire together" —— 记忆随使用改善
- **Temporal reasoning** —— Causal chain traversal、event sequences、temporal range queries
- **Brain versioning** —— Snapshot、rollback、diff brain state
- **Brain transplant** —— 在 brains 之间转移过滤后的知识
- **Vietnamese + English** —— 对 extraction 和 sentiment 的完整双语支持

## Depth Levels

| Depth | 名称 | 速度 | 用例 |
|-------|------|-------|----------|
| 0 | Instant | <10ms | Quick facts、recent context |
| 1 | Context | ~50ms | Standard recall（默认） |
| 2 | Habit | ~200ms | Pattern matching、workflow suggestions |
| 3 | Deep | ~500ms | Cross-domain associations、causal chains |

## 注意事项

- Memories 本地存储在 SQLite 中，位于 `~/.neuralmemory/brains/<brain>.db`
- 除非配置了可选的 embedding provider，否则不会将数据发送到外部服务
- Brain isolation：每个 brain 都是独立的，没有交叉污染
- `nmem_remember` 返回 fiber_id 用于 reference tracking
- Priority scale：0（trivial）到 10（critical），默认 5
- Memory types：fact、decision、preference、todo、insight、context、instruction、error、workflow、reference
