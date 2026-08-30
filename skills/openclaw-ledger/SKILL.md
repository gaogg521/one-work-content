---
name: openclaw-ledger
description: 面向 agent workspaces 的防篡改审计追踪。每个工作区变更都记录在 hash-chained log 中，如果有人篡改条目，链条就会断裂。
user-invocable: True
metadata: None
openclaw: None
emoji: 📒
os:
- darwin
- linux
- win32
requires: None
bins:
- python3
tags:
- AI
---

# OpenClaw Ledger

面向 agent workspaces 的防篡改审计追踪。每个工作区变更都记录在 hash-chained log 中 —— 如果有人篡改条目，链条就会断裂，你就会知道。

## 问题

Agents 修改文件、执行命令、安装技能 —— 并且没有留下可验证的记录。如果出了问题，你无法追踪发生了什么。如果日志存在，没有任何证据表明它们在事后没有被篡改。


## 命令

### 初始化

创建 ledger 并对当前工作区状态进行快照。

```bash
python3 {baseDir}/scripts/ledger.py init --workspace /path/to/workspace
```

### 记录变更

对当前状态进行快照并记录自上次记录以来的所有变更。

```bash
python3 {baseDir}/scripts/ledger.py record --workspace /path/to/workspace
python3 {baseDir}/scripts/ledger.py record -m "Installed new skill" --workspace /path/to/workspace
```

### 验证链条

验证 hash chain 是否完整 —— 没有条目被篡改。

```bash
python3 {baseDir}/scripts/ledger.py verify --workspace /path/to/workspace
```

### 查看日志

显示最近的 ledger 条目。

```bash
python3 {baseDir}/scripts/ledger.py log --workspace /path/to/workspace
python3 {baseDir}/scripts/ledger.py log -n 20 --workspace /path/to/workspace
```

### 快速状态

```bash
python3 {baseDir}/scripts/ledger.py status --workspace /path/to/workspace
```

## 工作原理

每个条目包含：
- Timestamp
- 前一个条目的 SHA-256 hash
- Event type 和 data（file changes、snapshots）

如果任何条目被修改、插入或删除，hash chain 就会断裂，`verify` 会检测到它。

## 退出代码

- `0` —— Clean / chain intact
- `1` —— 无 ledger 或轻微问题
- `2` —— Chain tampered / corrupt entries

## 无外部依赖

仅 Python 标准库。无需 pip install。无网络调用。所有内容都在本地运行。

## 跨平台

适用于 OpenClaw、Claude Code、Cursor 和任何使用 Agent Skills specification 的工具。
