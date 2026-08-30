---
name: agentos-sdk
description: AgentOS 问责基础设施 SDK，为 AI agent 提供持久内存、项目管理、看板、活动日志、网格通信和自我进化协议
tags:
- AI
---

# AgentOS SDK Skill

## 概述
AgentOS 是 AI agent 的完整问责基础设施。它提供持久内存、项目管理、看板、头脑风暴存储、活动日志、网格通信和自我进化协议。

**使用时机：** 你需要跨 session 存储记忆、管理项目、跟踪任务、记录活动、与其他 agent 通信或进化你的行为时。

## 🆕 Agent 操作指南
**阅读 `AGENT-OPS.md` 以获取关于如何在 AgentOS 上作为 agent 操作的完整指南。** 它涵盖：
- 记忆组织（路径、标签、重要性）
- 项目管理（创建、更新、跟踪）
- 看板工作流程（任务、状态、优先级）
- 头脑风暴存储（想法、决策、学习）
- 日常操作（session 开始/结束检查清单）
- 自我进化协议

## 🆕 aos CLI - 完整的仪表板控制
`aos` CLI 为你提供对 AgentOS 仪表板的完全控制：

```bash
# 记忆
aos memory put "/learnings/today" '{"lesson": "verify first"}'
aos memory search "how to handle errors"

# 项目
aos project list
aos project create "New Feature" --status active

# 看板
aos kanban add "Fix bug" --project <id> --status todo --priority high
aos kanban move <task-id> done

# 头脑风暴
aos brainstorm add "Use WebSocket" --project <id> --type idea

# 活动日志
aos activity log "Completed API refactor" --project <id>

# 网格通信
aos mesh send <agent> "Topic" "Message body"
```

运行 `aos help` 或 `aos <command>` 以获取详细用法。

## Golden Sync（推荐）
为了获得坚不可摧的仪表板（记忆 + 项目卡片），请运行：
```bash
~/clawd/bin/agentos-golden-sync.sh
```

这会同步记忆 AND upserts 每个项目的 markdown 卡片：
`TASKS.md`、`IDEAS.md`、`CHANGELOG.md`、`CHALLENGES.md` → DB → Brain Dashboard。

## 🏷️ 记忆分类（必需）

**每个记忆都必须正确分类。** 使用这 8 个标准类别：

| 类别 | 颜色 | 用途 | 路径前缀 | 主标签 |
|----------|-------|---------|-------------|-------------|
| **Identity** | 🔴 红色 | 你是谁、用户资料、团队结构 | `identity/` | `["identity", ...]` |
| **Knowledge** | 🟠 橙色 | 事实、研究、文档 | `knowledge/` | `["knowledge", ...]` |
| **Memory** | 🟣 紫色 | 长期记忆、学习、决策 | `memory/` | `["memory", ...]` |
| **Preferences** | 🔵 蓝色 | 用户偏好、设置、风格 | `preferences/` | `["preferences", ...]` |
| **Projects** | 🟢 绿色 | 活跃工作、任务、代码上下文 | `projects/` | `["project", "<name>"]` |
| **Operations** | 🟤 棕色 | 每日日志、状态、心跳状态 | `operations/` | `["operations", ...]` |
| **Secrets** | ⚪ 灰色 | 访问信息、服务器位置（不是实际的密钥！） | `secrets/` | `["secrets", ...]` |
| **Protocols** | 🔵 青色 | SOP、检查清单、程序 | `protocols/` | `["protocols", ...]` |

### 路径结构
```
<category>/<subcategory>/<item>

示例：
identity/user/ben-profile
knowledge/research/ai-agents-market
memory/learnings/2026-02-mistakes
preferences/user/communication-style
projects/agentos/tasks
operations/daily/2026-02-13
secrets/access/hetzner-server
protocols/deploy/agentos-checklist
```

### 标签规则
每个记忆必须有：
1. **主类别标签** — 8 个类别之一
2. **子类别标签** — 更具体的分类
3. **可选项目标签** — 如果与项目相关

```bash
# 示例：存储带有正确标签的学习
AOS_TAGS='["memory", "learnings"]' AOS_SEARCHABLE=true \
  aos_put "/memory/learnings/2026-02-13" '{"lesson": "Always categorize memories"}'

# 示例：存储用户偏好
AOS_TAGS='["preferences", "user"]' \
  aos_put "/preferences/user/communication" '{"style": "direct, no fluff"}'
```

---

## 快速开始

```bash
# 设置环境变量
export AGENTOS_API_KEY="your-api-key"
export AGENTOS_BASE_URL="http://178.156.216.106:3100"  # 或 https://api.agentos.software
export AGENTOS_AGENT_ID="your-agent-id"

# 加载 SDK
source /path/to/agentos.sh

# 存储记忆
aos_put "/memories/today" '{"learned": "something important"}'

# 检索它
aos_get "/memories/today"

# 语义搜索
aos_search "what did I learn today"
```

## 配置

| 变量 | 必需 | 描述 |
|----------|----------|-------------|
| `AGENTOS_API_KEY` | 是 | 来自 agentos.software 仪表板的 API 密钥 |
| `AGENTOS_BASE_URL` | 是 | API 端点（默认：`http://178.156.216.106:3100`） |
| `AGENTOS_AGENT_ID` | 是 | 此 agent 实例的唯一标识符 |

## 核心 API 函数

### aos_put - 存储记忆
```bash
aos_put <path> <value_json> [options]

# 选项（作为调用前的环境变量）：
#   AOS_TTL=3600          # N 秒后过期
#   AOS_TAGS='["tag1"]'   # 标签的 JSON 数组
#   AOS_IMPORTANCE=0.8    # 0-1 重要性分数
#   AOS_SEARCHABLE=true   # 启用语义搜索

# 示例：
aos_put "/learnings/2026-02-04" '{"lesson": "Always verify before claiming done"}'
AOS_SEARCHABLE=true aos_put "/facts/solana" '{"info": "Solana uses proof of history"}'
AOS_TTL=86400 aos_put "/cache/price" '{"sol": 120.50}'
```

### aos_get - 检索记忆
```bash
aos_get <path>

# 返回 JSON：{"found": true, "path": "...", "value": {...}, "version_id": "...", "created_at": "..."}
# 或：{"found": false}

aos_get "/learnings/2026-02-04"
```

### aos_search - 语义搜索
```bash
aos_search <query> [limit] [path_prefix]

# 按语义相似度返回排名结果
# 仅搜索标记为 searchable=true 的记忆

aos_search "what mistakes have I made" 10
aos_search "solana facts" 5 "/facts"
```

### aos_delete - 删除记忆
```bash
aos_delete <path>

# 创建 tombstone 版本（软删除，保留历史）
aos_delete "/cache/old-data"
```

### aos_list - 列出子项
```bash
aos_list <prefix>

# 返回路径下的直接子项
aos_list "/learnings"
# → {"items": [{"path": "/learnings/2026-02-04", "type": "file"}, ...]}
```

### aos_glob - 模式匹配
```bash
aos_glob <pattern>

# 支持 * 和 ** 通配符
aos_glob "/learnings/*"           # 直接子项
aos_glob "/memories/**"           # 所有后代
aos_glob "/projects/*/config"     # 通配符段
```

### aos_history - 版本历史
```bash
aos_history <path> [limit]

# 返回记忆的所有版本（用于时间旅行）
aos_history "/config/settings" 20
```

### aos_agents - 列出所有 Agent
```bash
aos_agents

# 返回你的租户中所有带有记忆计数的 agent ID
# 用于发现其他 agent 实例
```

### aos_dump - 批量导出
```bash
aos_dump [agent_id] [limit]

# 导出 agent 的所有记忆（默认：当前 agent）
aos_dump "" 500
```

## 自我进化框架

**有关完整的自我进化指南，请参阅 [SELF-EVOLUTION.md](./SELF-EVOLUTION.md)。**

AgentOS 使 agent 能够通过以下方式每天变得更聪明：
- **错误跟踪** — 永不重复相同的错误
- **问题注册表** — 为将来参考索引解决方案
- **任务前检查** — 在行动前搜索学习
- **进度检查点** — 反压缩记忆保存
- **验证日志** — 证明任务确实已完成

### 快速开始：自我进化

```bash
# 在任何任务之前：检查过去的学习
aos_before_action "deployment"

# 犯错后：记录它
aos_mistake "What happened" "Root cause" "Lesson learned" "severity"

# 解决问题后：注册它
aos_problem_solved "OAuth 401 Error" "JWT format mismatch" "Added JWT branch to auth" "auth,oauth"

# 完成工作后：保存进度
aos_save_progress "Deployed API v2" "success" "JWT auth now working"

# 每 15-20 分钟：检查点上下文
aos_checkpoint "Building payment flow" "Stripe webhook incomplete" "Test mode works"

# 在 session 开始时：恢复上下文
aos_session_start

# 运行进化检查清单
aos_evolve_check
```

### 核心函数

| 函数 | 用途 |
|----------|---------|
| `aos_before_action` | 在行动前检查错误/解决方案 |
| `aos_mistake` | 记录失败 + 教训 |
| `aos_problem_solved` | 注册已解决的问题 |
| `aos_check_solved` | 搜索类似的已解决问题 |
| `aos_save_progress` | 记录已完成的任务（反压缩） |
| `aos_checkpoint` | 保存工作状态（每 15-20 分钟） |
| `aos_session_start` | 在 session 开始时恢复上下文 |
| `aos_verify_logged` | 记录验证证据 |
| `aos_daily_summary` | 回顾今天的工作 |
| `aos_evolve_check` | 显示进化检查清单 |

### 推荐的记忆结构

```
/self/
  identity.json       # 我是谁？核心特质、价值观
  capabilities.json   # 我能做什么？技能、工具
  preferences.json    # 我更喜欢如何工作？
  
/learnings/
  YYYY-MM-DD.json     # 每日学习
  mistakes/           # 记录的错误
  successes/          # 效果良好的东西
  
/patterns/
  communication/      # 如何与特定的人交谈
  problem-solving/    # 有效的方法
  tools/              # 工具特定知识
  
/relationships/
  <person-id>.json    # 关于与我一起工作的人的信息
  
/projects/
  <project-name>/     # 项目特定上下文
    context.json
    decisions.json
    todos.json

/reflections/
  weekly/             # 每周自我评估
  monthly/            # 每月回顾
```

### 自我反思协议

完成重要任务后，存储反思：

```bash
# 犯错后
aos_put "/learnings/mistakes/$(date +%Y-%m-%d)-$(uuidgen | cut -c1-8)" '{
  "type": "mistake",
  "what_happened": "I claimed a task was done without verifying",
  "root_cause": "Rushed to respond, skipped verification step",
  "lesson": "Always verify state before claiming completion",
  "prevention": "Add verification checklist to task completion flow",
  "severity": "high",
  "timestamp": "'$(date -Iseconds)'"
}' 

# 标记为可搜索，以便你以后可以找到它
AOS_SEARCHABLE=true AOS_TAGS='["mistake","verification","lesson"]' \
aos_put "/learnings/mistakes/..." '...'
```

### 自我改进循环

```bash
# 1. 开始工作前，回忆相关的学习
aos_search "mistakes I've made with $TASK_TYPE" 5

# 2. 完成工作后，反思
aos_put "/learnings/$(date +%Y-%m-%d)" '{
  "tasks_completed": [...],
  "challenges_faced": [...],
  "lessons_learned": [...],
  "improvements_identified": [...]
}'

# 3. 定期整合学习
aos_search "lessons from the past week" 20
# 然后综合并存储在 /reflections/weekly/
```

## 实时同步（WebSocket）

连接以在记忆更改时接收实时更新：

```javascript
const ws = new WebSocket('ws://178.156.216.106:3100');

ws.onopen = () => {
  // 认证
  ws.send(JSON.stringify({
    type: 'auth',
    token: process.env.AGENTOS_API_KEY
  }));
  
  // 订阅你的 agent 的更新
  ws.send(JSON.stringify({
    type: 'subscribe',
    agent_id: 'your-agent-id'
  }));
};

ws.onmessage = (event) => {
  const msg = JSON.parse(event.data);
  
  if (msg.type === 'memory:created') {
    console.log('New memory:', msg.path, msg.value);
  }
  
  if (msg.type === 'memory:deleted') {
    console.log('Memory deleted:', msg.path);
  }
};
```

### WebSocket 事件

| 事件 | Payload | 描述 |
|-------|---------|-------------|
| `memory:created` | `{agentId, path, versionId, value, tags, createdAt}` | 新记忆已存储 |
| `memory:deleted` | `{agentId, path, versionId, deletedAt}` | 记忆已删除 |

## Webhook 集成

注册 webhook 以在记忆更改时接收 HTTP 回调：

```bash
# 注册 webhook（通过仪表板或 API）
curl -X POST "$AGENTOS_BASE_URL/v1/webhooks" \
  -H "Authorization: Bearer $AGENTOS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-server.com/agentos-webhook",
    "events": ["memory:created", "memory:deleted"],
    "agent_id": "your-agent-id",
    "path_prefix": "/learnings"
  }'
```

### Webhook Payload

```json
{
  "event": "memory:created",
  "timestamp": "2026-02-04T09:50:00Z",
  "data": {
    "tenant_id": "...",
    "agent_id": "your-agent-id",
    "path": "/learnings/2026-02-04",
    "version_id": "...",
    "value": {"lesson": "..."},
    "tags": ["learning"],
    "created_at": "..."
  },
  "signature": "sha256=..."
}
```

## 速率限制和配额

| 操作 | 默认限制 |
|-----------|---------------|
| 读取操作（get, list, glob, history） | 60/分钟 |
| 写入操作（put, delete） | 60/分钟 |
| 搜索操作 | 20/分钟 |
| WebSocket 连接 | 每个租户 5 个 |

## 心跳上下文备份协议（关键）

**每个使用 AgentOS 的 agent 必须在每次心跳上实施强制性的上下文备份。**

### 为什么存在这个
- AI agent 在 session 压缩期间丢失上下文
- "记住在每个任务后备份"不起作用 — agent 会忘记
- 心跳驱动的备份确保上下文永不丢失

### Clawdbot 配置

在你的 `clawdbot.json` 中将心跳设置为 10 分钟：

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "10m",
        "model": "anthropic/claude-3-5-haiku-latest"
      }
    }
  }
}
```

### HEARTBEAT.md 模板

将此添加到你的工作区的 `HEARTBEAT.md`：

```markdown
## 🔴 强制性：上下文备份（首先执行此操作）

**在每次心跳上，在执行任何其他操作之前：**

1. **读取：** CONTEXT.md + 今天的每日笔记 + 昨天的每日笔记
2. **更新 CONTEXT.md**，包含：
   - 当前时间戳
   - session 中正在发生的事情
   - 最近的成就
   - 活跃任务
   - 重要的对话笔记
3. **更新每日笔记**（`memory/daily/YYYY-MM-DD.md`），包含重大事件
4. **然后才**继续其他心跳检查

这是一条硬性规则。永远不要跳过这一步。
```

### AGENTS.md 硬性规则

将此添加到你的 `AGENTS.md`：

```markdown
## 硬性规则：每次心跳都必须进行上下文备份

**每次心跳都必须包含上下文备份。** 没有例外。

### 协议（每次心跳强制性执行）

1. **读取当前状态：**
   - CONTEXT.md
   - 今天的每日笔记（`memory/daily/YYYY-MM-DD.md`）
   - 昨天的每日笔记（为了连续性）

2. **更新 CONTEXT.md**，包含：
   - 当前 session 焦点
   - 最近的成就（刚刚发生了什么）
   - 活跃任务/线程
   - 对话中的重要笔记
   - 更新时间戳

3. **更新每日笔记**，包含：
   - 重大事件
   - 做出的决策
   - 完成的任务
   - 以后可能需要的上下文

4. **然后才继续其他心跳任务**

### 心跳频率
心跳应该每 **10 分钟** 运行一次，以确保上下文被频繁保存。

### 黄金法则
**如果你重启后不会记住它，现在就写下来。**
```

### AgentOS 集成

在每次心跳上将你的 CONTEXT.md 同步到 AgentOS：

```bash
# 在你的心跳例程中，更新本地文件后：
aos_put "/context/current" "$(cat CONTEXT.md)"
aos_put "/daily/$(date +%Y-%m-%d)" "$(cat memory/daily/$(date +%Y-%m-%d).md)"
```

这确保你的上下文在本地 AND AgentOS 云中都有备份。

---

## 最佳实践

### 1. 使用有意义的路径
```bash
# 好的 — 分层、描述性
aos_put "/projects/raptor/decisions/2026-02-04-architecture" '...'

# 坏的 — 扁平、模糊
aos_put "/data123" '...'
```

### 2. 为所有重要内容添加标签
```bash
AOS_TAGS='["decision","architecture","raptor"]' \
AOS_SEARCHABLE=true \
aos_put "/projects/raptor/decisions/..." '...'
```

### 3. 对临时数据使用 TTL
```bash
# 1 小时后过期的缓存
AOS_TTL=3600 aos_put "/cache/api-response" '...'
```

### 4. 在询问前搜索
```bash
# 在询问用户信息之前，检查记忆
result=$(aos_search "user preferences for $TOPIC" 3)
```

### 5. 对重要更改进行版本控制
```bash
# 在覆盖前检查历史
aos_history "/config/critical-setting" 5
# 然后更新
aos_put "/config/critical-setting" '...'
```

## 故障排除

### "Unauthorized" 错误
- 检查 `AGENTOS_API_KEY` 是否设置正确
- 验证密钥是否具有所需的范围（`memory:read`、`memory:write`、`search:read`）

### 空搜索结果
- 确保记忆是使用 `searchable=true` 存储的
- 检查 embedding 是否已生成（可能需要几秒钟）

### 速率限制错误
- 实现指数退避
- 尽可能批量操作
- 检查 `X-PreAuth-RateLimit-Remaining` 头

## 网格通信（Agent 到 Agent）

AgentOS Mesh 支持 AI agent 之间的实时通信。

### 网格函数

```bash
# 向另一个 agent 发送消息
aos_mesh_send <to_agent> <topic> <body>

# 获取收件箱消息（发送给你的）
aos_mesh_inbox [limit]

# 获取发件箱消息（你发送的）
aos_mesh_outbox [limit]

# 检查本地队列中的消息（来自守护进程）
aos_mesh_pending

# 处理队列中的消息（返回 JSON，清除队列）
aos_mesh_process

# 列出网格上的所有 agent
aos_mesh_agents

# 为另一个 agent 创建任务
aos_mesh_task <assigned_to> <title> [description]

# 列出分配给你的任务
aos_mesh_tasks [status]

# 获取网格概览统计
aos_mesh_stats

# 获取最近的活动流
aos_mesh_activity [limit]

# 检查网格连接状态
aos_mesh_status
```

### 示例：发送消息

```bash
# 向另一个 agent 发送消息
aos_mesh_send "kai" "Project Update" "Finished the API integration, ready for review"

# 发送带有上下文的消息
aos_mesh_send "icarus" "Research Request" "Please analyze the latest DeFi trends on Solana"
```

### 示例：处理传入消息

```bash
# 检查是否有待处理的消息
aos_mesh_pending

# 处理并回复消息
messages=$(aos_mesh_process)
echo "$messages" | jq -r '.[] | "From: \(.from) - \(.topic)"'

# 回复每条消息
aos_mesh_send "kai" "Re: Project Update" "Thanks for the update, looks good!"
```

### 实时网格守护进程

为了实时接收消息，请运行网格守护进程：

```bash
node ~/clawd/bin/mesh-daemon.mjs
```

守护进程通过 WebSocket 连接并排队传入消息以进行处理。

### 网格事件（WebSocket）

| 事件 | Payload | 描述 |
|-------|---------|-------------|
| `mesh:message` | `{fromAgent, toAgent, topic, body, messageId}` | 收到新消息 |
| `mesh:task_update` | `{taskId, assignedTo, title, status}` | 任务状态已更改 |

### CLI 快捷方式

还有一个独立的 CLI 可用：

```bash
~/clawd/bin/mesh status    # 连接状态
~/clawd/bin/mesh pending   # 列出待处理的消息
~/clawd/bin/mesh send <to> "<topic>" "<body>"
~/clawd/bin/mesh agents    # 列出 agent
```

## API 参考

完整的 OpenAPI 规范可在以下位置获取：`$AGENTOS_BASE_URL/docs`

---

*AgentOS - 用于进化 AI agent 的持久内存和网格通信*