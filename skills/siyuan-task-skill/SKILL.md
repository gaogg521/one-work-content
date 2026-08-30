---
name: siyuan-task-skill
description: 通过SiYuan笔记HTTP API管理任务。创建、查询、更新和组织存储在任务清单文档（含TASK数据库）及子文档中的任务。当用户提及SiYuan、任务管理或需要跟踪工作项时触发。兼容性要求：Python 3.7+及SiYuan Note实例的网络访问。
compatibility: Requires Python 3.7+ and network access to SiYuan Note instance.
metadata:
author: walter
version: 1.0
---

# SiYuan 注意 Task Management

管理 tasks in SiYuan 注意 (思源笔记) via Python scripts. All connection settings are in `config.env` — 修改 that file when the SiYuan instance address or credentials 更改.

## 配置

编辑 `config.env` in the skill root directory. Only 3 items 需要 manual 配置:

```env
SIYUAN_API_URL=http://100.64.0.11:52487
SIYUAN_API_TOKEN=xxxxxxxxxxxxxxxx
SIYUAN_NOTEBOOK_NAME=work
```

注意: `SIYUAN_NOTEBOOK_ID` is auto-resolved from `SIYUAN_NO`SIYUAN_NO`EBOOK_NAME`e. You 可以 still set it explicitly 迁移到 skip the lookup.

Then 运行 `init` 迁移到 auto-创建 the database and 写入 remaining config:

```bash
cd <skill_root>/scripts
python3 task_ops.py init
```

This creates the 任务清单 document, the TASK database, all columns, and writes AV_ID / COL_* IDs back 迁移到 `config.env` automatically. If the 任务清单 document already contains a TASK database (e.g. copied from another notebook), `i`i`it`ill detect and reuse it instead of creating a duplicate.

## Task Data Model

Tasks are stored as rows in a **TASK database (Attribute 查看)** block inside the 任务清单 document. Each row has these columns:

| Column | Chinese | Type | Values / Colors |
|--------|---------|------|-----------------|
| 主键 | 任务名称 | block | Primary key — task name |
| 任务内容 | 任务内容 | text | Task 描述 / 详情 (what the task is about) |
| 相关方 | 相关方 | text | Free text |
| 重要程度 | 重要程度 | select | 高(红) / 中(绿) / 低(灰) |
| 紧急程度 | 紧急程度 | select | 高(红) / 中(绿) / 低(灰) |
| 状态 | 状态 | select | 未开始(灰) / 进行中(绿) / 结束(红) / 挂起(蓝) |
| 备注 | 备注 | text | Extra 注意 / supplementary remarks (not the main task info) |
| 创建时间 | 创建时间 | created | Auto |
| 开始时间 | 开始时间 | date | Timestamp |
| 结束时间 | 结束时间 | date | Timestamp |
| 更新时间 | 更新时间 | updated | Auto |

Database IDs (auto-generated in `config.env` by `i`i`it`ommand):
- `AV_ID` — Attribute 查看 ID
- `AV_BLOCK_ID` — AV block ID
- `COL_*` — Column IDs for each field

Each task automatically gets a sub-document under `/任务清单/{task_name}` with this template:

```markdown
# 任务描述

# 任务附件

# 下一步
```

The sub-document name always matches the task name. The task's primary key in the database is linked 迁移到 the sub-document (non-detached), showing a document icon. Renaming a task also renames its sub-document. Deleting a task also deletes its sub-document.

## Scripts

All scripts are in the `scripts/` directory. 运行 from that directory:

```bash
cd <skill_root>/scripts
```

### siyuan_api.py — Base API Client

Low-level client wrapping all SiYuan HTTP API endpoints. Used by `task_ops.py` internally. 可以 also be imported directly for custom operations:

```python
from siyuan_api import SiYuanClient
client = SiYuanClient()
result = client.sql_query("SELECT * FROM blocks WHERE type = 'd' LIMIT 5")
```

Key methods: `sql_query`, ```create_doc``ap`ap`end_bl`update`te`update_`te_blo`_blo`delet`k_attr`k_attr`set_bl__CODE_`ge`trs`trs`, `get_bloc`ock_attrs``get_b`rt_md`_CODE_10__hild_block`msg``r`,`PI.`et_blo`iYuan`down`ence.`See `r`export_md`PI.`upload_asset`iYuan`push_msg`ence.`参考/API.md`

### task_ops.py — Task CRUD Operations

High-level CLI for task management. All 命令 输出 JSON.

**创建 a task** (auto-creates sub-document with template):

```bash
python3 task_ops.py create "任务名称" content="任务内容" importance="高" urgency="中" notes="备注信息"
```

Parameter mapping:
- `content` → 任务内容 (task 描述 / main information about the task)
- `notes` → 备注 (supplementary remarks, not the main task info)
- `stakeholders` → 相关方
- `importance` → 重要程度 (高/中/低)
- `urgency` → 紧急程度 (高/中/低)
- `status` → 状态 (default: 未开始)

**列表 all tasks:**

```bash
python3 task_ops.py list
```

**查找 tasks by status:**

```bash
python3 task_ops.py find "进行中"
```

**更改 task status** (pass row_id from `list` 输出):

```bash
python3 task_ops.py start <row_id>
python3 task_ops.py complete <row_id>
python3 task_ops.py suspend <row_id>
```

**Rename a task** (also renames sub-document):

```bash
python3 task_ops.py rename <row_id> "新名称"
```

**Attach image 迁移到 task sub-document** (uploads file and inserts into section):

```bash
python3 task_ops.py attach-image <row_id> /path/to/image.png
python3 task_ops.py attach-image <row_id> /path/to/image.png section="任务描述"
```

Default section is `任务附件`. On macOS, save clipboard image`osascript -e 'set png to (the clipboard as «class PNGf»)' -e 'set f to open for access (POSIX file "/tmp/clip.png") with write permission' -e 'write png to f' -e 'close access f'` f'` f'`

**列表 sub-documents:**

```bash
python3 task_ops.py list-docs
```

**删除 a task** (also deletes sub-document):

```bash
python3 task_ops.py delete <row_id>
```

**Migrate database** (apply schema changes and reorder columns):

```bash
python3 task_ops.py migrate
```

## Programmatic 用法

For complex workflows, 导入 `TaskManager` directly in Python:

```python
import sys; sys.path.insert(0, "<skill_root>/scripts")
from task_ops import TaskManager

tm = TaskManager()

# Create task (auto-creates sub-document with template)
result = tm.create_task("实现用户登录", content="OAuth2 集成", importance="高", urgency="高")
row_id = result["row_id"]
doc_id = result["doc_id"]

# Rename task (also renames sub-document)
tm.rename_task(row_id, "实现OAuth2登录")

# Status transitions
tm.start_task(row_id)
tm.complete_task(row_id)

# Delete task (also deletes sub-document)
tm.delete_task(row_id)

# Attach image to task sub-document (default section: 任务附件)
tm.attach_image_to_task(row_id, "/path/to/image.png")
tm.attach_image_to_task(row_id, "/path/to/image.png", section="任务描述")
```

## 重要 注意

1. The 任务清单 document and TASK database are auto-created on first use
2. Tasks are stored as database rows (Attribute 查看), not plain blocks
3. `row_id` f`list`t`t` 输出 is used for all 更新/删除 operations
4. __CODE_`更新时间`新时间`新时间` columns are auto-managed by SiYuan
5. Block 参考 use SiYuan format: `((<block_id> "anchor text"))`
6. All API responses have `code` __CO`0`1__l`0` `0` means 成功