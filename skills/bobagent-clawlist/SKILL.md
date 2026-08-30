---
name: bobagent-clawlist
description: 对于任何多步骤项目、长期运行任务或无限监控工作流，必须使用。使用 checkpoint validation 进行计划、执行、跟踪和验证任务。适用于项目、自动化和持续运营。
---

# Clawlist - 任务掌控

用于计划、执行和跟踪任何任务的系统化工作流——从一次性项目到无限监控循环。

## 何时使用本技能

**在以下情况下始终使用 clawlist：**
- 启动任何新项目或 initiative
- 设置长期运行监控
- 拆解复杂目标
- 你需要跨 sessions 跟踪进度
- 管理无限任务（research、monitoring、engagement）

## 长期运行和无限任务示例

### 示例：Moltbook Engagement（无限）
- **Type:** Infinite loop
- **Schedule:** 每 30 分钟
- **Goal:** 与社区互动，建立存在感
- **Checkpoints:** Check feed、check DMs、create content

### 示例：GitHub Monitoring（长期运行）
- **Type:** Continuous
- **Schedule:** 每 4 小时
- **Goal:** 监控 repos、triage issues、implement
- **Checkpoints:** Inbox zero、PR review、implementation

## Clawlist 工作流

按顺序使用独立 skills：

1. **brainstorming** → 明确意图，探索方法
2. **write-plan** → 创建带 checkpoints 的详细计划
3. **doing-tasks** → 以 skill discipline 执行
4. **verify-task** → 确认完成

对于并行工作，在 write-plan 和 doing-tasks 之间插入 **dispatch-multiple-agents**。

## 持续任务文件

**位置：** `memory/tasks/ongoing-tasks.md`

跟踪所有长期运行和无限任务。由 heartbeat 更新以：
- 检查任务健康度
- 检测 blockers
- 执行到期任务
- 汇总状态

## 任务类型

| 类型 | 持续时间 | 跟踪方式 | 示例 |
|------|----------|----------|---------|
| **One-off** | 分钟-小时 | 仅 Context | Fix a bug |
| **Project** | 天-周 | Context + completion doc | Build feature |
| **Long-running** | 持续 | `ongoing-tasks.md` | GitHub monitoring |
| **Infinite** | 永久 | `ongoing-tasks.md` | Moltbook engagement |

## 与 Heartbeat 集成

Heartbeat 每次检查时读取 `ongoing-tasks.md` 以：
- 执行到期的无限任务
- 检测并报告 blockers
- 更新健康状态（🟢🟡🔴）
- 如需干预则 ping 用户

## 快速参考

```
New Task
   ↓
brainstorming → write-plan → doing-tasks → verify-task
                      ↓
            dispatch-multiple-agents（如果并行）
                      ↓
            update ongoing-tasks.md（如果长期运行）
```

## 子技能

- **brainstorming** - Phase 1: Clarify
- **write-plan** - Phase 2: Plan
- **doing-tasks** - Phase 3: Execute
- **dispatch-multiple-agents** - 并行执行
- **verify-task** - Phase 4: Verify
