---
name: xmtp-agent
description: 使用 Agent SDK 构建和扩展 XMTP 代理，支持创建配置代理、实现命令、附件、反应、群组、交易、内联操作和域名解析等功能。
---

# XMTP 代理

在 XMTP 网络上构建事件驱动的消息代理。此技能是入口点；使用下面的子技能获取特定功能。

## 何时应用

- 启动或配置新的 XMTP 代理
- 添加行为：命令、附件、反应、群组、支付、内联操作或地址/域名解析

## 子技能

| 子技能 | 使用场景 |
|-----------|----------|
| **building-agents** | 设置、环境变量、文本/生命周期事件、中间件 |
| **handling-commands** | 斜杠命令、验证器、消息过滤器、类型守卫 |
| **handling-attachments** | 发送/接收文件、远程附件、上传存储 |
| **sending-reactions** | 发送或接收反应、思考/回复模式 |
| **managing-groups** | 创建群组、添加成员、角色、安装时欢迎 |
| **handling-transactions** | USDC 转账、余额检查、交易引用 |
| **creating-inline-actions** | 内联操作、确认/选择辅助程序、配置菜单 |
| **resolving-domains** | 解析地址、Farcaster 个人资料、提取提及 |

## 如何使用

1. 选择与任务匹配的子技能（例如斜杠命令 → `handling-commands`）。
2. 阅读该子技能的 `SKILL.md` 及其 `rules/` 以获取分步指南。
3. 对于 SDK 或 API 详情，使用 xmtp-docs 技能（索引 + 特定页面获取）。

## 快速开始

安装 Agent SDK，从环境创建代理，处理文本，然后启动：

```bash
npm install @xmtp/agent-sdk
```

使用 **building-agents** 中的模式创建代理并处理消息（设置、事件、中间件）。对于命令、附件、反应、群组、交易、内联操作或解析，使用上面相应的子技能。
