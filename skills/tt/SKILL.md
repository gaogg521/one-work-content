---
name: tt
description: 通过 wacli CLI 向他人发送 WhatsApp 消息或搜索/同步 WhatsApp 历史记录（不用于普通用户聊天）。
---

# wacli

仅当用户明确要求你给 WhatsApp 上的其他人发消息，或要求同步/搜索 WhatsApp 历史记录时才使用 `wacli`。
请勿将 `wacli` 用于普通用户聊天；Clawdbot 会自动路由 WhatsApp 对话。
如果用户正在 WhatsApp 上与你聊天，除非他们要求你联系第三方，否则不应使用此工具。

安全
- 需要明确的收件人 + 消息文本。
- 发送前确认收件人 + 消息。
- 如有任何歧义，请提出澄清问题。

认证 + 同步
- `wacli auth`（二维码登录 + 初始同步）
- `wacli sync --follow`（持续同步）
- `wacli doctor`

查找聊天 + 消息
- `wacli chats list --limit 20 --query "name or number"`
- `wacli messages search "query" --limit 20 --chat <jid>`
- `wacli messages search "invoice" --after 2025-01-01 --before 2025-12-31`

历史回填
- `wacli history backfill --chat <jid> --requests 2 --count 50`

发送
- 文本：`wacli send text --to "+14155551212" --message "Hello! Are you free at 3pm?"`
- 群组：`wacli send text --to "1234567890-123456789@g.us" --message "Running 5 min late."`
- 文件：`wacli send file --to "+14155551212" --file /path/agenda.pdf --caption "Agenda"`

注意
- 存储目录：`~/.wacli`（可用 `--store` 覆盖）。
- 解析时使用 `--json` 获取机器可读输出。
- 回填需要你的手机在线；结果尽力而为。
- WhatsApp CLI 不用于常规用户聊天；它是用来给其他人发消息的。
- JID：直接聊天格式为 `<number>@s.whatsapp.net`；群组格式为 `<id>@g.us`（使用 `wacli chats list` 查找）。
