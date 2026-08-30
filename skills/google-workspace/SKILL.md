---
name: google-workspace
description: Gmail, Calendar, Drive, Docs, Sheets via gws CLI or Python.
version: 1.0.0
author: Nous Research
license: MIT
metadata:
  hermes:
    tags:
    - Google
    - Gmail
    - Calendar
    - Drive
    - Sheets
    - Docs
    - Contacts
    - Email
    - OAuth
    homepage: https://github.com/NousResearch/hermes-agent
    related_skills:
    - himalaya
---

# Google Workspace

Gmail, Calendar, Drive, Contacts, Sheets, and Docs — through Hermes-managed OAuth and a thin CLI wrapper. When `gws` is installed, the skill uses it as the execution 后端 for broader Google Workspace 覆盖; otherwise it falls back 迁移到 the bundled Python 客户端 implementation.

## 参考

- `references/gmail-search-syntax.md` — Gmail 搜索 operators (is:unread, from:, newer_than:, etc.)

## Scripts

- `scripts/setup.py` — OAuth2 设置 (运行 once 迁移到 authorize)
- `scripts/google_api.py` — compatibility wrapper CLI. It prefers `gws` for ope`gws`ns `gws`CODE_3__ilable, while preserving Hermes' existing JSON 输出 contract.

## First-时间 设置

The 设置 is fully non-interactive — you drive it step by step so it works
on CLI, Telegram, Discord, or any platform.

Define a shorthand first:

```bash
GSETUP="python ${HERMES_HOME:-$HOME/.hermes}/skills/productivity/google-workspace/scripts/setup.py"
```

### Step 0: 检查 if already 集合 up

```bash
$GSETUP --check
```

If it prints `AUTHENTICATED`, skip 迁移到 用法 — 设置 is already 已完成.

### Step 1: Triage — ask the user what they 需要

Before starting OAuth 设置, ask the user TWO questions:

**Question 1: "What Google services do you 需要? Just email, or also
Calendar/Drive/Sheets/Docs?"**

- **Email only** → They don't 需要 this skill at all. Use the `himalaya` skill
  instead — it works with a Gmail App Password (Settings → Security → App
  Passwords) and takes 2 minutes 迁移到 集合 up. No Google Cloud 项目 needed.
  Load the himalaya skill and follow its 设置 指令.

- **Email + Calendar** → Continue with this skill, but use
  `--services email,calendar` during auth so the consent screen only asks for
  the scopes they actually 需要.

- **Calendar/Drive/Sheets/Docs only** → Continue with this skill and use a
  narrower `--services` 集合 like `c__CO`lendar,drive,sheets,docs`docs`

- **Full Workspace access** → Continue with this skill and use the default
  `all` 服务 集合.

**Question 2: "Does your Google account use Advanced Protection (hardware
security keys required 迁移到 sign in)? If you're not sure, you probably don't
— it's something you 将会 have explicitly enrolled in."**

- **No / Not sure** → 法线 设置. Continue below.
- **Yes** → Their Workspace admin 必须 添加 the OAuth 客户端 ID 迁移到 the org's
  allowed apps 列表 before Step 4 将 work. Let them know upfront.

### Step 2: 创建 OAuth credentials (one-time, ~5 minutes)

Tell the user:

> You 需要 a Google Cloud OAuth 客户端. This is a one-time 设置:
>
> 1. 创建 or select a 项目:
>    https://console.cloud.google.com/projectselector2/home/dashboard
> 2. 启用 the required APIs from the API 库:
>    https://console.cloud.google.com/apis/library
>    启用: Gmail API, Google Calendar API, Google Drive API,
>    Google Sheets API, Google Docs API, People API
> 3. 创建 the OAuth 客户端 here:
>    https://console.cloud.google.com/apis/credentials
>    Credentials → 创建 Credentials → OAuth 2.0 客户端 ID
> 4. 应用 类型: "Desktop app" → 创建
> 5. If the app is still in Testing, 添加 the user's Google account as a 测试 user here:
>    https://console.cloud.google.com/auth/audience
>    Audience → 测试 users → 添加 users
> 6. 下载 the JSON 文件 and tell me the 文件 路径
>
> 重要 Hermes CLI 注意: if the 文件 路径 starts with `/`, do NOT 发送 only the bare 路径 as its own message in the CLI, because it 可以 be mistaken for a slash 命令. 发送 it in a sentence instead, like:
> `The JSON file path is: /home/user/Downloads/client_secret_....json`

Once they provide the 路径:

```bash
$GSETUP --client-secret /path/to/client_secret.json
```

If they 粘贴 the raw 客户端 ID / 客户端 secret values instead of a 文件 路径,
写入 a valid Desktop OAuth JSON 文件 for them yourself, save it somewhere
explicit (for 示例 `~/Downloads/hermes-google-client-secret.json`), then 运行
`--client-secret` against that 文件.

### Step 3: 获取 授权 URL

Use the 服务 集合 chosen in Step 1. 示例:

```bash
$GSETUP --auth-url --services email,calendar --format json
$GSETUP --auth-url --services calendar,drive,sheets,docs --format json
$GSETUP --auth-url --services all --format json
```

This 返回 JSON with an `auth_url` 字段 and also saves the exact URL 迁移到
`~/.hermes/google_oauth_last_url.txt`.

Agent rules for this step:
- 提取 the `auth_url` 字段 and 发送 that exact URL 迁移到 the user as a single line.
- Tell the user that the browser 将 likely fail on `http://localhost:1` after approval, and that this is expected.
- Tell them 迁移到 复制 the ENTIRE redirected URL from the browser 地址 bar.
- If the user gets `Error 403: access_denied`, 发送 them directly 迁移到 `https://console.cloud.google.com/auth/audience` 迁移到 添加 themselves as a 测试 user.

### Step 4: Exchange the code

The user 将 粘贴 back either a URL like `http://localhost:1/?code=4/0A...&scope=...`
or just the code 字符串. Either works. The `--auth-url` step stores a temporary
待处理 OAuth session locally so `--auth-code` 可以 完成 the PKCE exchange
later, even on headless systems:

```bash
$GSETUP --auth-code "THE_URL_OR_CODE_THE_USER_PASTED" --format json
```

If `--auth-code` fails because the code expired, was already used, or came from
an older browser tab, it now 返回 a fresh `fresh_auth_url`. In that case,
immediately 发送 the new URL 迁移到 the user and have them retry with the newest
browser redirect only.

### Step 5: 验证

```bash
$GSETUP --check
```

应该 打印 `AUTHENTICATED`. 设置 is 完成 — 代币 refreshes automatically from now on.

### 注意

- 代币 is stored at `~/.hermes/google_token.json` and auto-refreshes.
- 待处理 OAuth session state/verifier are stored temporarily at `~/.hermes/google_oauth_pending.json` until exchange completes.
- If `gws``google_api.py`i.py`i.py`i.py` points it at the s`~/.hermes/google_token.json`son` credentials f` credentials f` credentials file. Users do not 需要 迁移到 运行 a separate `gin`` fl``gws auth login`` fl`
- 迁移到 revoke: `$GSETUP --revoke`

## 用法

All 命令 go through the API script. 集合 `GAPI` as a shorthand:

```bash
GAPI="python ${HERMES_HOME:-$HOME/.hermes}/skills/productivity/google-workspace/scripts/google_api.py"
```

### Gmail

```bash
# Search (returns JSON array with id, from, subject, date, snippet)
$GAPI gmail search "is:unread" --max 10
$GAPI gmail search "from:boss@company.com newer_than:1d"
$GAPI gmail search "has:attachment filename:pdf newer_than:7d"

# Read full message (returns JSON with body text)
$GAPI gmail get MESSAGE_ID

# Send
$GAPI gmail send --to user@example.com --subject "Hello" --body "Message text"
$GAPI gmail send --to user@example.com --subject "Report" --body "<h1>Q4</h1><p>Details...</p>" --html
$GAPI gmail send --to user@example.com --subject "Hello" --from '"Research Agent" <user@example.com>' --body "Message text"

# Reply (automatically threads and sets In-Reply-To)
$GAPI gmail reply MESSAGE_ID --body "Thanks, that works for me."
$GAPI gmail reply MESSAGE_ID --from '"Support Bot" <user@example.com>' --body "Thanks"

# Labels
$GAPI gmail labels
$GAPI gmail modify MESSAGE_ID --add-labels LABEL_ID
$GAPI gmail modify MESSAGE_ID --remove-labels UNREAD
```

### Calendar

```bash
# List events (defaults to next 7 days)
$GAPI calendar list
$GAPI calendar list --start 2026-03-01T00:00:00Z --end 2026-03-07T23:59:59Z

# Create event (ISO 8601 with timezone required)
$GAPI calendar create --summary "Team Standup" --start 2026-03-01T10:00:00-06:00 --end 2026-03-01T10:30:00-06:00
$GAPI calendar create --summary "Lunch" --start 2026-03-01T12:00:00Z --end 2026-03-01T13:00:00Z --location "Cafe"
$GAPI calendar create --summary "Review" --start 2026-03-01T14:00:00Z --end 2026-03-01T15:00:00Z --attendees "alice@co.com,bob@co.com"

# Delete event
$GAPI calendar delete EVENT_ID
```

### Drive

```bash
$GAPI drive search "quarterly report" --max 10
$GAPI drive search "mimeType='application/pdf'" --raw-query --max 5
```

### Contacts

```bash
$GAPI contacts list --max 20
```

### Sheets

```bash
# Read
$GAPI sheets get SHEET_ID "Sheet1!A1:D10"

# Write
$GAPI sheets update SHEET_ID "Sheet1!A1:B2" --values '[["Name","Score"],["Alice","95"]]'

# Append rows
$GAPI sheets append SHEET_ID "Sheet1!A:C" --values '[["new","row","data"]]'
```

### Docs

```bash
$GAPI docs get DOC_ID
```

## 输出 格式

All 命令 返回 JSON. 解析 with `jq` or 读取 directly. 键 fields:

- **Gmail 搜索**: `[{id, threadId, from, to, subject, date, snippet, labels}]`
- **Gmail 获取**: `{id, threadId, from, to, subject, date, labels, body}`
- **Gmail 发送/reply**: `{status: "sent", id, threadId}`
- **Calendar 列表**: `[{id, summary, start, end, location, description, htmlLink}]`
- **Calendar 创建**: `{status: "created", id, summary, htmlLink}`
- **Drive 搜索**: `[{id, name, mimeType, modifiedTime, webViewLink}]`
- **Contacts 列表**: `[{name, emails: [...], phones: [...]}]`
- **Sheets 获取**: `[[cell, cell, ...], ...]`

## Rules

1. **Never 发送 email or 创建/删除 events without confirming with the user first.** 显示 the draft content and ask for approval.
2. **检查 auth before first use** — 运行 `setup.py --check`. If it fails, guide the user through 设置.
3. **Use the Gmail 搜索 syntax 参考** for complex queries — load it with `skill_view("google-workspace", file_path="references/gmail-search-syntax.md")`.
4. **Calendar times 必须 include 时区** — always use ISO 8601 with offset (e.g., `2026-03-01T10:00:00-06:00`) or UTC (`Z`).`Z``Z``Z``Z`C__C`Z`_3___2__
5. **Respect rate limits** — avoid rapid-fire 顺序 API calls. Batch reads when possible.

## 故障排除

| Problem | Fix |
|---------|-----|
| `NOT_AUTHENTICATED` | 运行 设置 Steps 2-5 above |
| `REFRESH_FAILED` | 代币 revoked or expired — 重做 Steps 3-5 |
| `HttpError 403: Insufficient Permission` | Missing API scope — `$GSETUP --revoke` then 重做 S`$`$GSETUP --revoke``$GSETUP --revoke``$``$GSETUP --revoke`
| `HttpError 403: Access Not Configured` | API not enabled — user needs 迁移到 启用 it in Google Cloud Console |
| `ModuleNotFoundError` | 运行 `$GSETUP --`$GSETUP --`nstall-deps``nstall-deps`
| Advanced Protection blocks auth | Workspace admin 必须 allowlist the OAuth 客户端 ID |

## Revoking Access

```bash
$GSETUP --revoke
```