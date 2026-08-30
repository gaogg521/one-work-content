---
name: zoho
description: 与Zoho CRM、Projects和Meeting API交互。用于管理deals、contacts、leads、tasks、projects、milestones、meeting recordings或任何Zoho工作空间数据。在提及Zoho、CRM、deals、pipeline、projects、tasks、milestones、meetings、recordings、standups时触发。
author: Zone 99 team
homepage: https://99.zone
repository: https://github.com/shreefentsar/clawdbot-zoho
tags:
- API
- CI/CD
---

# Zoho 集成 (CRM + Projects + Meeting)

由 [Zone 99](https://99.zone) 制作 · [GitHub](https://github.com/shreefentsar/clawdbot-zoho) · [贡献](https://github.com/shreefentsar/clawdbot-zoho/issues)

## 快速开始

使用 `zoho` CLI 包装器 —— 它会自动处理 OAuth token 刷新和缓存。

```bash
zoho help          # 显示所有命令
zoho token         # 打印当前访问令牌（自动刷新）
```

## 认证设置

### 步骤 1：注册您的应用程序

1. 前往 [Zoho API Console](https://api-console.zoho.com/)
2. 点击 **Add Client** → 选择 **Server-based Applications**
3. 填写：
   - **Client Name**: 您的应用名称（例如 "Clawdbot Zoho Integration"）
   - **Homepage URL**: 您的域名或 `https://localhost`
   - **Redirect URI**: `https://localhost/callback`（或您控制的任何 URL —— 您只需要它一次来获取 code）
4. 点击 **Create**
5. 记下 **Client ID** 和 **Client Secret**

### 步骤 2：生成授权码（Grant Token）

构建此 URL 并在浏览器中打开（替换占位符）：

```
https://accounts.zoho.com/oauth/v2/auth
  ?response_type=code
  &client_id=YOUR_CLIENT_ID
  &scope=ZohoCRM.modules.ALL,ZohoCRM.settings.ALL,ZohoProjects.projects.ALL,ZohoProjects.tasks.ALL,ZohoMeeting.recording.READ,ZohoMeeting.meeting.READ,ZohoMeeting.meetinguds.READ,ZohoFiles.files.READ
  &redirect_uri=https://localhost/callback
  &access_type=offline
  &prompt=consent
```

> **重要：** 使用与您数据中心匹配的 accounts URL：
> | 区域 | Accounts URL |
> |--------|-------------|
> | US | `https://accounts.zoho.com` |
> | EU | `https://accounts.zoho.eu` |
> | IN | `https://accounts.zoho.in` |
> | AU | `https://accounts.zoho.com.au` |
> | JP | `https://accounts.zoho.jp` |
> | UK | `https://accounts.zoho.uk` |
> | CA | `https://accounts.zohocloud.ca` |
> | SA | `https://accounts.zoho.sa` |

授权后，您将被重定向到类似如下的 URL：
```
https://localhost/callback?code=1000.abc123...&location=us&accounts-server=https://accounts.zoho.com
```

复制 `code` 参数值。**此 code 在 2 分钟后过期** —— 立即进入步骤 3。

### 步骤 3：用 Code 换取 Refresh Token

运行此 curl 命令（替换占位符）：

```bash
curl -X POST "https://accounts.zoho.com/oauth/v2/token" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "grant_type=authorization_code" \
  -d "redirect_uri=https://localhost/callback" \
  -d "code=PASTE_CODE_FROM_STEP_2"
```

响应：
```json
{
  "access_token": "1000.xxxx.yyyy",
  "refresh_token": "1000.xxxx.zzzz",
  "api_domain": "https://www.zohoapis.com",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

保存 **refresh_token** —— 这是您的长期凭证。access token 在 1 小时后过期，但 CLI 会使用 refresh token 自动刷新它。

### 步骤 4：查找您的 Org ID

**CRM/Projects Org ID：**
```bash
# 在使用 client_id、client_secret、refresh_token 设置 .env 后：
zoho raw GET /crm/v7/org | jq '.org[0].id'
```

**Meeting Org ID：**
登录 [Zoho Meeting](https://meeting.zoho.com) → 管理员设置 → 在 URL 或设置页面中查找 Organization ID。它与 CRM org ID 不同。

### 步骤 5：配置 .env

在技能目录中创建 `.env`：

```bash
ZOHO_CLIENT_ID=1000.XXXXXXXXXXXXXXXXXXXXXXXXX
ZOHO_CLIENT_SECRET=your_client_secret_here
ZOHO_REFRESH_TOKEN=1000.your_refresh_token_here
ZOHO_ORG_ID=123456789              # CRM/Projects org ID
ZOHO_MEETING_ORG_ID=987654321      # Meeting org ID（与 CRM 不同）
ZOHO_CRM_DOMAIN=https://www.zohoapis.com
ZOHO_PROJECTS_DOMAIN=https://projectsapi.zoho.com/restapi
ZOHO_MEETING_DOMAIN=https://meeting.zoho.com
ZOHO_ACCOUNTS_URL=https://accounts.zoho.com
```

> 如果您在非 US 数据中心，请调整域名 URL（例如 `.eu`、`.in`、`.com.au`）。

### OAuth 作用域参考

| 作用域 | 用途 |
|-------|----------|
| `ZohoCRM.modules.ALL` | 读/写 CRM 记录（Deals、Contacts、Leads 等） |
| `ZohoCRM.settings.ALL` | 读取 CRM 字段定义和组织设置 |
| `ZohoProjects.projects.ALL` | 读/写项目 |
| `ZohoProjects.tasks.ALL` | 读/写任务、里程碑、缺陷、工时记录 |
| `ZohoMeeting.recording.READ` | 列出和访问会议录像 |
| `ZohoMeeting.meeting.READ` | 列出会议和会话详情 |
| `ZohoMeeting.meetinguds.READ` | 下载录像文件 |
| `ZohoFiles.files.READ` | 下载文件（录像、转录文本） |

如果您只需要 CRM 或只需要 Meeting，您可以请求更少的作用域。授权 URL 的作用域参数是逗号分隔的。

### 认证故障排除

- **"invalid_code"** → 授权码已过期（2 分钟有效期）。重做步骤 2。
- **"invalid_client"** → Client ID 错误，或您数据中心的 accounts-server URL 错误。
- **"invalid_redirect_uri"** → curl 中的 redirect_uri 必须与您在 API Console 中注册的完全匹配。
- **Token refresh fails** → Refresh token 可能被撤销。重做步骤 2-3 以获取新的。
- **"Given URL is wrong"** → 您访问了错误数据中心的 API 域名。

## CRM 命令

```bash
# 列出任何模块的记录
zoho crm list Deals
zoho crm list Deals "page=1&per_page=5&sort_by=Created_Time&sort_order=desc"
zoho crm list Contacts
zoho crm list Leads

# 获取特定记录
zoho crm get Deals 1234567890

# 使用条件搜索
zoho crm search Deals "(Stage:equals:Closed Won)"
zoho crm search Contacts "(Email:contains:@acme.com)"
zoho crm search Leads "(Lead_Source:equals:Web)"

# 创建记录
zoho crm create Contacts '{"data":[{"Last_Name":"Smith","First_Name":"John","Email":"j@co.com"}]}'
zoho crm create Deals '{"data":[{"Deal_Name":"New Project","Stage":"Qualification","Amount":50000}]}'

# 更新记录
zoho crm update Deals 1234567890 '{"data":[{"Stage":"Closed Won"}]}'

# 删除记录
zoho crm delete Deals 1234567890
```

### CRM 模块
Leads、Contacts、Accounts、Deals、Tasks、Events、Calls、Notes、Products、Quotes、Sales_Orders、Purchase_Orders、Invoices

### 搜索运算符
equals、not_equal、starts_with、contains、not_contains、in、not_in、between、greater_than、less_than

## Projects 命令

```bash
# 列出所有项目
zoho proj list

# 获取项目详情
zoho proj get 12345678

# 任务
zoho proj tasks 12345678
zoho proj create-task 12345678 "name=Fix+login+bug&priority=High&start_date=01-27-2026"
zoho proj update-task 12345678 98765432 "percent_complete=50"

# 其他
zoho proj milestones 12345678
zoho proj tasklists 12345678
zoho proj bugs 12345678
zoho proj timelogs 12345678
```

### 任务字段
name、start_date (MM-DD-YYYY)、end_date、priority (None/Low/Medium/High)、owner、description、tasklist_id、percent_complete

## Meeting 命令

```bash
# 列出所有录像
zoho meeting recordings
zoho meeting recordings | jq '[.recordings[] | {topic, sDate, sTime, durationInMins, erecordingId}]'

# 下载录像（使用录像列表中的 downloadUrl）
zoho meeting download "https://files-accl.zohopublic.com/public?event-id=..." output.mp4

# 列出会议/会话
zoho meeting list
zoho meeting list "fromDate=2026-01-01T00:00:00Z&toDate=2026-01-31T23:59:59Z"

# 获取会议详情
zoho meeting get 1066944216
```

### 录像响应字段
`zoho meeting recordings` 的关键字段：
- `erecordingId` — 加密录像 ID（用于去重/跟踪）
- `topic` — 会议标题
- `sDate`、`sTime` — 开始日期/时间（人类可读）
- `startTimeinMs` — 开始时间的 epoch 毫秒（用于日期筛选）
- `durationInMins` — 录像时长
- `downloadUrl` / `publicDownloadUrl` — MP4 下载 URL
- `transcriptionDownloadUrl` — Zoho 生成的转录文本（如果可用）
- `summaryDownloadUrl` — Zoho 生成的摘要（如果可用）
- `fileSize` / `fileSizeInMB` — 录像文件大小
- `status` — 例如 `UPLOADED`
- `meetingKey` — 会议标识符
- `creatorName` — 谁开始了录像

### 会议录像管道
用于自动化 standup/会议摘要：

```bash
# 1. 列出录像，按今天的日期筛选（epoch 毫秒）
zoho meeting recordings | jq --argjson start "$START_MS" --argjson end "$END_MS" \
  '[.recordings[] | select(.startTimeinMs >= $start and .startTimeinMs <= $end)]'

# 2. 下载录像
zoho meeting download "$DOWNLOAD_URL" /tmp/recording.mp4

# 3. 提取音频
ffmpeg -i /tmp/recording.mp4 -vn -acodec pcm_s16le -ar 16000 -ac 1 /tmp/audio.wav -y

# 4. 通过 Gemini Flash API 转录（非常适合阿拉伯语 + 英语混合）
# 有关完整实现，请参见 scripts/standup-summarizer.sh

# 5. 使用 Claude/GPT 总结转录文本
# 6. 清理临时文件
```

完整的 standup 摘要脚本包含在 `scripts/standup-summarizer.sh`。

## 原始 API 调用

对于未被子命令覆盖的任何内容：
```bash
# CRM 端点
zoho raw GET /crm/v7/settings/fields?module=Deals
zoho raw GET /crm/v7/org

# Meeting 端点
zoho raw GET "https://meeting.zoho.com/meeting/api/v2/{zsoid}/recordings.json"

# 自定义模块
zoho raw GET /crm/v7/Custom_Module
```

## 使用模式

### 查看 deals/pipeline 时
```bash
zoho crm list Deals "sort_by=Created_Time&sort_order=desc&per_page=10" | jq '.data[] | {Deal_Name, Stage, Amount, Closing_Date}'
```

### 查看项目进度时
```bash
zoho proj list | jq '.projects[] | {name, status, id: .id_string}'
zoho proj tasks <project_id> | jq '.tasks[] | {name, status: .status.name, percent_complete, priority}'
```

### 从对话中创建任务时
```bash
zoho proj create-task <project_id> "name=Task+description&priority=High&start_date=MM-DD-YYYY&end_date=MM-DD-YYYY"
```

### 总结会议录像时
```bash
# 最近的录像快速列表
zoho meeting recordings | jq '[.recordings[:5] | .[] | {topic, sDate, sTime, durationInMins, fileSize}]'

# 下载最新录像
URL=$(zoho meeting recordings | jq -r '.recordings[0].downloadUrl')
zoho meeting download "$URL" /tmp/latest.mp4
```

## 速率限制
- CRM: 100 请求/分钟
- Projects: 因套餐而异
- Meeting: 标准 API 限制
- Token 刷新：按需调用（自动缓存）

## 参考
- [CRM API 字段](references/crm-api.md)
- [Projects API 端点](references/projects-api.md)
- [Meeting API 参考](references/meeting-api.md)
