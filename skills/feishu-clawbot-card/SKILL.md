---
name: feishu-clawbot-card
description: 飞书上 AI Agents 的通用名片协议，支持创建、交换和存储标准化身份名片（ClawCards）
tags:
- AI
- 飞书
---

# Feishu ClawBot Card (FCC)

**飞书上 AI Agents 的通用名片协议。**

此技能允许 OpenClaw bots 创建、交换和存储标准化身份名片（"ClawCards"）。它相当于你的 AI 用来记住谁是谁的通讯录。

## 📦 安装

```bash
openclaw install HMyaoyuan/feishu-clawbot-card
```

## 🚀 使用指南

### 1. 🆔 铸造你的名片（创建身份）
首先，定义*你*是谁。运行一次以在本地注册表中注册你自己。

```bash
node skills/feishu-clawbot-card/index.js mint '{
  "display_name": "MyBotName",
  "feishu_id": "cli_a...", 
  "avatar": { "url": "https://..." },
  "bio": {
    "species": "Robot",
    "mbti": "INTJ",
    "desc": "I am a helpful coding assistant."
  },
  "capabilities": ["coding", "search"]
}'
```
*注意：`feishu_id` 应该是你的 App ID (`cli_...`) 或 User Open ID (`ou_...`)。*

### 2. 📤 分享你的名片（导出）
生成一个可分享的 JSON 代码块，发送给其他 bots 或人类。

```bash
# 获取特定 bot 的 JSON（按名称或 ID）
node skills/feishu-clawbot-card/index.js export "MyBotName"
```
**输出：** 一个 JSON 块。复制它并在聊天中发送！

### 3. 📥 保存朋友的名片（导入）
当有人发送给你他们的名片 JSON（遵循 FCC-v1 协议）时，将其保存到你的注册表。

```bash
# 粘贴接收到的 JSON 字符串
node skills/feishu-clawbot-card/index.js import '{"protocol":"fcc-v1", ...}'
```

### 4. 📇 查看注册表（列表）
查看你认识的所有 bots。

```bash
node skills/feishu-clawbot-card/index.js list
```

### 5. 🎨 展示名片（渲染）
生成精美的飞书富文本（Post）JSON，以在聊天中展示名片。

```bash
node skills/feishu-clawbot-card/index.js render "MyBotName"
```

## 📜 协议模式 (FCC v1)

有效的名片必须遵循以下 JSON 结构：

```json
{
  "protocol": "fcc-v1",
  "id": "uuid...",
  "display_name": "Name",
  "feishu_id": "cli_... or ou_...",
  "avatar": { "url": "https://..." },
  "bio": {
    "species": "...",
    "mbti": "...",
    "desc": "..."
  },
  "capabilities": ["tag1", "tag2"]
}
```
