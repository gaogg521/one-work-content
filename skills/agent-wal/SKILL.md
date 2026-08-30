---
name: agent-wal
description: 用于 agent 状态持久化的 Write-Ahead Log 协议。防止在对话压缩期间丢失更正、决策和上下文。使用时机：(1) 收到用户纠正时 — 在响应之前记录它，(2) 做出重要决策或分析时 — 在继续之前记录它，(3) 预压缩内存刷新 — 将工作缓冲区刷新到 WAL，(4) session 开始时 — 重放未应用的 WAL 条目以恢复丢失的上下文，(5) 任何时候你想确保某些东西在压缩后仍然存在。
tags:
- AI
---

# Agent WAL（Write-Ahead Log）

在响应之前将重要状态写入磁盘。防止 #1 agent 失败模式：在压缩期间丢失更正和上下文。

## 核心规则

**在响应之前写入。** 如果某件事值得记住，先 WAL 它。

## 何时 WAL

| 触发器 | 操作类型 | 示例 |
|---------|------------|---------|
| 用户纠正你 | `correction` | "No, use Podman not Docker" |
| 你做出关键决策 | `decision` | "Using CogVideoX-2B for text-to-video" |
| 重要分析/结论 | `analysis` | "WAL/VFM patterns should be core infra not skills" |
| 状态更改 | `state_change` | "GPU server SSH key auth configured" |
| 用户说 "remember this" | `correction` | 无论他们说了什么 |

## 命令

所有命令通过 `scripts/wal.py`（相对于本技能目录）：

```bash
# 在响应之前写入
python3 scripts/wal.py append agent1 correction "Use Podman not Docker for all EvoClaw tooling"
python3 scripts/wal.py append agent1 decision "CogVideoX-5B with multi-GPU via accelerate"
python3 scripts/wal.py append agent1 analysis "Signed constraints prevent genome tampering"

# 工作缓冲区（对话期间批量写入，在压缩前刷新）
python3 scripts/wal.py buffer-add agent1 decision "Some decision"
python3 scripts/wal.py flush-buffer agent1

# Session 开始时：重放丢失的上下文
python3 scripts/wal.py replay agent1

# 应用重放的条目后
python3 scripts/wal.py mark-applied agent1 <entry_id>

# 维护
python3 scripts/wal.py status agent1
python3 scripts/wal.py prune agent1 --keep 50
```

## 集成点

### Session 开始时
1. 运行 `replay` 以获取未应用的条目
2. 将摘要读入你的上下文
3. 在合并它们后将条目标记为已应用

### 用户纠正时
1. 使用 action_type `correction` 运行 `append` BEFORE 响应
2. 然后用纠正后的行为响应

### 预压缩刷新时
1. 运行 `flush-buffer` 以持久化任何缓冲的条目
2. 然后照常写入每日记忆文件

### 对话期间
对于不太重要的项目，使用 `buffer-add` 批量写入。缓冲区在 `flush-buffer` 时刷新到 WAL（在预压缩期间调用）或手动刷新。

## 存储

WAL 文件：`~/clawd/memory/wal/<agent_id>.wal.jsonl`
缓冲区文件：`~/clawd/memory/wal/<agent_id>.buffer.jsonl`

条目是 append-only JSONL。每个条目：
```json
{"id": "abc123", "timestamp": "ISO8601", "agent_id": "agent1", "action_type": "correction", "payload": "Use Podman not Docker", "applied": false}
```
