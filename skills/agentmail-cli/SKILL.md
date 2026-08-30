---
name: agentmail-cli
description: 通过 AgentMail API 管理电子邮件收件箱和消息。创建一次性收件箱、发送/接收电子邮件以及列出消息。当 agent 需要发送或接收电子邮件、创建临时收件箱或检查传入消息时使用。
metadata: None
openclaw: None
emoji: 📧
install:
- bins:
  - agentmail
  id: npm
  kind: node
  label: 通过 npm 安装 agentmail-cli
  package: '@stepandel/agentmail-cli'
primaryEnv: AGENTMAIL_API_KEY
requires: None
bins:
- agentmail
env:
- AGENTMAIL_API_KEY
homepage: https://github.com/stepandel/agentmail-cli
tags:
- AI
- API
---

[AgentMail](https://agentmail.to) 的 CLI — 创建收件箱、发送消息和阅读电子邮件。

## API 密钥设置

API 密钥必须在任何命令工作之前配置。两种方法：

1. **配置文件（对持久 agent 推荐）：**
```
agentmail config set-key YOUR_API_KEY
```
这将密钥存储在 `~/.agentmail/config.json` 中，并在 session 之间持久化。

2. **环境变量：**
```
export AGENTMAIL_API_KEY=YOUR_API_KEY
```

验证配置：
```
agentmail config show
```

如果命令因认证错误而失败，请重新运行 `agentmail config set-key` — 仅环境变量可能无法在 shell session 之间持久化。

## 始终使用 --json

始终向每个命令传递 `--json` 以获得机器可读的输出。需要时使用 `jq` 进行解析。

## 收件箱命令

创建收件箱：
```
agentmail inbox create --json
agentmail inbox create --domain example.com --json
agentmail inbox create --username support --domain example.com --display-name "Support Team" --json
```

列出收件箱：
```
agentmail inbox list --json
agentmail inbox list --limit 10 --json
```

获取收件箱详情：
```
agentmail inbox get <inbox-id> --json
```

删除收件箱：
```
agentmail inbox delete <inbox-id>
```

## 消息命令

发送消息：
```
agentmail message send --from <inbox-id> --to recipient@example.com --subject "Subject" --text "Body text" --json
```

发送 HTML：
```
agentmail message send --from <inbox-id> --to recipient@example.com --subject "Subject" --html "<h1>Hello</h1>" --json
```

多个收件人、抄送、密送：
```
agentmail message send --from <inbox-id> --to "a@example.com,b@example.com" --cc "cc@example.com" --bcc "bcc@example.com" --subject "Subject" --text "Body" --json
```

列出收件箱中的消息：
```
agentmail message list <inbox-id> --json
agentmail message list <inbox-id> --limit 20 --json
```

获取特定消息：
```
agentmail message get <inbox-id> <message-id> --json
```

删除消息（删除整个线程）：
```
agentmail message delete <inbox-id> <message-id>
```

## 常见工作流程

```bash
# 1. 创建收件箱，捕获 ID
INBOX_ID=$(agentmail inbox create --json | jq -r '.inboxId')

# 2. 发送电子邮件
agentmail message send --from "$INBOX_ID" --to user@example.com --subject "Hello" --text "Message body" --json

# 3. 检查回复
agentmail message list "$INBOX_ID" --json
```

## 注意

- 在 https://agentmail.to 获取 API 密钥
- 配置文件位置：`~/.agentmail/config.json`
- 环境变量 `AGENTMAIL_API_KEY` 优先于配置文件
- 删除消息会删除包含它的整个线程