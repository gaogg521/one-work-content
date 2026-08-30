---
name: agent-team-orchestration
description: 使用定义的角色、任务生命周期、交接协议和审查工作流程来编排多 agent 团队。使用时机：(1) 设置具有不同专业化的 2+ agent 团队，(2) 定义任务路由和生命周期（收件箱 → 规范 → 构建 → 审查 → 完成），(3) 创建 agent 之间的交接协议，(4) 建立审查和质量关卡，(5) 管理 agent 之间的异步通信和工件共享。
tags:
- AI
- Kubernetes
---

# Agent 团队编排

用于运行具有清晰角色、结构化任务流和质量关卡的 multi-agent 团队的生产 playbook。

## 快速开始：最小 2-Agent 团队

一个 builder 和一个 reviewer。最简单的有用团队。

### 1. 定义角色

```
Orchestrator (you) — 路由任务、跟踪状态、报告结果
Builder agent     — 执行工作、生成工件
```

### 2. 生成任务

```
1. 创建任务记录（文件、数据库或任务板）
2. 生成 builder，包含：
   - 任务 ID 和描述
   - 工件的输出路径
   - 交接说明（要生成什么、放在哪里）
3. 完成时：审查工件、标记完成、报告
```

### 3. 添加 Reviewer

```
Builder 生成工件 → Reviewer 检查它 → Orchestrator 发布或返回
```

这就是核心循环。下面的一切都是这个模式的扩展。

## 核心概念

### 角色

每个 agent 有一个主要角色。重叠会导致混乱。

| 角色 | 用途 | 模型指导 |
|------|---------|---------------|
| **Orchestrator** | 路由工作、跟踪状态、做出优先级判断 | High-reasoning model（处理判断） |
| **Builder** | 生成工件 — 代码、文档、配置 | 对于机械工作可以使用 cost-effective models |
| **Reviewer** | 验证质量、对差距提出异议 | High-reasoning model（捕获 builder 遗漏的东西） |
| **Ops** | Cron 作业、standups、健康检查、调度 | 最便宜的可靠 model |

→ *在定义新团队或添加 agent 时阅读 [references/team-setup.md](references/team-setup.md)。*

### 任务状态

每个任务都经过定义的生命周期：

```
Inbox → Assigned → In Progress → Review → Done | Failed
```

**规则：**
- Orchestrator 拥有状态转换 — 不要依赖 agent 更新自己的状态
- 每次转换都会收到评论（who, what, why）
- Failed 是一个有效的结束状态 — 捕获原因并继续前进

→ *在设计任务流或调试卡住的任务时阅读 [references/task-lifecycle.md](references/task-lifecycle.md)。*

### 交接

当工作在 agent 之间传递时，交接消息包括：

1. **做了什么** — 更改/输出的摘要
2. **工件在哪里** — 确切的文件路径
3. **如何验证** — 测试命令或验收标准
4. **已知问题** — 任何不完整或有风险的东西
5. **下一步是什么** — 接收 agent 的明确下一步行动

糟糕的交接：*"Done, check the files."*
好的交接：*"Built auth module at `/shared/artifacts/auth/`. Run `npm test auth` to verify. Known issue: rate limiting not implemented yet. Next: reviewer checks error handling edge cases."*

### 审查

跨角色审查防止质量漂移：

- **Builders 审查规范** — "Is this feasible? What's missing?"
- **Reviewers 检查构建** — "Does this match the spec? Edge cases?"
- **Orchestrator 审查优先级** — "Is this the right work right now?"

跳过审查步骤，质量在 3-5 个任务内下降。每次都这样。

→ *在设置 agent 通信渠道时阅读 [references/communication.md](references/communication.md)。*
→ *在实施特定工作流程时阅读 [references/patterns.md](references/patterns.md)。*

## 参考文件

| 文件 | 何时阅读... |
|------|-------------|
| [team-setup.md](references/team-setup.md) | 定义 agent、角色、模型、工作区 |
| [task-lifecycle.md](references/task-lifecycle.md) | 设计任务状态、转换、评论 |
| [communication.md](references/communication.md) | 设置异步/同步通信、工件路径 |
| [patterns.md](references/patterns.md) | 实施特定工作流程（规范→构建→测试、并行研究、升级） |

## 常见陷阱

### 没有清晰的工件输出路径就生成

Agent 产生了出色的工作，但你找不到它。始终在生成提示中指定确切的输出路径。使用具有可预测结构的共享工件目录。

### 没有审查步骤 = 质量漂移

"It's a small change, skip review." 这样做三次，你就有累积错误。每个工件至少得到一双没有生成它的眼睛。

### Agent 不对任务进度进行评论

沉默的 agent 造成协调盲点。要求在以下位置评论：start, blocker, handoff, completion。如果一个 agent 沉默了，假设它卡住了。

### 在分配前不验证 agent 能力

将基于浏览器的测试分配给没有浏览器访问权限的 agent。将图像工作分配给仅文本的 model。在路由之前检查能力。

### Orchestrator 做执行工作

Orchestrator 路由和跟踪 — 它不构建。当你开始 "just quickly doing this one thing" 的那一刻，你就失去了对其余团队的监督。

## 何时不使用本技能

- **单 agent 设置** — 只需遵循标准 AGENTS.md 约定。团队编排增加了 solo agent 不需要的开销。
- **一次性任务委派** — 直接使用 `sessions_spawn`。本技能用于具有多次交接的持续工作流程。
- **简单的问题路由** — 如果你只是将问题转发给专家，那是一个消息，而不是一个工作流程。

本技能用于 **持续的团队工作流程** — 具有多次任务的重复协作模式，其中 agent 在多个任务中相互依赖对方的输出。
