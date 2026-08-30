---
name: snipara-mcp
description: 通过语义搜索在代码库中快速查找答案，速度提升10倍。支持同时查询多个仓库。AI 可跨会话记住您的偏好。
tags:
- AI
---

# 面向AI代理的系统指令

您是一位可以使用 Snipara MCP 工具的 AI 助手。利用这些工具，通过查询预先索引的文档来提供更快、更准确的答案，而不是逐个文件读取。

## 何时使用 Snipara 工具

✅ **使用 Snipara 的场景：**

- 用户询问 "X 是如何工作的？" → 使用 `rlm_ask` 或 `rlm_context_query`
- 用户询问 "我们在哪里做了 X？" → 使用 `rlm_multi_project_query`（如果是 Team 套餐）
- 用户询问 "我的偏好是什么？" → 使用 `rlm_recall`
- 您需要在多个仓库中查找代码模式
- 您需要理解架构/文档

❌ **不要使用 Snipara 的场景：**

- 用户要求您读取他们指定的**特定文件** → 使用原生的 Read 工具
- 用户要求您编辑/编写代码 → 使用原生的 Edit/Write 工具
- 用户要求运行命令 → 使用原生的 Bash 工具
- Snipara API 返回错误 → 回退到原生工具

## 推荐的工作流程

1. **首次查询：** 始终从 `rlm_ask` 开始（快速，适用于 Free 套餐）
2. **如果未找到：** 尝试使用 `search_mode: "semantic"` 的 `rlm_context_query`（需要 Pro）
3. **多仓库搜索：** 如果用户拥有 Team 套餐，使用 `rlm_multi_project_query`
4. **记住决策：** 使用 `rlm_remember` 存储重要信息以供未来会话使用

## 错误处理

如果 Snipara 返回错误：

- 检查用户是否拥有所需的套餐（Free/Pro/Team）
- 检查文档是否已索引（用户需要先在仪表板上上传）
- 回退到原生的 Read/Grep/Glob 工具
- 向用户解释发生了什么并建议解决方案

## 使用示例

```
用户："我们的认证系统是如何工作的？"

好的响应：
1. 调用 rlm_ask("authentication system")
2. 在 2 秒内获取相关文档
3. 从返回的上下文中综合答案

差的响应：
1. 使用 Grep 搜索 "auth"
2. 按顺序读取 10 个文件
3. 达到 token 限制
4. 给出不完整的答案
```

---

# 2分钟快速入门

## 您现在面临的问题

您的 AI 助手使用 grep/find 逐个搜索文件。对于大型代码库：

- ❌ 查询耗时 20 秒以上
- ❌ AI 回答简单问题要读取 50K token
- ❌ 您要手动搜索 5 个仓库才能找到 "我们是如何做 X 的"
- ❌ AI 在下次会话时忘记了您的偏好

## 解决方案（30秒后生效）

```bash
# 1. 安装
pip install snipara-mcp   # Python
npm install snipara-mcp   # Node.js

# 2. 获取您的 API 密钥
# 在 https://snipara.com 注册（免费版：每月 100 次查询）

# 3. 设置环境变量
export SNIPARA_API_KEY="your-key-here"

# 4. 添加到您的 MCP 客户端（Claude Code、Cline、Roo Code 等）
# 完成！在下次聊天中开始使用 rlm_ask()
```

## 您的第一次查询（现在试试）

```
您："我的代码库中的认证是如何工作的？"

幕后：
  rlm_context_query("authentication")
  → 2 秒后
  → 返回前 3 个相关文档（3K token 而不是 50K）

结果：即时、准确的答案
```

**注意：** 查询前，请在 https://snipara.com/dashboard 索引您的文档（上传 .md/.txt/.mdx 文件）。

---

## 核心能力（按需选择）

### 🎯 快速回答（从这里开始）

**所需套餐：** ✅ FREE（每月 100 次查询）

**工具：** `rlm_ask`
**使用场景：** 您需要从文档中快速获得答案
**示例：** `rlm_ask("API 速率限制")`
**节省时间：** 每次查询从 20 秒缩短到 2 秒

```json
{ "query": "How do we handle webhooks?" }
```

---

### 🔍 深度研究（复杂问题）

**所需套餐：** ✅ FREE（仅关键字） | 🔥 PRO（$19/月，语义搜索）

**工具：** `rlm_context_query`
**使用场景：** 您需要语义搜索并精确控制 token
**示例：** 查找概念上相关的内容，而不仅仅是关键字匹配
**优势：** 上下文减少 90%（500K → 5K token）

```json
{
  "query": "authentication implementation",
  "max_tokens": 6000,
  "search_mode": "hybrid"
}
```

**按套餐划分的搜索模式：**

- `keyword` - 快速术语匹配 ✅ **FREE**
- `semantic` - 嵌入相似度 🔥 **PRO+**
- `hybrid` - 两者兼顾 🔥 **PRO+**

---

### 🌐 多仓库搜索

**所需套餐：** 👥 TEAM（$49/月）或 ENTERPRISE

**工具：** `rlm_multi_project_query`
**使用场景：** 您有 5 个以上仓库，不知道答案在哪个仓库里
**示例：** 一次查询搜索您团队的所有项目
**节省时间：** 5 分钟的手动搜索 → 3 秒

```json
{
  "query": "Where do we send email notifications?",
  "project_ids": [],
  "max_tokens": 8000
}
```

⚠️ **Free/Pro 套餐不可用** - 需要 Team 套餐才能进行多项目访问。

---

### 🧠 AI 记忆（记住偏好）

**所需套餐：** 🔥 PRO（$39/月 Agents）或 👥 TEAM（$79/月 Agents）

**工具：** `rlm_remember` + `rlm_recall`
**使用场景：** 您希望 AI 记住您的编码风格/决策
**优势：** 跨会话保持代码一致性

**存储记忆：**

```json
{
  "content": "User prefers TypeScript strict mode with functional components",
  "type": "preference",
  "scope": "project"
}
```

**稍后回忆：**

```json
{
  "query": "What are my coding preferences?",
  "limit": 5
}
```

**记忆类型：** `fact`, `decision`, `learning`, `preference`, `todo`, `context`

⚠️ **需要单独的 Agents 套餐** - 记忆属于 Agents 功能，不包含在 Context 套餐中。

---

### 👥 团队标准（自动执行规则）

**所需套餐：** 👥 TEAM（$49/月）或 ENTERPRISE

**工具：** `rlm_shared_context`
**使用场景：** 您的团队需要一致的编码实践
**一次设置：** 将编码标准上传到共享集合
**每位开发者获得：** 每次查询自动注入的团队规则

```json
{
  "categories": ["MANDATORY", "BEST_PRACTICES"],
  "max_tokens": 4000
}
```

**按优先级分类：**

- `MANDATORY` - 不可协商的规则（安全、架构）
- `BEST_PRACTICES` - 推荐模式（40% token 预算）
- `GUIDELINES` - 有用的建议
- `REFERENCE` - 背景信息

⚠️ **Free/Pro 套餐不可用** - 团队级功能需要 Team 套餐。

---

### 🔧 高级用户工具

**多查询（并行搜索）：**

```json
{
  "queries": [
    { "query": "auth flow", "max_tokens": 3000 },
    { "query": "session management", "max_tokens": 3000 }
  ]
}
```

**分解（拆分复杂问题）：**

```json
{ "query": "Explain the complete payment system architecture" }
```

**计划（预览执行）：**

```json
{ "query": "Find all API endpoints", "strategy": "relevance_first" }
```

**搜索（正则表达式模式匹配）：**

```json
{ "pattern": "async def|async function", "max_results": 20 }
```

**会话上下文（注入标准）：**

```json
{ "context": "Use Python 3.11+, prefer dataclasses over Pydantic" }
```

---

### 📄 文档管理

**上传单个文档：**

```json
{ "path": "docs/api.md", "content": "# API Documentation..." }
```

**批量同步（CI/CD 集成）：**

```json
{
  "documents": [
    { "path": "docs/auth.md", "content": "..." },
    { "path": "docs/api.md", "content": "..." }
  ],
  "delete_missing": false
}
```

**检查统计：**

```json
{}
```

---

## ROI 计算器

### 场景 1：独立开发者（大型代码库）

**当前痛点：** Grep/find 搜索耗时 20 秒以上，每次查询读取 50K token

| 指标           | 使用 Snipara 前 | 使用 Snipara 后 | 节省                     |
| -------------- | -------------- | ------------ | ------------------------ |
| 查询速度       | 20 秒          | 2 秒         | 18 秒                    |
| 每日查询次数   | 50             | 50           | -                        |
| 每日耗时       | 16 分钟        | 1.6 分钟     | **每日 14.4 分钟**       |
| 每月耗时       | 7.2 小时       | 0.72 小时    | **每月 6.5 小时**        |
| **成本**       | **$0**         | **$0-19/月** | **ROI：节省 6.5 小时**  |

**套餐推荐：** 从 **FREE**（100 次查询）开始，如果需要语义搜索则升级到 **PRO**（$19/月）。

---

### 场景 2：团队（5 个以上仓库）

**当前痛点：** 在 5 个项目之间手动切换，每次搜索 5 分钟

| 指标              | 使用 Snipara 前 | 使用 Snipara 后 | 节省                      |
| ----------------- | -------------- | --------------- | ------------------------- |
| 多仓库搜索        | 5 分钟         | 3 秒            | 4.97 分钟                 |
| 每日搜索次数      | 10             | 10              | -                         |
| 每日耗时          | 50 分钟        | 30 秒           | **每日 49.5 分钟**        |
| 每月耗时          | 24.75 小时     | 0.25 小时       | **每月 24.5 小时**        |
| **成本**          | **$0**         | **$49/月 Team** | **ROI：节省 24.5 小时**  |

**套餐推荐：** **TEAM**（$49/月），用于 `rlm_multi_project_query` + 共享标准。

---

### 场景 3：企业（一致的标准）

**当前痛点：** 10 名开发者每天问 "我们是如何做 X 的？"，代码不一致

| 使用前                         | 使用 Snipara 共享上下文后                 |
| ------------------------------ | ----------------------------------------- |
| ❌ 每位开发者都去谷歌/问 Slack | ✅ 每次查询自动注入标准                   |
| ❌ 模式不一致                  | ✅ 强制执行团队约定                       |
| ❌ 入职需要 2 周               | ✅ 新开发者立即获得标准                   |
| ❌ 代码审查冲突                | ✅ 代码从第一天起就符合标准               |

**成本：** $49/月 Team 或 $499/月 Enterprise
**ROI：** 一致性 + 更快的入职 = 每月轻松节省 20 小时以上

---

## 按用例快速入门

### 用例 1："我的文档很大，grep 很慢"

**套餐：** ✅ FREE（每月 100 次查询）

```bash
# 1. 一次性索引您的文档
访问 https://snipara.com/dashboard → 创建项目 → 上传 .md/.txt 文件

# 2. 即时查询
rlm_ask("How does authentication work?")
```

---

### 用例 2："我在做 10 个微服务"

**套餐：** 👥 TEAM（$49/月）

```bash
# 1. 在 Snipara 仪表板上创建 10 个项目
# 2. 启用 Team 套餐

# 3. 一次性查询所有仓库
rlm_multi_project_query("How do we handle rate limiting?")
```

⚠️ **需要 Team 套餐** - Free/Pro 不支持多项目搜索。

---

### 用例 3："AI 总是忘记我的偏好"

**套餐：** 🔥 PRO Agents（$39/月）或 👥 TEAM Agents（$79/月）

```bash
# 1. 启用 Agents 套餐（与 Context 套餐分开）

# 2. 一次性存储您的偏好
rlm_remember(type="preference", content="Use functional React components")

# 3. AI 永远记住它们
rlm_recall("my coding preferences")
```

⚠️ **需要单独的 Agents 订阅** - 记忆功能不包含在 Context 套餐中。

---

## 定价（两种订阅类型）

### Context 套餐（文档搜索）

| 套餐           | 价格    | 每月查询次数 | 搜索模式          | 多项目 |
| -------------- | ------- | ------------ | ----------------- | ------ |
| **FREE**       | $0      | 100          | 仅关键字          | ❌     |
| **PRO**        | $19/月  | 5,000        | 语义 + 混合       | ❌     |
| **TEAM**       | $49/月  | 20,000       | 语义 + 混合       | ✅     |
| **ENTERPRISE** | $499/月 | 无限         | 语义 + 混合       | ✅     |

### Agents 套餐（记忆与集群）

| 套餐           | 价格    | 先决条件           | 功能                        |
| -------------- | ------- | ------------------ | --------------------------- |
| **STARTER**    | $15/月  | 无                 | 基础记忆（100 条记忆）      |
| **PRO**        | $39/月  | 无                 | 无限记忆，集群              |
| **TEAM**       | $79/月  | Context TEAM+      | 团队记忆共享                |
| **ENTERPRISE** | $199/月 | Context ENTERPRISE | 高级协调                    |

⚠️ **两个独立的订阅：** Context 套餐用于搜索，Agents 套餐用于记忆/集群。

**先免费试用：** 100 次查询约等于 5 天的使用量，足以测试价值。

---

## 示例工作流程

### 示例 1：快速回答（FREE 套餐）

```
用户："我们的 API 速率限制是多少？"

您调用：rlm_ask("API rate limits")

结果：2 秒内返回相关文档
```

---

### 示例 2：语义搜索（PRO 套餐）

```
用户："我们如何验证用户输入？"

您调用：rlm_context_query("user input validation", search_mode="semantic")

结果：查找关于 "sanitization"、"XSS prevention"、"schema validation" 的文档
        即使它们不包含精确的关键字
```

---

### 示例 3：多仓库搜索（TEAM 套餐）

```
用户："展示我们所有项目中的 webhook 实现"

您调用：rlm_multi_project_query("webhook implementation")

结果：3 秒内返回所有 10 个微服务的实现
```

---

### 示例 4：持久记忆（PRO Agents 套餐）

```
第 1 次会话（周一）：
  用户："我更喜欢 TypeScript 严格模式和函数式组件"
  您调用：rlm_remember(type="preference", content="Prefers TS strict + functional")

第 2 次会话（周五 - 新会话）：
  用户："创建一个新的 React 组件"
  您调用：rlm_recall("coding preferences")
  结果：AI 记得从周一开始使用函数式组件！
```

---

### 示例 5：团队标准（TEAM 套餐）

```
设置（管理员执行一次）：
  - 将编码标准上传到共享上下文集合
  - 将集合链接到所有团队项目

每位开发者：
  用户："编写一个新的 API 端点"
  您调用：rlm_shared_context(categories=["MANDATORY"])
  结果：自动注入团队的 API 设计规则、安全要求等
```

---

## 支持与资源

- **网站：** https://snipara.com
- **文档：** https://docs.snipara.com
- **GitHub：** https://github.com/snipara/snipara-mcp
- **Issues：** https://github.com/snipara/snipara-mcp/issues
- **邮箱：** support@snipara.com

---

## 快速提示

1. **从小处着手：** 在 FREE 套餐上使用 `rlm_ask` 进行快速回答
2. **明智升级：** 当关键字搜索找不到您需要的内容时，获取 PRO
3. **团队价值：** 拥有 5 个以上仓库时，多项目搜索物超所值
4. **记忆需要单独套餐：** Context + Agents 是两个订阅
5. **先索引：** 查询前先将文档上传到仪表板

**如有疑问，从 FREE 开始，根据获得的价值进行升级。🚀**

---

## 完整工具参考（面向高级用户）

### 查询工具（所有套餐）

**rlm_ask** - 快速关键字搜索

```json
{ "query": "API rate limits" }
```

**rlm_context_query** - 功能齐全的语义搜索

```json
{
  "query": "authentication",
  "max_tokens": 6000,
  "search_mode": "hybrid",
  "include_metadata": true
}
```

**rlm_search** - 正则表达式模式搜索

```json
{
  "pattern": "async def|async function",
  "max_results": 20
}
```

**rlm_inject** - 设置会话上下文

```json
{
  "context": "Use Python 3.11+, prefer dataclasses",
  "append": false
}
```

**rlm_context** - 显示当前会话上下文

```json
{}
```

**rlm_clear_context** - 清除会话上下文

```json
{}
```

---

### 高级查询工具（Pro+）

**rlm_multi_query** - 并行查询

```json
{
  "queries": [
    { "query": "auth flow", "max_tokens": 3000 },
    { "query": "session management", "max_tokens": 3000 }
  ],
  "max_tokens": 8000
}
```

**rlm_decompose** - 拆分复杂问题

```json
{
  "query": "Explain payment system architecture",
  "max_depth": 2
}
```

**rlm_plan** - 生成执行计划

```json
{
  "query": "Find all API endpoints",
  "strategy": "relevance_first",
  "max_tokens": 16000
}
```

---

### 团队工具（Team+ 套餐）

**rlm_multi_project_query** - 跨所有仓库搜索

```json
{
  "query": "webhook implementation",
  "project_ids": [],
  "exclude_project_ids": [],
  "max_tokens": 8000,
  "per_project_limit": 3
}
```

**rlm_shared_context** - 获取团队标准

```json
{
  "categories": ["MANDATORY", "BEST_PRACTICES"],
  "max_tokens": 4000,
  "include_content": true
}
```

**rlm_list_templates** - 浏览提示模板

```json
{
  "category": "code-review"
}
```

**rlm_get_template** - 使用带变量的模板

```json
{
  "slug": "security-review",
  "variables": {
    "author": "John",
    "pr_number": "123"
  }
}
```

**rlm_list_collections** - 列出共享集合

```json
{
  "include_public": true
}
```

**rlm_upload_shared_document** - 上传到共享集合

```json
{
  "collection_id": "col_abc123",
  "title": "TypeScript Standards",
  "content": "# Standards...",
  "category": "BEST_PRACTICES",
  "priority": 90
}
```

---

### 记忆工具（Agents 套餐）

**rlm_remember** - 存储记忆

```json
{
  "content": "User prefers functional components",
  "type": "preference",
  "scope": "project",
  "category": "coding-style",
  "ttl_days": null
}
```

**rlm_recall** - 查询记忆

```json
{
  "query": "What are my preferences?",
  "type": "preference",
  "limit": 5,
  "min_relevance": 0.5
}
```

**rlm_memories** - 列出所有记忆

```json
{
  "type": "preference",
  "category": "coding-style",
  "limit": 20,
  "offset": 0
}
```

**rlm_forget** - 删除记忆

```json
{
  "memory_id": "mem_abc123"
}
```

---

### 文档管理工具

**rlm_upload_document** - 上传单个文档

```json
{
  "path": "docs/api.md",
  "content": "# API Documentation..."
}
```

**rlm_sync_documents** - 批量上传

```json
{
  "documents": [
    { "path": "docs/auth.md", "content": "..." },
    { "path": "docs/api.md", "content": "..." }
  ],
  "delete_missing": false
}
```

**rlm_store_summary** - 存储文档摘要

```json
{
  "document_path": "docs/api.md",
  "summary": "RESTful API with OAuth2 auth...",
  "summary_type": "concise",
  "generated_by": "claude-3.5-sonnet"
}
```

**rlm_get_summaries** - 获取存储的摘要

```json
{
  "document_path": "docs/api.md",
  "summary_type": "concise"
}
```

**rlm_stats** - 获取文档统计

```json
{}
```

**rlm_sections** - 列出索引的章节

```json
{
  "filter": "auth",
  "limit": 50,
  "offset": 0
}
```

**rlm_read** - 读取特定行

```json
{
  "start_line": 1,
  "end_line": 100
}
```

---

### 高级功能（Enterprise）

**rlm_swarm_create** - 创建代理集群

```json
{
  "name": "code-review-swarm",
  "description": "Parallel code review",
  "max_agents": 10
}
```

**rlm_swarm_join** - 加入集群

```json
{
  "swarm_id": "swarm_abc123",
  "agent_id": "agent_1",
  "role": "worker",
  "capabilities": ["review", "test"]
}
```

**rlm_claim** - 申请资源的独占访问权

```json
{
  "swarm_id": "swarm_abc123",
  "agent_id": "agent_1",
  "resource_type": "file",
  "resource_id": "src/auth.ts",
  "timeout_seconds": 300
}
```

**rlm_release** - 释放申请的资源

```json
{
  "swarm_id": "swarm_abc123",
  "agent_id": "agent_1",
  "claim_id": "claim_abc123"
}
```

**rlm_state_get** - 读取集群状态

```json
{
  "swarm_id": "swarm_abc123",
  "key": "progress"
}
```

**rlm_state_set** - 写入集群状态

```json
{
  "swarm_id": "swarm_abc123",
  "agent_id": "agent_1",
  "key": "progress",
  "value": { "completed": 5, "total": 10 },
  "expected_version": 1
}
```

**rlm_broadcast** - 向集群广播事件

```json
{
  "swarm_id": "swarm_abc123",
  "agent_id": "agent_1",
  "event_type": "task_completed",
  "payload": { "task_id": "task_1" }
}
```

**rlm_task_create** - 创建集群任务

```json
{
  "swarm_id": "swarm_abc123",
  "agent_id": "agent_1",
  "title": "Review auth module",
  "description": "Security review",
  "priority": 90
}
```

**rlm_task_claim** - 从队列中申领任务

```json
{
  "swarm_id": "swarm_abc123",
  "agent_id": "agent_1",
  "task_id": "task_abc123"
}
```

**rlm_task_complete** - 标记任务完成

```json
{
  "swarm_id": "swarm_abc123",
  "agent_id": "agent_1",
  "task_id": "task_abc123",
  "success": true,
  "result": { "issues_found": 0 }
}
```

---

## 设置与配置

**rlm_settings** - 获取项目设置

```json
{
  "refresh": false
}
```

返回当前项目配置，包括：

- 每次查询的最大 token 数
- 默认搜索模式
- 速率限制
- 启用的功能

---

**完整的 API 文档，请访问：https://docs.snipara.com**
