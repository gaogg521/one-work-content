---
name: elite-longterm-memory
version: 1.2.3
description: 适用于 Cursor、Claude、ChatGPT 和 Copilot 的终极 AI agent 记忆系统。WAL 协议 + 向量搜索 + git-notes + 云备份。再也不会丢失上下文。Vibe-coding 就绪。
author: NextFrontierBuilds
keywords:
- memory
- ai-agent
- ai-coding
- long-term-memory
- vector-search
- lancedb
- git-notes
- wal
- persistent-context
- claude
- claude-code
- gpt
- chatgpt
- cursor
- copilot
- github-copilot
- openclaw
- moltbot
- vibe-coding
- agentic
- ai-tools
- developer-tools
- devtools
- typescript
- llm
- automation
metadata: None
openclaw: None
emoji: 🧠
requires: None
env:
- OPENAI_API_KEY
plugins:
- memory-lancedb
tags:
- AI
- Git
---

# Elite Longterm Memory 🧠

**AI agents 的终极记忆系统。** 将6种经过验证的方法组合成一个坚不可摧的架构。

再也不会丢失上下文。再也不会忘记决策。再也不会重复错误。

## 架构概览

```
┌─────────────────────────────────────────────────────────────────┐
│                    ELITE LONGTERM MEMORY                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   HOT RAM   │  │  WARM STORE │  │  COLD STORE │             │
│  │             │  │             │  │             │             │
│  │ SESSION-    │  │  LanceDB    │  │  Git-Notes  │             │
│  │ STATE.md    │  │  Vectors    │  │  Knowledge  │             │
│  │             │  │             │  │  Graph      │             │
│  │ (survives   │  │ (semantic   │  │ (permanent  │             │
│  │  compaction)│  │  search)    │  │  decisions) │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│         │                │                │                     │
│         └────────────────┼────────────────┘                     │
│                          ▼                                      │
│                  ┌─────────────┐                                │
│                  │  MEMORY.md  │  ← 精选的长期记忆              │
│                  │  + daily/   │    (人类可读)                  │
│                  └─────────────┘                                │
│                          │                                      │
│                          ▼                                      │
│                  ┌─────────────┐                                │
│                  │ SuperMemory │  ← 云备份（可选）              │
│                  │    API      │                                │
│                  └─────────────┘                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 5 层记忆体系

### 第1层：HOT RAM (SESSION-STATE.md)
**来源：bulletproof-memory**

在压缩后仍然保留的活跃工作记忆。Write-Ahead Log 协议。

```markdown
# SESSION-STATE.md — 活跃工作记忆

## 当前任务
[我们现在正在做什么]

## 关键上下文
- 用户偏好：...
- 已做决策：...
- 阻塞项：...

## 待处理操作
- [ ] ...
```

**规则：** 在响应之前写入。由用户输入触发，而非 agent 记忆。

### 第2层：WARM STORE (LanceDB 向量)
**来源：lancedb-memory**

跨所有记忆的语义搜索。自动召回注入相关上下文。

```bash
# 自动召回（自动发生）
memory_recall query="project status" limit=5

# 手动存储
memory_store text="User prefers dark mode" category="preference" importance=0.9
```

### 第3层：COLD STORE (Git-Notes 知识图谱)
**来源：git-notes-memory**

结构化的决策、学习内容和上下文。支持分支感知。

```bash
# 存储决策（静默执行——绝不宣布）
python3 memory.py -p $DIR remember '{"type":"decision","content":"Use React for frontend"}' -t tech -i h

# 检索上下文
python3 memory.py -p $DIR get "frontend"
```

### 第4层：精选归档 (MEMORY.md + daily/)
**来源：OpenClaw 原生**

人类可读的长期记忆。每日日志 + 提炼的智慧。

```
workspace/
├── MEMORY.md              # 精选长期记忆（精华内容）
└── memory/
    ├── 2026-01-30.md      # 每日日志
    ├── 2026-01-29.md
    └── topics/            # 按主题分类的文件
```

### 第5层：云备份 (SuperMemory) — 可选
**来源：supermemory**

跨设备同步。与你的知识库聊天。

```bash
export SUPERMEMORY_API_KEY="your-key"
supermemory add "Important context"
supermemory search "what did we decide about..."
```

### 第6层：自动提取 (Mem0) — 推荐
**新增：自动事实提取**

Mem0 自动从对话中提取事实。减少 80% 的 token。

```bash
npm install mem0ai
export MEM0_API_KEY="your-key"
```

```javascript
const { MemoryClient } = require('mem0ai');
const client = new MemoryClient({ apiKey: process.env.MEM0_API_KEY });

// 对话自动提取事实
await client.add(messages, { user_id: "user123" });

// 检索相关记忆
const memories = await client.search(query, { user_id: "user123" });
```

优势：
- 自动提取偏好、决策、事实
- 去重并更新现有记忆
- 相比原始历史减少 80% 的 token
- 自动跨会话工作

## 快速设置

### 1. 创建 SESSION-STATE.md (Hot RAM)

```bash
cat > SESSION-STATE.md << 'EOF'
# SESSION-STATE.md — 活跃工作记忆

此文件是 agent 的 "RAM" —— 在压缩、重启、分心后仍然保留。

## 当前任务
[无]

## 关键上下文
[尚无]

## 待处理操作
- [ ] 无

## 近期决策
[尚无]

---
*最后更新：[时间戳]*
EOF
```

### 2. 启用 LanceDB (Warm Store)

在 `~/.openclaw/openclaw.json` 中：

```json
{
  "memorySearch": {
    "enabled": true,
    "provider": "openai",
    "sources": ["memory"],
    "minScore": 0.3,
    "maxResults": 10
  },
  "plugins": {
    "entries": {
      "memory-lancedb": {
        "enabled": true,
        "config": {
          "autoCapture": false,
          "autoRecall": true,
          "captureCategories": ["preference", "decision", "fact"],
          "minImportance": 0.7
        }
      }
    }
  }
}
```

### 3. 初始化 Git-Notes (Cold Store)

```bash
cd ~/clawd
git init  # 如果尚未初始化
python3 skills/git-notes-memory/memory.py -p . sync --start
```

### 4. 验证 MEMORY.md 结构

```bash
# 确保你有：
# - 工作区根目录下的 MEMORY.md
# - 用于每日日志的 memory/ 文件夹
mkdir -p memory
```

### 5. （可选）设置 SuperMemory

```bash
export SUPERMEMORY_API_KEY="your-key"
# 添加到 ~/.zshrc 以持久化
```

## Agent 指令

### 会话开始时
1. 读取 SESSION-STATE.md —— 这是你的热上下文
2. 运行 `memory_search` 获取相关的前期上下文
3. 检查 memory/YYYY-MM-DD.md 中的近期活动

### 对话期间
1. **用户给出具体细节？** → 在响应之前写入 SESSION-STATE.md
2. **做出了重要决策？** → 静默存储到 Git-Notes
3. **表达了偏好？** → 使用 `memory_store` 并设置 importance=0.9

### 会话结束时
1. 用最终状态更新 SESSION-STATE.md
2. 如果值得长期保留，将重要事项移至 MEMORY.md
3. 在 memory/YYYY-MM-DD.md 中创建/更新每日日志

### 记忆卫生（每周）
1. 审查 SESSION-STATE.md —— 归档已完成的任务
2. 检查 LanceDB 中的垃圾内容：`memory_recall query="*" limit=50`
3. 清除不相关的向量：`memory_forget id=<id>`
4. 将每日日志整合到 MEMORY.md

## WAL 协议（关键）

**预写日志：** 在响应之前写入状态，而不是之后。

| 触发条件 | 操作 |
|---------|--------|
| 用户陈述偏好 | 写入 SESSION-STATE.md → 然后响应 |
| 用户做出决策 | 写入 SESSION-STATE.md → 然后响应 |
| 用户给出截止日期 | 写入 SESSION-STATE.md → 然后响应 |
| 用户纠正你 | 写入 SESSION-STATE.md → 然后响应 |

**为什么？** 如果你先响应，然后在保存之前崩溃/压缩，上下文就会丢失。WAL 确保持久性。

## 示例工作流

```
用户："让我们在这个项目中使用 Tailwind，而不是 vanilla CSS"

Agent（内部）：
1. 写入 SESSION-STATE.md："决策：使用 Tailwind，不用 vanilla CSS"
2. 存储到 Git-Notes：关于 CSS 框架的决策
3. memory_store："用户偏好 Tailwind 胜过 vanilla CSS" importance=0.9
4. 然后响应："好的 —— 就用 Tailwind..."
```

## 维护命令

```bash
# 审计向量记忆
memory_recall query="*" limit=50

# 清除所有向量（核选项）
rm -rf ~/.openclaw/memory/lancedb/
openclaw gateway restart

# 导出 Git-Notes
python3 memory.py -p . export --format json > memories.json

# 检查记忆健康状态
du -sh ~/.openclaw/memory/
wc -l MEMORY.md
ls -la memory/
```

## 记忆失效的原因

了解根本原因有助于你修复它们：

| 失效模式 | 原因 | 修复方法 |
|--------------|-------|-----|
| 忘记一切 | `memory_search` 被禁用 | 启用并添加 OpenAI key |
| 文件未加载 | Agent 跳过读取记忆 | 添加到 AGENTS.md 规则 |
| 事实未被捕获 | 没有自动提取 | 使用 Mem0 或手动记录 |
| 子 agent 隔离 | 不继承上下文 | 在任务提示中传递上下文 |
| 重复错误 | 教训未被记录 | 写入 memory/lessons.md |

## 解决方案（按工作量排序）

### 1. 快速见效：启用 memory_search

如果你有 OpenAI key，启用语义搜索：

```bash
openclaw configure --section web
```

这会启用对 MEMORY.md + memory/*.md 文件的向量搜索。

### 2. 推荐：Mem0 集成

自动从对话中提取事实。减少 80% 的 token。

```bash
npm install mem0ai
```

```javascript
const { MemoryClient } = require('mem0ai');

const client = new MemoryClient({ apiKey: process.env.MEM0_API_KEY });

// 自动提取和存储
await client.add([
  { role: "user", content: "I prefer Tailwind over vanilla CSS" }
], { user_id: "ty" });

// 检索相关记忆
const memories = await client.search("CSS preferences", { user_id: "ty" });
```

### 3. 更好的文件结构（无依赖）

```
memory/
├── projects/
│   ├── strykr.md
│   └── taska.md
├── people/
│   └── contacts.md
├── decisions/
│   └── 2026-01.md
├── lessons/
│   └── mistakes.md
└── preferences.md
```

保持 MEMORY.md 作为摘要（<5KB），链接到详细文件。

## 立即修复清单

| 问题 | 修复方法 |
|---------|-----|
| 忘记偏好 | 在 MEMORY.md 中添加 `## Preferences` 部分 |
| 重复错误 | 将每个错误记录到 `memory/lessons.md` |
| 子 agent 缺乏上下文 | 在生成任务提示中包含关键上下文 |
| 忘记近期工作 | 严格的每日文件纪律 |
| 记忆搜索不工作 | 检查 `OPENAI_API_KEY` 是否已设置 |

## 故障排除

**Agent 在对话中持续遗忘：**
→ SESSION-STATE.md 未被更新。检查 WAL 协议。

**注入不相关的记忆：**
→ 禁用 autoCapture，提高 minImportance 阈值。

**记忆过大，召回缓慢：**
→ 运行卫生清理：清除旧向量，归档每日日志。

**Git-Notes 未持久化：**
→ 运行 `git notes push` 以与远程同步。

**memory_search 返回空：**
→ 检查 OpenAI API key：`echo $OPENAI_API_KEY`
→ 验证 openclaw.json 中 memorySearch 已启用

---

## 链接

- bulletproof-memory: https://clawdhub.com/skills/bulletproof-memory
- lancedb-memory: https://clawdhub.com/skills/lancedb-memory
- git-notes-memory: https://clawdhub.com/skills/git-notes-memory
- memory-hygiene: https://clawdhub.com/skills/memory-hygiene
- supermemory: https://clawdhub.com/skills/supermemory

---

*由 [@NextXFrontier](https://x.com/NextXFrontier) 构建 —— Next Frontier AI 工具包的一部分*
