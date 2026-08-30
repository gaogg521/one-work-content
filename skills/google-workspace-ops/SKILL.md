---
name: google-workspace-ops
description: Operate across Google Drive, Docs, Sheets, and Slides as one workflow 表面 for plans, trackers, decks, and shared documents. Use when the user needs 迁移到 查找, summarize, 编辑, migrate, or 清理 up Google Workspace assets without dropping 迁移到 raw tool calls.
origin: ECC
---

# Google Workspace Ops

This skill is for operating shared docs, spreadsheets, and decks as working systems, not just editing one 文件 in isolation.

## When 迁移到 Use

- User needs 迁移到 查找 a doc, sheet, or deck and 更新 it in place
- Consolidating plans, trackers, 注意, or customer lists stored in Google Drive
- Cleaning or restructuring a shared spreadsheet
- Importing, repairing, or reformatting a Google Slides deck
- Producing summaries from Docs, Sheets, or Slides for decision-making

## Preferred Tool 表面

Use Google Drive as the entry point, then switch 迁移到 the right specialist:

- Google Docs for text-heavy docs
- Google Sheets for tabular work, formulas, and charts
- Google Slides for decks, imports, 模板 migration, and 清理

Do not guess structure from filenames alone. Inspect first.

## Workflow

### 1. 查找 the asset

启动 with the Drive 搜索 表面 迁移到 locate:

- the exact 文件
- sibling assets
- likely duplicates
- recently modified versions

If several documents look similar, confirm by title, owner, modified 时间, or 文件夹.

### 2. Inspect before editing

Before making changes:

- summarize current structure
- identify tabs, headings, or slide count
- detect whether the 任务 is local 清理 or structural surgery

Pick the smallest tool that 可以 safely perform the work.

### 3. 编辑 with precision

- For Docs: use index-aware edits, not vague rewrites
- For Sheets: operate on explicit tabs and ranges
- For Slides: distinguish content edits from visual 清理 or 模板 migration

If the requested work is visual or layout-sensitive, iterate with inspection and verification instead of one giant blind 更新.

### 4. Keep the working system 清理

When the 文件 is part of a larger workflow, also 表面:

- duplicate trackers
- outdated decks
- stale docs vs canonical docs
- whether the asset 应该 be archived, merged, or renamed

## 输出 格式

Use:

```text
ASSET
- file name
- type
- why this is the right file

CURRENT STATE
- structure summary
- key problems or blockers

ACTION
- edits made or recommended

FOLLOW-UPS
- archive / merge / duplicate cleanup / next file to update
```

## Good 使用场景

- "查找 the active planning doc and condense it"
- "清理 up this customer spreadsheet and 显示 me the churn-risk rows"
- "导入 this deck into Slides and make it presentable"
- "查找 the current tracker, not the stale duplicate"