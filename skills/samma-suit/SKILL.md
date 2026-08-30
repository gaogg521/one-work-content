---
name: samma-suit
description: 为 OpenClaw agent 提供 8 层安全治理：预算控制(budget control)、权限管理(permission management)、审计日志(audit log)、终止开关(kill switch)、身份签名(identity signature)、技能审查(skill review)、进程隔离(process isolation)与网关保护(gateway protection)。触发词：安全治理(security governance)、预算控制、权限管理、审计日志、终止开关(kill switch)、agent 安全
metadata: None
openclaw: None
emoji: 🛡️
primaryEnv: SAMMA_API_KEY
requires: None
bins: None
env:
- SAMMA_API_KEY
user-invocable: True
command-dispatch: prompt
tags:
- 安全
- AI
---

# Sammā Suit — OpenClaw 的安全治理

你正在帮助用户安装和配置 Sammā Suit，这是一个开源安全框架，它作为生命周期钩子为 OpenClaw 添加 8 层强制治理。

## 功能说明

Sammā Suit 拦截 OpenClaw 的插件钩子以强制执行：
- **NIRVANA** — 终止开关。如果 agent 被终止，则阻止所有活动。
- **DHARMA** — 权限。根据允许的权限集检查工具。
- **SANGHA** — 技能审查。通过白名单 + AST 扫描阻止未经批准的技能。
- **KARMA** — 预算控制。每个 agent 的月度支出上限，带有硬性上限。
- **BODHI** — 隔离。为每个 agent 注入超时、token 和资源限制。
- **METTA** — 身份。对出站消息进行 Ed25519 加密签名。
- **SILA** — 审计追踪。记录每个工具调用、消息和会话事件。

## 安装

运行：
