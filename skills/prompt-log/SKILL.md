---
name: prompt-log
description: 从 AI 编码会话日志（Clawdbot、Claude Code、Codex）中提取对话记录。当要求从 .jsonl 会话文件中导出 prompt history、session logs 或 transcripts 时使用。
tags:
- AI
- 日志分析
---

# Prompt Log

## 快速开始

在会话文件上运行捆绑的脚本：

```bash
scripts/extract.sh <session-file>
```

## 输入

- **Session file**: 来自 Clawdbot、Claude Code 或 Codex 的 `.jsonl` 会话日志。
- **Optional filters**: `--after` 和 `--before` ISO timestamps。
- **Optional output**: `--output` 路径用于 markdown transcript。

## 输出

- 写入 markdown transcript。默认保存到当前项目的 `.prompt-log/YYYY-MM-DD-HHMMSS.md`。

## 示例

```bash
scripts/extract.sh ~/.codex/sessions/2026/01/12/abcdef.jsonl
scripts/extract.sh ~/.claude/projects/my-proj/xyz.jsonl --after "2026-01-12T10:00:00" --before "2026-01-12T12:00:00"
scripts/extract.sh ~/.clawdbot/agents/main/sessions/123.jsonl --output my-transcript.md
```

## 依赖

- 需要 PATH 中有 `jq`。
- 在 macOS 上如果可用则使用 `gdate`；否则回退到 `date`。
