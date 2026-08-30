---
name: vapi
description: 管理 Vapi 语音代理（助手）、通话、电话号码、工具和网络钩子，支持 REST API 和 CLI 集成
tags:
- API
---

# Vapi (vapi.ai) — OpenClaw 技能

当你需要从 OpenClaw 代理管理 **Vapi 语音代理**（助手）、通话、电话号码、工具和网络钩子时，使用此技能。

此技能是 **API 优先**（Vapi REST），并可选择性与 **Vapi CLI** 集成以用于 MCP 文档 / 本地工作流。

## 你可以做什么

- 创建/更新/列出 **助手**
- 开始/检查/结束 **通话**
- 管理 **电话号码**
- 创建/管理 **工具**（函数调用）
- 配置 **网络钩子** 并检查事件

## 必需的密钥

设置以下之一：

- `VAPI_API_KEY`（推荐）— Vapi 仪表板 API 密钥。

### 如何提供密钥（推荐）

- 存储为 Gateway 密钥/环境变量（首选），或
- 在运行辅助脚本前导出到你的 shell。

切勿将密钥粘贴到公共日志中。

## 端点

基础 URL：

- `https://api.vapi.ai`

认证：

- `Authorization: Bearer $VAPI_API_KEY`

API 参考：

- https://api.vapi.ai/api (Swagger)

## 工具选项

此技能支持 **两种** 方法；你可以根据每次部署决定。

- 设置 `VAPI_MODE=api` 以优先使用 REST（默认）
- 设置 `VAPI_MODE=cli` 以优先使用 Vapi CLI（交互式）

### 选项 A — 通过辅助脚本使用 REST（随处可用）

此仓库包含一个微型 Node 辅助程序：

- `skills/vapi/bin/vapi-api.mjs`

示例：

```bash
# 列出助手
VAPI_API_KEY=... node skills/vapi/bin/vapi-api.mjs assistants:list

# 创建助手
VAPI_API_KEY=... node skills/vapi/bin/vapi-api.mjs assistants:create \
  --name "Claw Con Concierge" \
  --modelProvider openai --model gpt-4o-mini \
  --voiceProvider 11labs --voiceId rachel

# 开始外呼（示例形状；有关必填字段请参阅 swagger）
VAPI_API_KEY=... node skills/vapi/bin/vapi-api.mjs calls:create \
  --assistantId asst_xxx \
  --to "+14155551234" \
  --from "+14155559876"
```

### 选项 B — Vapi CLI（适合交互式操作）

如果 `VAPI_MODE=cli`，优先使用 CLI 进行管理任务，如果 CLI 未安装则回退到 REST。

文档：
- https://docs.vapi.ai/cli
- https://github.com/VapiAI/cli

安装：

```bash
curl -sSL https://vapi.ai/install.sh | bash
vapi login
```

### 选项 C — 用于你的 IDE 的 MCP 文档服务器

这可改进 IDE 辅助（Cursor/Windsurf/VSCode）：
- https://docs.vapi.ai/cli/mcp

```bash
vapi mcp setup
```

## 代理使用指南

当用户请求 Vapi 更改时：

1. 明确 **范围**：助手 vs 电话号码 vs 网络钩子 vs 工具调用。
2. 首先优先进行 **只读** 查询（列表/获取），然后再进行破坏性更改。
3. 创建助手时，询问：
   - 助手名称
   - 模型提供商/模型
   - 语音提供商/语音 ID
   - 工具/函数调用需求
   - 网络钩子 URL（如果使用服务器事件）
4. 发起通话时，确认：
   - 去电/来电号码
   - assistantId
   - 合规约束（录音、同意）

## 此技能中的文件

- `skills/vapi/SKILL.md` — 此文件
- `skills/vapi/bin/vapi-api.mjs` — 最小 REST 辅助程序

## 来源

- Vapi 文档介绍：https://docs.vapi.ai/quickstart/introduction
- Vapi CLI：https://github.com/VapiAI/cli
- Vapi MCP：https://docs.vapi.ai/cli/mcp
- Vapi API (Swagger)：https://api.vapi.ai/api
- 示例服务器 (Node)：https://github.com/VapiAI/example-server-javascript-node
- OpenClaw：https://github.com/openclaw/openclaw
