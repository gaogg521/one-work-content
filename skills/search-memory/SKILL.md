---
name: search-memory
description: 面向 Openclaw 的本地优先记忆搜索与索引。当你需要 (1) 索引记忆文件、(2) 从 CLI 搜索记忆，或 (3) 为记忆查找绑定斜杠命令时使用。
tags:
- 搜索
---

# Search Memory

## 概述

索引本地记忆文件并运行带近期加权的快速关键词搜索。

## 快速开始

1) 构建/更新索引（增量缓存）：
```bash
scripts/index-memory.py
```

2) 搜索索引：
```bash
scripts/search-memory.py "your query" --top 5
```

## 说明

- 索引包含 `MEMORY.md` 以及 `memory/**/*.md`。
- 缓存存放于 `memory/cache/` 下。
- 搜索使用关键词评分 + 近期加权（最近 30/90 天）。
