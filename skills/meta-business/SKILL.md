---
name: meta-business
description: 用于 WhatsApp、Instagram、Facebook Pages 和 Messenger 自动化的 Meta Business CLI。
metadata: None
openclaw: None
emoji: 📱
install:
- bins:
  - meta
  command: bun install -g meta-business-cli
  id: bun
  kind: command
  label: Install meta CLI (bun)
requires: None
bins:
- meta
tags:
- 元数据
---

# Meta Business CLI

使用 `meta` 通过 Graph API 实现 WhatsApp、Instagram、Facebook Pages 和 Messenger 自动化。

Setup（一次）

- `meta config set app.id YOUR_APP_ID`
- `meta config set app.secret YOUR_APP_SECRET`
- `meta auth login`（OAuth PKCE flow，打开浏览器）
- `meta doctor`（验证连接性和权限）
- 或使用 `--token YOUR_TOKEN` 配合任何命令跳过 OAuth

Surface config

- WhatsApp: `meta config set whatsapp.phoneNumberId ID` 和 `meta config set whatsapp.businessAccountId ID`
- Instagram: `meta config set instagram.accountId ID`
- Pages/Messenger: `meta config set pages.pageId ID`
- 显示全部：`meta config list`

WhatsApp 命令

- 发送文本：`meta wa send "+1234567890" --text "Hello" --json`
- 发送图片：`meta wa send "+1234567890" --image "https://example.com/photo.jpg" --caption "Look" --json`
- 发送模板：`meta wa send "+1234567890" --template "hello_world" --template-lang en_US --json`
- 发送文档：`meta wa send "+1234567890" --document "https://example.com/file.pdf" --json`
- 列出模板：`meta wa template list --json`
- 获取模板：`meta wa template get TEMPLATE_NAME --json`
- 删除模板：`meta wa template delete TEMPLATE_NAME --json`
- 上传媒体：`meta wa media upload ./photo.jpg --json`
- 获取媒体 URL：`meta wa media url MEDIA_ID --json`
- 下载媒体：`meta wa media download MEDIA_ID ./output.jpg`
- 查看分析：`meta wa analytics --days 30 --granularity DAY --json`

Instagram 命令

- 发布图片：`meta ig publish --image "https://example.com/photo.jpg" --caption "My post" --json`
- 发布视频：`meta ig publish --video "https://example.com/video.mp4" --caption "Watch this" --json`
- 发布 Reel：`meta ig publish --video "https://example.com/reel.mp4" --reel --caption "New reel" --json`
- 查看账户洞察：`meta ig insights --period day --days 30 --json`
- 查看媒体洞察：`meta ig insights --media-id MEDIA_ID --json`
- 列出评论：`meta ig comments list MEDIA_ID --json`
- 回复评论：`meta ig comments reply COMMENT_ID "Thanks!" --json`
- 隐藏评论：`meta ig comments hide COMMENT_ID --json`
- 删除评论：`meta ig comments delete COMMENT_ID --json`

Facebook Pages 命令

- 创建帖子：`meta fb post --message "Hello from the CLI" --json`
- 创建链接帖子：`meta fb post --message "Check this out" --link "https://example.com" --json`
- 列出帖子：`meta fb list --limit 10 --json`
- 查看洞察：`meta fb insights --period day --days 30 --json`

Messenger 命令

- 发送文本：`meta messenger send PSID --text "Hello" --json`
- 发送图片：`meta messenger send PSID --image "https://example.com/photo.jpg" --json`
- 使用 tag 发送：`meta messenger send PSID --text "Update" --type MESSAGE_TAG --tag HUMAN_AGENT --json`
- 列出对话：`meta messenger receive --json`
- 查看对话：`meta messenger receive --conversation-id CONV_ID --json`

Webhook 命令

- 启动监听器：`meta webhook listen --port 3000 --verify-token TOKEN --app-secret SECRET`
- 测试验证：`meta webhook verify --verify-token TOKEN --json`
- 订阅事件：`meta webhook subscribe --object whatsapp_business_account --fields messages --callback-url "https://example.com/webhook" --json`

Diagnostics

- `meta doctor --json` 检查 config、credentials、token validity、Graph API 连接性、permissions 和 surface-specific asset access。

Notes

- 自动化时始终使用 `--json` 以获得结构化输出。
- 当必需参数作为 flags 传递时，所有命令都以非交互方式工作。
- 使用 `--token TOKEN` 覆盖任何命令的存储凭证。
- 使用 `--api-version v22.0` 固定特定的 Graph API 版本。
- WhatsApp 需要配置 phone number ID 和 business account ID。
- Instagram 发布需要图片/视频的公共 URL（不是本地文件）。
- Messenger 在 24 小时窗口外发消息需要 message tag。
- 首次使用前运行 `meta doctor` 验证设置。
