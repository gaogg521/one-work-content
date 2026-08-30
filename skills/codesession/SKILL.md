---
name: codesession
description: 使用 codesession-cli 跟踪 agent session 成本、文件变更和 git commits。强制执行预算限制，并通过 web dashboard 提供详细的 session 分析。v1.9.6 - Agent 名称跟踪、并发 sessions、web dashboard。
metadata: None
openclaw: None
homepage: https://github.com/brian-mwirigi/codesession-cli
install:
- bins:
  - cs
  id: npm
  kind: node
  label: Install codesession-cli (npm)
  package: codesession-cli
requires: None
bins:
- cs
tags:
- AI
- Git
- Grafana
- Web
---

# Session Cost Tracking (codesession-cli)

跟踪 agent session 成本、文件变更和 git commits。强制执行预算限制，并通过完整的 web dashboard 提供详细的 session 分析。

**最新版本：v1.9.6** - Agent 名称跟踪、并发 session 支持、带分析的 web dashboard。

[npm](https://www.npmjs.com/package/codesession-cli) • [GitHub](https://github.com/brian-mwirigi/codesession-cli) • [Changelog](https://github.com/brian-mwirigi/codesession-cli/blob/main/CHANGELOG.md)

## 安装

```bash
# 1. 从 npm 全局安装 CLI
npm install -g codesession-cli

# 2. 安装 OpenClaw skill
clawhub install codesession
```

安装后，`cs` 命令全局可用。OpenClaw agent 将自动使用它来跟踪 sessions。

> **要求：** Node.js 18+ 和 C/C++ build tools（需要编译 embedded SQLite module）。
>
> | OS | 安装 build tools |
> |---|---|
> | **Ubuntu/Debian** | `sudo apt-get install -y build-essential python3` |
> | **macOS** | `xcode-select --install` |
> | **Windows** | `npm install -g windows-build-tools` 或安装 Visual Studio Build Tools |
> | **Alpine** | `apk add build-base python3` |
>
> 数据本地存储在 `~/.codesession/sessions.db`。

## 何时使用

- **始终** 在多步骤任务开始时启动 tracked session
- **始终** 在每次 API 调用后记录 AI 使用情况
- **始终** 在任务完成时结束 session
- 在昂贵操作前检查预算
- 使用 `cs dashboard` 在浏览器中审查 session 数据

## 命令

### 开始跟踪
```bash
# Agent 模式（始终使用 --json 以获得结构化输出）：
cs start "task description" --json --close-stale

# 如果 session 之前未关闭则恢复（例如崩溃后）：
cs start "task description" --json --resume

# 人类/交互模式（保持运行并带有 live file watcher）：
cs start "task description"
```

> **Agent 模式 vs 交互模式：** 使用 `--json` 时，session 在数据库中创建，打印 JSON 并立即退出——session 保持 "active"，并在运行 `cs end` 时跟踪 git 变更。不使用 `--json` 时，进程保持运行并带有 live file watcher 和 git commit poller，直到你按 Ctrl+C 或在另一个终端运行 `cs end`。

### 记录 AI 使用情况（每次 API 调用后）
```bash
# 使用 granular tokens（成本从内置定价自动计算）：
cs log-ai -p anthropic -m claude-sonnet-4 --prompt-tokens 8000 --completion-tokens 2000 --json

# 使用 agent 名称跟踪（v1.9.1 新增）：
cs log-ai -p anthropic -m claude-sonnet-4 --prompt-tokens 8000 --completion-tokens 2000 --agent "Code Review Bot" --json

# 使用手动成本：
cs log-ai -p anthropic -m claude-opus-4-6 -t 15000 -c 0.30 --json

# 使用所有字段：
cs log-ai -p openai -m gpt-4o --prompt-tokens 5000 --completion-tokens 1500 -c 0.04 --agent "Research Agent" --json
```
Providers: `anthropic`、`openai`、`google`、`mistral`、`deepseek`
成本从可配置的 pricing table 自动计算（17+ 内置 models）。使用 `cs pricing list --json` 查看已知 models。如果 model 未知，手动提供 `-c <cost>`。

**Agent 名称（可选）：** 使用 `--agent "Agent Name"` 跟踪哪个 agent 执行了工作。非常适合 multi-agent systems、A/B testing 和按 agent 归因成本。Agent 名称出现在 dashboard 中，可用于按 agent 过滤/分析成本。

### 检查当前状态
```bash
cs status --json
```
返回包含当前 session 成本、tokens、文件变更、持续时间的 JSON。所有 JSON 响应都包含 `schemaVersion` 和 `codesessionVersion` 字段。

### 结束 session 并获取摘要
```bash
cs end -n "completion notes" --json
```
结束时，codesession 自动扫描自 session 开始以来的所有文件变更和 commits——即使使用了 `--json` 模式（无需 live watcher）。

### Web Dashboard
```bash
cs dashboard
# 打开 http://localhost:3737 并显示完整的 analytics UI

cs dashboard --port 4000       # 自定义端口
cs dashboard --no-open         # 不自动打开浏览器
```

Dashboard 显示：
- **Overview** — KPIs、每日成本/token 趋势、支出预测、成本速度
- **Sessions** — 可搜索/排序的表格、每个 session 的详情及时间线、文件、commits、AI 调用、notes
- **Models** — 按 model 和按 provider 的成本细分、token 比例、使用图表
- **Insights** — 文件热点、活动 heatmap、项目细分、pricing table

### 查看 session 详情
```bash
cs show --json --files --commits
```

### 查看历史统计
```bash
cs stats --json
```

### 导出 sessions
```bash
cs export --format json --limit 10
cs export --format csv
```

### 添加 notes / annotations
```bash
cs note "Starting refactor phase"
cs note "Tests passing, moving to cleanup"
```
带时间戳的 annotations 出现在 `cs show --json` 的 `annotations` 下。

### 恢复 stale sessions
```bash
cs recover --max-age 12
```
自动结束任何超过 12 小时的 active sessions。

## Agent 工作流

Agents 应在每个命令上 **始终** 使用 `--json` 以获得结构化、可解析的输出。

1. 任务开始时：`cs start "Fix authentication bug" --json --close-stale`
2. 添加上下文 notes：`cs note "analyzing auth flow" --json`
3. 每次 AI 调用后：`cs log-ai -p anthropic -m claude-sonnet-4 --prompt-tokens 8000 --completion-tokens 2000 --agent "Bug Fixer" --json`
4. 检查支出：`cs status --json` —— 读取 `aiCost` 字段
5. 任务结束时：`cs end -n "Fixed the auth bug, added tests" --json`
6. 审查过去的 sessions：`cs dashboard`

**提示：** 使用 `--agent` 标志标识你的 agent 工作，在 multi-agent systems 中特别有用，不同的 agents 处理不同的任务（例如 "Code Review Bot"、"Test Writer"、"Documentation Agent"）。

## Pricing

Pricing 是可配置的。运行 `cs pricing list` 查看所有已知 model 价格。覆盖或添加 models：

```bash
# 普通 model key
cs pricing set my-model 5.00 15.00

# Provider 命名空间 key（避免冲突）
cs pricing set gpt-4o 2.50 10.00 --provider openai
```

如果 pricing table 中没有该 model，记录时必须提供 `-c <cost>`。

## 预算意识

如果用户设置了预算或你检测到高支出：
- 在昂贵操作前检查 `cs status --json`
- 如果单个 session 的 `aiCost` 超过 $5.00，警告用户
- 如果成本上升，建议更便宜的 models

## 重要

- 在 `cs start` 上使用 `--close-stale` 以避免之前崩溃导致的 "session_active" 错误
- 如果 `cs` 未安装，跳过 session 跟踪——不要阻塞用户的任务
- **始终** 在每个命令上使用 `--json` —— agents 必须使用结构化输出
- 使用 `--json` 时，`cs start` 创建 session 并退出（无需长期运行进程）
- Sessions 按 **git root** 划分——从 subdirectory 运行仍然匹配 repo-level session
- 在 `--json` 模式下出错时，exit code 始终为 `1`，响应为 `{ "error": { "code": "...", "message": "..." } }`
- 检查 JSON 响应中的 `schemaVersion` 以检测 breaking changes

## 更新检查

定期检查 CLI 是否最新（例如在 session 开始时或每天一次）：

```bash
# 将已安装版本与 npm 上最新版本比较：
cs --version          # 已安装版本
npm view codesession-cli version  # npm 上最新版本
```

如果已安装版本落后，通知用户：
> "codesession-cli update available: v1.8.4 -> v1.9.0. Run `npm install -g codesession-cli` to update."

不要阻塞工作流进行更新——只需通知并继续。

## JSON 输出

所有命令都支持 `--json` 以获得 machine-readable 输出。当你需要以编程方式解析 session 数据时使用此功能。
