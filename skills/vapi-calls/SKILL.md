---
name: vapi-calls
description: 用于电话呼叫的高级 AI 语音助手，支持说服、销售、餐厅预订、提醒与通知。
emoji: 📞
author: César Morillas
version: 1.0.0
license: MIT
repository: https://github.com/vapi-ai/openclaw-vapi-calls
requires: None
bins:
- python3
env:
- VAPI_API_KEY
- VAPI_ASSISTANT_ID
- VAPI_PHONE_NUMBER_ID
- WEBHOOK_BASE_URL
pip:
- requests
tools:
- name: make_vapi_call
  description: 触发一个具有特定任务的自主 AI 电话呼叫，并等待最终报告。
  parameters:
    type: object
    properties:
      phone_number:
        type: string
        description: 接收方的电话号码（E.164 格式，例如 +34669000000）。
      first_message:
        type: string
        description: 初始问候语。使用 'DEFAULT' 以使用 agent 配置的问候语。
      system_prompt:
        type: string
        description: AI 指令。使用 'DEFAULT' 以使用 agent 配置的模型/提示词。如果提供了自定义文本，呼叫将强制使用 GPT-4o
          Mini + endCall 工具。
      end_message:
        type: string
        description: 可选。结束语。使用 'DEFAULT' 以跳过覆盖。
    required:
    - phone_number
    - first_message
    - system_prompt
tags:
- AI
---

# Vapi Calls - Agent 指令

使用此技能来执行任何需要通过电话进行语音交互的任务。

## 配置与网络要求

⚠️ **重要：** 此技能要求您的机器能够从互联网访问，以接收实时呼叫更新。

### 1. 环境变量
在您的 OpenClaw `config.json`（或 Gateway 环境）中配置以下内容：

- `VAPI_API_KEY`: 您的 Vapi 私有 API 密钥。
- `VAPI_ASSISTANT_ID`: 用作基础的 Vapi Assistant 的 ID。
- `VAPI_PHONE_NUMBER_ID`: Vapi 电话号码的 ID。
- `WEBHOOK_BASE_URL`: **关键。** 此 agent 可从公共网络访问的 HTTPS URL（例如 `https://my-claw.com` 或 `https://xyz.ngrok-free.app`）。**不要包含尾部斜杠。**
- `WEBHOOK_PORT` (可选): 要监听的本地端口（默认：`4430`）。
- `VAPI_LLM_PROVIDER`: (可选) 自定义模式使用的提供商（默认：`openai`）。
- `VAPI_LLM_MODEL`: (可选) 自定义模式使用的模型（默认：`gpt-4o-mini`）。

### 2. 连接设置
您必须将 `WEBHOOK_PORT`（默认 4430）暴露到互联网。

**选项 A: Cloudflare Tunnel（推荐）**
`cloudflared tunnel --url http://localhost:4430`

**选项 B: Ngrok**
`ngrok http 4430`

将 `WEBHOOK_BASE_URL` 设置为生成的 URL（例如 `https://random-name.trycloudflare.com`）。

## 用法

### 自定义任务（动态）
提供一个特定的 `system_prompt`。系统将自动使用 **GPT-4o Mini** 并启用 **endCall** 工具。AI 将能够自主挂断电话。

### 原生 Agent（静态）
为 `first_message`、`system_prompt` 和 `end_message` 传入 `"DEFAULT"`。系统将使用在 Vapi Dashboard 中定义的精确配置（模型、语音、提示词）。

## 故障排除

- **呼叫挂起 / 无报告：** 检查 `WEBHOOK_BASE_URL` 是否可从互联网访问。Python 脚本仅在呼叫期间在 `WEBHOOK_PORT` 上启动临时服务器。
- **API 400 错误：** 检查您的 `VAPI_PHONE_NUMBER_ID` 和 `VAPI_ASSISTANT_ID`。
