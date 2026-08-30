---
name: openclaw-marshal
description: 为工作区定义安全策略并审计合规性。根据 command、network 和 data handling 规则检查已安装的技能，生成审计就绪的合规报告。
user-invocable: True
metadata: None
openclaw: None
emoji: 📋
os:
- darwin
- linux
- win32
requires: None
bins:
- python3
tags:
- 网络
---

# OpenClaw Marshal

为你的工作区定义安全策略并审计合规性。根据 command、network 和 data handling 规则检查已安装的技能。生成审计就绪的合规报告。

## 为什么这很重要

Agent workspaces 积累执行命令、访问网络和处理数据的技能。如果没有定义的安全策略，就无法知道已安装的技能是否符合你组织的要求 —— 或者你的工作区本身是否满足基本的安全卫生标准。

本技能让你可以一次性定义策略并审计所有内容。


## 命令

### 初始化策略

创建一个带有合理默认值的默认安全策略文件（`.marshal-policy.json`）。

```bash
python3 {baseDir}/scripts/marshal.py policy --init --workspace /path/to/workspace
```

### 显示策略

显示当前活动的策略。

```bash
python3 {baseDir}/scripts/marshal.py policy --show --workspace /path/to/workspace
```

### 策略摘要

已加载策略规则的快速概览。

```bash
python3 {baseDir}/scripts/marshal.py policy --workspace /path/to/workspace
```

### 完整合规审计

根据活动策略审计所有已安装的技能和工作区配置。报告合规分数、违规和建议。

```bash
python3 {baseDir}/scripts/marshal.py audit --workspace /path/to/workspace
```

### 检查特定技能

根据策略检查单个技能。报告每条规则的通过/失败。

```bash
python3 {baseDir}/scripts/marshal.py check openclaw-warden --workspace /path/to/workspace
```

### 生成合规报告

生成格式化的、可复制的合规报告，适合审计文档。

```bash
python3 {baseDir}/scripts/marshal.py report --workspace /path/to/workspace
```

### 快速状态

单行摘要：策略已加载、合规分数、严重违规计数。

```bash
python3 {baseDir}/scripts/marshal.py status --workspace /path/to/workspace
```

## 工作区自动检测

如果省略 `--workspace`，脚本会尝试：
1. `OPENCLAW_WORKSPACE` 环境变量
2. 当前目录（如果存在 AGENTS.md）
3. `~/.openclaw/workspace`（默认）

## 检查内容

| Category | Checks | Severity |
|----------|--------|----------|
| **Command Safety** | Dangerous patterns (eval、exec、pipe-to-shell、rm -rf /) | CRITICAL |
| **Command Policy** | 来自策略的 Blocked 和 review-required commands | HIGH/MEDIUM |
| **Network Policy** | Domain allow/blocklists、suspicious TLD patterns | CRITICAL/HIGH |
| **Data Handling** | Secret scanner 已安装、PII scanner 已配置 | HIGH/MEDIUM |
| **Workspace Hygiene** | .gitignore、audit trail (ledger)、skill signing (signet) | HIGH/MEDIUM |
| **Configuration** | Debug modes、verbose logging 保持启用 | LOW |

## 策略格式

`.marshal-policy.json` 文件定义所有规则：

- **commands.allow** —— 允许的 binaries
- **commands.block** —— 阻止的 command patterns
- **commands.review** —— 需要人工审查的 commands
- **network.allow_domains** —— 允许的网络 domains
- **network.block_domains** —— 阻止的 domains
- **network.block_patterns** —— Wildcard domain blocks（例如 `*.tk`）
- **data_handling.pii_scan** —— 要求 PII scanning
- **data_handling.secret_scan** —— 要求 secret scanning
- **workspace.require_gitignore** —— 要求 .gitignore
- **workspace.require_audit_trail** —— 要求 ledger
- **workspace.require_skill_signing** —— 要求 signet

## 退出代码

- `0` —— 合规，无问题
- `1` —— 需要审查（中/高发现）
- `2` —— 检测到严重违规

## 无外部依赖

仅 Python 标准库。无需 pip install。无网络调用。所有内容都在本地运行。

## 跨平台

适用于 OpenClaw、Claude Code、Cursor 和任何使用 Agent Skills specification 的工具。
