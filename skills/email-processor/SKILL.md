---
name: email-processor
description: 自动化Gmail收件箱处理：分类未读邮件，自动将营销/订阅/推广邮件标记为已读，并提取需要关注的重要邮件。 触发条件：用户请求\"检查邮件\"、\"处理未读邮件\"、\"清理收件箱\"、\"标记订阅邮件为已读\"或任何Gmail自动化任务。 依赖要求：gog CLI（brew install steipete/tap/gogcli）+ Google Cloud OAuth凭证
---

# Email Processor

Automates Gmail inbox triage by categorizing unread emails and marking low-priority items as 读取.

## 功能

1. Fetches all unread emails
2. Identifies marketing, newsletters, promotions, and news
3. Marks low-priority emails as 读取
4. Surfaces 重要 emails (GitHub, security alerts, direct communications)
5. Generates a 摘要 report

## 先决条件

### 1. 安装 gog CLI

```bash
brew install steipete/tap/gogcli
```

验证 安装:
```bash
gog --version
```

### 2. Google Cloud OAuth 设置

1. Go 迁移到 [Google Cloud Console](https://console.cloud.google.com/)
2. 创建 a new 项目 (or use existing)
3. 启用 the Gmail API:
   - Navigate 迁移到 "APIs & Services" > "库"
   - 搜索 "Gmail API" and 启用 it
4. 创建 OAuth credentials:
   - Go 迁移到 "APIs & Services" > "Credentials"
   - Click "创建 Credentials" > "OAuth 客户端 ID"
   - Select "Desktop app" as 应用 类型
   - 下载 the JSON 文件

### 3. Authenticate gog

```bash
# Set credentials
gog auth credentials /path/to/client_secret.json

# Add your Gmail account
gog auth add your@gmail.com --services gmail

# Verify
gog auth list
```

## 用法

### Quick 处理

```bash
bash ~/.openclaw/workspace/skills/email-processor/scripts/process-emails.sh
```

### Manual Processing (via Codex)

1. **获取 unread emails:**
   ```bash
   gog gmail 搜索 'is:unread' --json --max 100
   ```

2. **Mark specific 线程 as 读取:**
   ```bash
   gog gmail 线程 修改 <thread-id> --移除 UNREAD --force
   ```

3. **Mark marketing emails (batch):**
   ```bash
   gog gmail 搜索 'is:unread' --json --max 100 | \
     jq -r '.threads[] | select(.labels | contains(["CATEGORY_PROMOTIONS"])) | .id' | \
     while read id; do gog gmail thread modify "$id" --remove UNREAD --force; done
   ```

## Email Categories

### Auto-Marked as 读取

- `CATEGORY_PROMOTIONS` - Promotional emails
- `[Superhuman]/AI/News` - Newsletters
- `[Superhuman]/AI/Marketing` - Marketing emails
- `[Superhuman]/AI/Pitch` - Cold outreach/pitches
- `[Superhuman]/AI/AutoArchived` - Auto-categorized low priority

### Preserved (重要)

- GitHub notifications (PRs, issues, security alerts)
- Direct personal communications
- Financial/transaction emails
- Security alerts
- Calendar invites

## Verification

检查 设置 is working:

```bash
# Test gog connectivity
gog gmail search 'is:unread' --max 5

# Check account
gog auth list

# Test modify (dry run - just list what would be marked)
gog gmail search 'is:unread' --json --max 10 | \
  jq -r '.threads[] | select(.labels | contains(["CATEGORY_PROMOTIONS"])) | {id: .id, subject: .subject}'
```

## 故障排除

| 问题 | Solution |
|-------|----------|
| `gog: command not found` | 运行 `brew 安装 `brew`brew 安装 `teip`brew 安装 `gcli`
| `authentication required` | 运行 `gog auth crede`gog auth crede`tials`添加` |`gog auth`tials`_CODE_3___`gog`__CODE_``
| `token expired` | 运行 `gog `gog ``uth 刷新 your@gmail.com`
| `Gmail API not enabled` | 启用 at https://console.cloud.google.com/apis/library/gmail.googleapis.com |
| Rate 极限 errors | 添加 delays between requests or reduce batch size |

## Labels 参考

Gmail automatically applies these labels:
- `CATEGORY_PERSONAL` - Personal emails
- `CATEGORY_SOCIAL` - Social notifications
- `CATEGORY_PROMOTIONS` - Promotions
- `CATEGORY_UPDATES` - Updates/notifications
- `CATEGORY_FORUMS` - Forum messages
- `IMPORTANT` - Marked 重要
- `UNREAD` - Unread status

Superhuman AI labels (if using Superhuman):
- `[Superhuman]/AI/News` - Newsletters
- `[Superhuman]/AI/Marketing` - Marketing
- `[Superhuman]/AI/Pitch` - Pitches
- `[Superhuman]/AI/AutoArchived` - Auto-archived