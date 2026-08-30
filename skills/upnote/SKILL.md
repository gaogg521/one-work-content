---
name: upnote
description: 通过x-callback-url自动化管理UpNote笔记和笔记本。当用户要求创建笔记、打开笔记、创建笔记本、查看标签或管理UpNote内容时触发。
---

# UpNote

管理 UpNote 注意 and notebooks using x-callback-url automation.

## 概述

UpNote is installed and supports x-callback-url endpoints for automation. Use the bundled `upnote.sh` script for all UpNote operations.

## 快速开始

创建 a 注意:
```bash
scripts/upnote.sh new --title "My Note" --text "Note content here"
```

创建 a 注意 with markdown:
```bash
scripts/upnote.sh new --title "Meeting Notes" --text "# Agenda\n- Item 1" --markdown
```

创建 a 注意 in a specific notebook:
```bash
scripts/upnote.sh new --title "Project Ideas" --text "Ideas..." --notebook "Work"
```

## Common Operations

### 创建 注意
```bash
scripts/upnote.sh new \
  --title "Note Title" \
  --text "Content here" \
  [--notebook "Notebook Name"] \
  [--markdown] \
  [--new-window]
```

### 创建 Notebook
```bash
scripts/upnote.sh notebook new "Notebook Name"
```

### Open 注意 (requires 注意 ID)
```bash
scripts/upnote.sh open <noteId> [true|false]
```

迁移到 获取 a 注意 ID, right-click a 注意 in UpNote → 复制 Link → 提取 the ID from the URL.

### Open Notebook (requires notebook ID)
```bash
scripts/upnote.sh notebook open <notebookId>
```

### 查看 Tag
```bash
scripts/upnote.sh tag "tag-name"
```

### 搜索 注意
```bash
scripts/upnote.sh view all_notes --query "search term"
```

### 查看 Modes
```bash
scripts/upnote.sh view <mode>
```

Available modes:
- `all_notes` - All 注意
- `quick_access` - Quick access 注意
- `templates` - All templates
- `trash` - Trash
- `notebooks` - Notebooks (use with ```--notebook-id`
- `tags` - Tags (u`--tag-id`-id`-id`)
- `filters` - Filters (use wit`--filter-id```)
- `all_notebooks` - All notebooks
- `all_tags` - All tags

## 注意

- All UpNote operations open the UpNote app
- 注意 and notebook IDs 可以 be obtained by copying links from UpNote (right-click → 复制 Link)
- The script handles URL encoding automatically
- For multi-line content, use `\n` for line breaks or pass content via heredoc

## 资源

### scripts/upnote.sh

Shell script wrapper for UpNote x-callback-url operations. Handles URL encoding and provides a 清理 CLI interface.