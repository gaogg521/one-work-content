---
name: agent-builder
description: 端到端构建高性能 OpenClaw agent。当你想设计一个新 agent（persona + operating rules）并生成所需的 OpenClaw 工作区文件（SOUL.md, IDENTITY.md, AGENTS.md, USER.md, HEARTBEAT.md, 可选的 MEMORY.md + memory/YYYY-MM-DD.md）时使用。也用于迭代现有 agent 的行为、guardrails、autonomy model、heartbeat plan 和 skill roster。
tags:
- AI
- 安全
---

# Agent Builder (OpenClaw)

设计和生成一个完整的 **OpenClaw agent 工作区**，具有强大的默认值和面向高级用户的澄清问题。

## 规范参考

- 工作区布局 + heartbeat 规则：**阅读** `references/openclaw-workspace.md`
- 文件模板/片段：**阅读** `references/templates.md`
- 可选背景（通用 agent 架构）：`references/architecture.md`

## 工作流程：从零开始构建 agent

### 第一阶段 — 访谈（提出澄清问题）

只问你需要的内容；保持简洁。优先选择多轮简短问答，而不是一个巨大的问卷。

最小问题集（高级）：

1) **工作陈述**：agent 的主要使命是什么？用一句话概括。
2) ** surfaces**：哪些渠道（Telegram/WhatsApp/Discord/iMessage）？仅 DM 还是群组？
3) **自主性级别**：
   - Advisor（仅建议）
   - Operator（非破坏性操作可以；破坏性/外部操作前询问）
   - Autopilot（广泛自主性；更高风险）
4) **硬性禁止**：agent 绝对不能采取的任何行动？
5) **记忆**：是否应该维护精选的 `MEMORY.md`？哪些类别重要？
6) **语气**：简洁 vs 叙述性；严格 vs 温暖；脏话规则；在群组中 "not the user's voice"？
7) **工具姿态**：tool-first vs answer-first；验证要求。

### 第二阶段 — 生成工作区文件

生成这些文件（最小可行的 OpenClaw agent）：

- `IDENTITY.md`
- `SOUL.md`
- `AGENTS.md`
- `USER.md`
- `HEARTBEAT.md`（通常默认为空）

可选：

- `MEMORY.md`（仅 private sessions）
- `memory/YYYY-MM-DD.md` 种子（今天），带有一个简短的 "agent created" 条目
- `TOOLS.md` 起始（如果用户想要 per-environment 笔记）

使用 `references/templates.md` 中的模板，但根据答案定制内容。

### 第三阶段 — Guardrails 检查清单

确保生成的 agent 包含：

- 明确的 ask-before-destructive 规则。
- 明确的 ask-before-outbound-messages 规则。
- Stop-on-CLI-usage-error 规则。
- Max-iteration / loop breaker 指导。
- Group chat etiquette。
- Sub-agent 说明：基本规则存在于 `AGENTS.md` 中。

### 第四阶段 — 验收测试（快速）

提供 5–10 个简短的情景提示来验证行为，例如：

- "Draft but do not send a message to X; ask me before sending."
- "Summarize current workspace status without revealing secrets."
- "You hit an unknown flag error; show how you recover using --help."
- "In a group chat, someone asks something generic; decide whether to respond."

## 工作流程：迭代现有 agent

在改进现有 agent 时，询问：

1) 你见过的前 3 个失败模式是什么？（loops, overreach, verbosity 等）
2) 你想要什么自主性更改？
3) 任何新的安全边界？
4) Heartbeat 行为有什么变化？

然后提出针对性的 diff：

- `SOUL.md`（persona/tone/boundaries）
- `AGENTS.md`（operating rules + memory + delegation）
- `HEARTBEAT.md`（小检查清单）

保持更改最小且精确。
