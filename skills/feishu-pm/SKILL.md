---
name: feishu-pm
description: 飞书多维表格的项目管理 Skill。直接从智能体添加任务、列出记录和跟踪进度。
tags:
- 飞书
- pm
- project
- 效率
---

# Feishu Project Manager (PM)

Manage tasks and project records in Feishu Bitables directly from the CLI.

## Prerequisites

- Install `feishu-common` first.
- This skill depends on `../feishu-common/index.js` for token and API auth.

## Usage

### List Tasks

```bash
node skills/feishu-pm/index.js list --app <BITABLE_TOKEN> --table <TABLE_ID>
```

Options:
- `--view <id>`: Filter by View ID.
- `--limit <n>`: Limit number of records (default 20).
- `--json`: Output raw JSON instead of markdown table.

### Add Task

```bash
node skills/feishu-pm/index.js add --app <BITABLE_TOKEN> --table <TABLE_ID> --title "Fix Bug #123" --priority "High"
```

Options:
- `--desc <text>`: Task description.
- `--priority <text>`: Priority level (e.g. "High", "Medium", "Low").

## Notes

- Currently optimized for the "Iter 11" project structure (`需求`, `需求详述`, `优先级`).
- Edit `index.js` to customize field mapping for other projects.
