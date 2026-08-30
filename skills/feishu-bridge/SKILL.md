---
name: feishu-bridge
description: 通过 WebSocket 长连接将飞书 (Lark) 机器人连接到 Clawdbot。无需公网服务器、域名或 ngrok。在设置飞书/Lark 作为消息渠道、排查飞书桥接问题或管理桥接服务（启动/停止/日志）时使用。涵盖在飞书开放平台创建机器人、凭证设置、桥接启动、macOS launchd 自动重启以及群聊行为调优。
tags:
- 飞书
---

# Feishu Bridge

Bridge Feishu bot messages to Clawdbot Gateway over local WebSocket.

## Architecture

```
Feishu user → Feishu cloud ←WS→ bridge.mjs (local) ←WS→ Clawdbot Gateway → AI agent
```

- Feishu SDK connects outbound (no inbound port / public IP needed)
- Bridge authenticates to Gateway using the existing gateway token
- Each Feishu chat maps to a Clawdbot session (`feishu:<chatId>`)

## Setup

### 1. Create Feishu bot

1. Go to [open.feishu.cn/app](https://open.feishu.cn/app) → Create self-built app → Add **Bot** capability
2. Enable permissions: `im:message`, `im:message.group_at_msg`, `im:message.p2p_msg`
3. Events: add `im.message.receive_v1`, set delivery to **WebSocket long-connection**
4. Publish the app (create version → request approval)
5. Note the **App ID** and **App Secret**

### 2. Store secret

```bash
mkdir -p ~/.clawdbot/secrets
echo "YOUR_APP_SECRET" > ~/.clawdbot/secrets/feishu_app_secret
chmod 600 ~/.clawdbot/secrets/feishu_app_secret
```

### 3. Install & run

```bash
cd <skill-dir>/feishu-bridge
npm install
FEISHU_APP_ID=cli_xxx node bridge.mjs
```

### 4. Auto-start (macOS)

```bash
FEISHU_APP_ID=cli_xxx node setup-service.mjs
launchctl load ~/Library/LaunchAgents/com.clawdbot.feishu-bridge.plist
```

## Diagnostics

```bash
# Check service
launchctl list | grep feishu

# Logs
tail -f ~/.clawdbot/logs/feishu-bridge.err.log

# Stop
launchctl unload ~/Library/LaunchAgents/com.clawdbot.feishu-bridge.plist
```

## Group chat behavior

Bridge replies only when: user @-mentions the bot, message ends with `?`/`？`, contains request verbs (帮/请/分析/总结…), or calls the bot by name. Customize the name list in `bridge.mjs` → `shouldRespondInGroup()`.

## Environment variables

| Variable | Required | Default |
|---|---|---|
| `FEISHU_APP_ID` | ✅ | — |
| `FEISHU_APP_SECRET_PATH` | — | `~/.clawdbot/secrets/feishu_app_secret` |
| `CLAWDBOT_CONFIG_PATH` | — | `~/.clawdbot/clawdbot.json` |
| `CLAWDBOT_AGENT_ID` | — | `main` |
| `FEISHU_THINKING_THRESHOLD_MS` | — | `2500` |
