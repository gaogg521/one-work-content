---
name: openpond-cli
description: 使用 OpenPond CLI 创建仓库、监视部署并在没有 Web UI 的情况下运行工具。
metadata: None
short-description: OpenPond CLI 工作流
tags:
- Web
---

# OpenPond CLI

当代理需要通过 CLI 创建或管理 OpenPond 应用而不使用 MCP 时，使用此技能。

## 快速设置

- 安装：`npm i -g openpond-code`（或 `npx --package openpond-code openpond <cmd>`）
- 认证：运行 `openpond login` 或设置 `OPENPOND_API_KEY`
- 非交互式登录：`openpond login --api-key opk_...`

## 常见工作流

- 创建内部仓库并附加远程：
  - `openpond repo create --name my-repo --path .`
- 非交互式推送（令牌化远程）：
  - `openpond repo create --name my-repo --path . --token`
  - `git add . && git commit -m "init"`
  - `openpond repo push --path . --branch main`
  - `openpond repo push` 读取 `.git/config`，临时令牌化 `origin`，并在推送后恢复它。
- 监视部署：
  - `openpond deploy watch handle/repo --branch main`
- 列出和运行工具：
  - `openpond tool list handle/repo`
  - `openpond tool run handle/repo myTool --body '{"foo":"bar"}'`
- 账户级 API：
  - `openpond apps list [--handle <handle>] [--refresh]`
  - `openpond apps tools`
  - `openpond apps performance --app-id app_123`
  - `openpond apps agent create --prompt "Build a daily digest agent"`

## OpenTool 透传

使用 CLI 通过 `npx` 运行 OpenTool 命令：

- `openpond opentool init --dir .`
- `openpond opentool validate --input tools`
- `openpond opentool build --input tools --output dist`

## 配置和 URL

- 可选环境变量：`OPENPOND_BASE_URL`、`OPENPOND_API_URL`、`OPENPOND_TOOL_URL`、`OPENPOND_API_KEY`
- 缓存文件：`~/.openpond/cache.json`（下次使用时自动刷新）
