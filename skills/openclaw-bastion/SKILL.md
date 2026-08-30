---
name: openclaw-bastion
description: 为 agent 工作空间提供运行时 prompt injection 防御，扫描输入输出边界中的隐藏指令、Unicode 陷阱与危险命令。触发词：prompt injection 防御、工作空间安全(workspace security)、Bastion、输入扫描(input scan)
user-invocable: True
metadata: None
openclaw: None
emoji: 🏛️
os:
- darwin
- linux
- win32
requires: None
bins:
- python3
tags:
- AI
- 安全
---

# OpenClaw Bastion

用于 agent 工作空间的运行时 prompt injection 防御。当其他工具监视工作空间身份文件时，Bastion 保护输入/输出边界——agent 读取的文件、web 内容、API 响应和用户提供的文档。

## 为什么重要

Agents 处理来自多个来源的内容：本地文件、API 响应、web 页面、用户上传。任何这些都可能包含 prompt injection 攻击——操纵 agent 行为的隐藏指令。Bastion 在 agent 行动前扫描此内容。


## 命令

### 扫描注入

扫描文件或目录中的 prompt injection 模式。检测指令覆盖、系统 prompt 标记、隐藏 Unicode、markdown 外泄、HTML 注入、shell 注入、编码 payload、分隔符混淆、多轮操纵和危险命令。

如果未指定目标，则扫描整个工作空间。

```bash
python3 {baseDir}/scripts/bastion.py scan
```

扫描特定文件或目录：

```bash
python3 {baseDir}/scripts/bastion.py scan path/to/file.md
python3 {baseDir}/scripts/bastion.py scan path/to/directory/
```

### 快速文件检查

快速的单文件注入检查。与 `scan` 使用相同的检测模式，针对单个文件。

```bash
python3 {baseDir}/scripts/bastion.py check path/to/file.md
```

### 边界分析

分析整个工作空间的内容边界安全性。识别：
- 包含混合可信/不可信内容的 agent 指令文件
- 可写的指令文件（被 compromise skills 攻击的攻击面）
- 每个关键文件的 blast radius 评估

```bash
python3 {baseDir}/scripts/bastion.py boundaries
```

### 命令白名单

显示当前命令白名单和黑名单策略。如果不存在则创建默认的 `.bastion-policy.json`。

```bash
python3 {baseDir}/scripts/bastion.py allowlist
python3 {baseDir}/scripts/bastion.py allowlist --show
```

策略文件定义哪些命令被认为是安全的以及哪些模式被阻止。直接编辑 JSON 文件以自定义。Bastion Pro 通过 hooks 在运行时强制执行此策略。

### 状态

工作空间注入防御态势的快速摘要：扫描的文件、按严重度分类的发现、边界安全性和整体态势评级。

```bash
python3 {baseDir}/scripts/bastion.py status
```

## 工作空间自动检测

如果省略 `--workspace`，脚本尝试：
1. `OPENCLAW_WORKSPACE` 环境变量
2. 当前目录（如果存在 `AGENTS.md`）
3. `~/.openclaw/workspace`（默认）

## 检测内容

| 类别 | 模式 | 严重度 |
|----------|----------|----------|
| **指令覆盖** | "ignore previous", "disregard above", "you are now", "new system prompt", "forget your instructions", "override safety", "act as if no restrictions", "entering developer mode" | CRITICAL |
| **系统 prompt 标记** | `<system>`, `[SYSTEM]`, `<<SYS>>`, `<\|im_start\|>system`, `[INST]`, `### System:` | CRITICAL |
| **隐藏指令** | 多轮操纵 ("in your next response, you must")、隐形模式 ("do not tell the user") | CRITICAL |
| **HTML 注入** | `<script>`, `<iframe>`, `<img onerror=>`, hidden divs, `<svg onload=>` | CRITICAL |
| **Markdown 外泄** | Image tags with encoded data in URLs | CRITICAL |
| **危险命令** | `curl \| bash`, `wget \| sh`, `rm -rf /`, fork bombs | CRITICAL |
| **Unicode 技巧** | Zero-width 字符、RTL overrides、隐形格式化 | WARNING |
| **Homoglyph 替换** | Cyrillic/Latin lookalikes 混入 ASCII 文本 | WARNING |
| **Base64 payloads** | 代码块外的大型编码 blob | WARNING |
| **Shell 注入** | 代码块外的 `$(command)` subshell 执行 | WARNING |
| **分隔符混淆** | 带有注入内容的虚假代码块边界 | WARNING |

## 上下文感知扫描

- 围栏代码块 (` ``` `) 内的模式被跳过以避免误报
- 基于发现数量和严重度的每文件风险评分
- 自我排除：Bastion 跳过自己的 skill 文件（描述注入模式）

## 退出码

| Code | 含义 |
|------|---------|
| 0 | 干净，无问题 |
| 1 | 检测到警告（建议审查） |
| 2 | 关键发现（需要采取行动） |

## 无外部依赖

仅 Python 标准库。无需 pip install。无网络调用。一切在本地运行。

## 跨平台

适用于 OpenClaw、Claude Code、Cursor 和任何使用 Agent Skills 规范的工具。
