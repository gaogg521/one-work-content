---
name: cloudphone
description: 通过 mcporter 调用 cpc-mcp-server AutoJS Agent 工具，执行云 Android 自动化任务并获取结果。触发词：云手机(cloud phone)、AutoJS、mcporter、Android 自动化(Android automation)、cpc-mcp-server。
metadata: None
openclaw: None
emoji: 📱
install:
- bins:
  - mcporter
  id: node
  kind: node
  label: 安装 mcporter (node)
  package: mcporter
primaryEnv: CLOUDPHONE_API_KEY
requires: None
bins:
- mcporter
env:
- CLOUDPHONE_API_KEY
tags:
- AI
- 云服务
- 自动化
---

## 此技能的功能

`cloudphone` 引导 agent 通过 `mcporter` 调用 `cpc-mcp-server` 工具，在云手机环境中运行 Android 自动化任务。

适用于：
- 基于 AutoJS 的云手机自动化
- App 回归/冒烟测试执行
- 远程批量操作工作流
- 云 Android 设备上的脚本交互

---

## 何时使用此技能

当用户请求以下操作时，使用此技能：
- “在云手机上运行脚本”
- “使用 AutoJS 自动化此 App 流程”
- “远程执行 Android UI 步骤并返回截图/日志”
- “使用 cpc-mcp-server 进行云设备自动化”

---

## 何时不使用此技能

以下情况请勿使用此技能：
- 本地 ADB/模拟器自动化（非云设备）
- iOS 自动化（例如 XCUITest）
- 不涉及真实设备执行的静态脚本/代码审查
- 没有可执行任务目标的纯咨询请求

---

## 必需的前提条件（执行前必须通过）

在发起任何调用之前，请确保认证配置正确。

`cpc-mcp-server` 需要：

- `Authorization: Bearer <API_KEY>`

此技能通过以下方式标准化 API key：

- `CLOUDPHONE_API_KEY`（必需）

### 要求

1. 设置环境变量 `CLOUDPHONE_API_KEY`。
2. 确保在执行前 MCP 服务器 header 注入已激活：
   - `Authorization: Bearer $CLOUDPHONE_API_KEY`
3. 切勿在仓库文件、`SKILL.md` 或 config JSON 中硬编码或提交真实 key。

> 此技能仅定义命名和前置条件。  
> Secret 注入的实现由运行时/环境配置处理。

---

## 调用模型（通过 mcporter）

此技能不直接调用 MCP 工具。它使用 `mcporter` CLI 在 `cpc-mcp-server` 上调用工具。

常用命令模式：
- 列出已配置的 MCP 服务器：
  - `mcporter list`
- 查看服务器 schema：
  - `mcporter list cpc-mcp-server --schema`
- 使用 JSON 参数调用工具（推荐）：
  - `mcporter call cpc-mcp-server.<tool> --args '<json>'`

> 对于长指令、多语言文本和特殊字符，优先使用 `--args` JSON。

---

## 最小输入检查清单（创建任务前）

首先收集以下字段（仅在缺失时追问）：
1. 目标 app（名称/包名）
2. 预期操作（要做什么）
3. 成功标准（怎样算完成）
4. 预期输出类型（截图/日志/文本结果）

---

## 工具 1：创建任务 (`createAutoJsAgentTask`)

### 目标

创建并派发 AutoJS Agent 任务，然后获取 `taskId`（以及可能的 `sessionId`）。

### 推荐调用

```bash
mcporter call cpc-mcp-server.createAutoJsAgentTask --args '{
  "instruction": "在云手机上打开 <APP_NAME>，使用测试账号登录，导航到 Orders 页面，截取屏幕截图，并返回订单数量。",
  "lang": "zh"
}'
```
