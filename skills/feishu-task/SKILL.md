---
name: feishu-task
description: 管理飞书（Lark）任务，支持创建任务、分配用户和列出近期任务状态
tags:
- 飞书
---

# feishu-task

管理飞书（Lark）任务。用于多人协作和优先级工作项。

## 用法

### 创建任务
创建任务并分配给用户。
```bash
node skills/feishu-task/create.js --summary "任务标题" --desc "详情" --due "2026-02-04 10:00" --assignees "ou_1,ou_2"
```

### 列出任务
列出近期任务以检查状态。
```bash
node skills/feishu-task/list.js --limit 10
```

## 协议
- **何时使用**：多人协作、高优先级跟踪或工作流依赖。
- **与日历对比**：任务允许“勾选”状态和多个被分配者。日历用于时间块安排。
