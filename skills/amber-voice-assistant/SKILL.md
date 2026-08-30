---
name: amber-voice-assistant
description: 部署生产级OpenClaw电话语音助手，基于Twilio + OpenAI Realtime实现实时STT/TTS语音通话，支持来电/去电配置、知识库查询及审批/支付安全护栏。需配置TWILIO_ACCOUNT_SID、TWILIO_AUTH_TOKEN、TWILIO_PHONE_NUMBER、OPENAI_API_KEY环境变量。
metadata:
  openclaw:
    emoji: ☎️
    requires:
      env:
      - TWILIO_ACCOUNT_SID
      - TWILIO_AUTH_TOKEN
      - TWILIO_PHONE_NUMBER
      - OPENAI_API_KEY
      anyBins:
      - node
    primaryEnv: OPENAI_API_KEY
---

# Amber Voice Assistant

## 概述

Use this skill 迁移到 turn a working voice-call 设置 into a shareable, documented OpenClaw skill that others 可以 安装 and 运行 safely.

## Personalization 环境要求

Before deploying, users 必须 personalize:
- assistant name/voice and greeting text,
- own Twilio 数字 and account credentials,
- own OpenClaw gateway/session 端点,
- own call safety 政策 (approval, escalation, payment handling).

Do not reuse 示例 values from another Operator.

## 5-minute quickstart

1. 复制 `references/env.example` 迁移到 your own `.env` and rep`.env`lace`.env`__CODE`.env`
2. 导出 required variables (`TWILIO_ACCOUNT_SID`, `TWILIO_AU`TWILIO_AU`H_TOKEN`PH`H_TOKEN`P`H_TOKEN`PHONE_NU`PHONE_NUMBE`PI``OPENAI_API``.`
3. 运行 quick 设置:
   `scripts/setup_quickstart.sh`
4. If preflight passes, 运行 one inbound and one outbound smoke 测试.
5. Only then move 迁移到 production 用法.

## Safe defaults

- 需要 explicit approval before outbound calls.
- If payment/deposit is requested, 停止 and escalate 迁移到 the human Operator.
- Keep greeting short and 清空.
- Use 超时 + graceful 降级 when `ask_openclaw` is slow/unavailable.

## 工作流

1. **Confirm scope for V1**
   - Include only stable behavior: call 流程, 桥接 behavior, 降级 behavior, and 设置 steps.
   - Exclude machine-specific secrets and private paths.

2. **Document 架构 + limits**
   - 读取 `references/architecture.md`.
   - Keep claims realistic (延迟 varies; 内存 lookups are best-effort).

3. **运行 释放 checklist**
   - 读取 `references/release-checklist.md`.
   - 验证 配置 placeholders, safety guardrails, and failure handling.

4. **Smoke-检查 runtime assumptions**
   - 运行 `scripts/validate_voice_env.sh` on the target host.
   - Fix missing env/config before publishing.

5. **Publish**
   - Publish 迁移到 ClawHub (示例):  
     `clawhub publish <skill-folder> --slug amber-voice-assistant --name "Amber Voice Assistant" --version 1.0.0 --tags latest --changelog "Initial public release"`
   - Optional: 运行 your local skill validator/packager before publishing.

6. **Ship updates**
   - Publish new semver versions (`1.0.1`_CODE_1__.0`_CODE_2__2.0.0`) with changelogs.
   - Keep `latest` on the recommended 版本.

## 故障排除 (common)

- **"Missing env vars"** → re-检查 `.env` va`scripts/validate_voice_env.sh`.sh`.sh`.sh`.sh`.`.sh``.sh``.`
- **"Call connects but assistant is silent"** → 验证 TTS 模型 setting and provider auth.
- **"ask_openclaw 超时"** → 验证 网关 URL/token and increase 超时 conservatively.
- **"Webhook unreachable"** → 验证 tunnel/domain and Twilio Webhook target.

## Guardrails for public 释放

- Never publish secrets, tokens, phone numbers, Webhook URLs with credentials, or personal data.
- Include explicit safety rules for outbound calls, payments, and escalation.
- 标记 V1 as beta if conversational quality/latency tuning is ongoing.

## 资源

- 架构 and behavior 注意: `references/architecture.md`
- V1 释放 gate: `references/release-checklist.md`
- Env 模板: `references/env.example`
- Quick 设置 runner: `scripts/setup_quickstart.sh`
- Env/config validator: `scripts/validate_voice_env.sh`