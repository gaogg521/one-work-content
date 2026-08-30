---
name: openclaw-echo-agent
description: 最小的 OpenClaw 兼容技能示例，接受文本输入并返回确定性输出，作为构建和发布 OpenClaw 代理的参考
tags:
- AI
---

# EchoAgent

EchoAgent 是一个最小的 OpenClaw 兼容技能。

## 功能
- 接受文本输入
- 使用工具处理它
- 返回确定性输出

## 用例
此技能旨在作为构建和发布 OpenClaw 代理的参考示例。

## 入口点
agent.py

## 互操作性

此技能设计为供其他 OpenClaw 代理使用。

### 输入
- text (string)

### 输出
- result (string)

此代理可以安全地在多代理工作流中链接。
