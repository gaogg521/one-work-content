---
name: clawemail
description: 通过 ClawEmail.com 服务使用 Google Workspace — Gmail、Drive、Docs、Sheets、Slides、Calendar、Forms。当用户要求发送电子邮件、创建文档、管理文件、安排事件或使用任何 Google 服务时，主动使用。
metadata:
  openclaw:
    emoji: 🦞
    requires:
      env:
      - CLAWEMAIL_CREDENTIALS
    primaryEnv: CLAWEMAIL_CREDENTIALS
tags:
- 办公
- 文档
- 管理
---

# Claw — 面向 AI 代理的 Google Workspace

通过你的 @clawemail.com 账户使用 `claw` 处理 Gmail、Drive、Docs、Sheets、Slides、Calendar 和 Forms。

## 设置

1. 将你的 ClawEmail 凭证 JSON 保存到 `~/.config/clawemail/credentials.json`
2. 设置环境变量：`export CLAWEMAIL_CREDENTIALS=~/.config/clawemail/credentials.json`

在 https://clawemail.com 获取凭证 — 注册，然后访问 `/connect/YOUR_PREFIX` 以授权 OAuth。

## 获取访问令牌

所有 API 调用都需要 Bearer 令牌。使用辅助脚本刷新并缓存它：

```bash
TOKEN=$(~/.openclaw/skills/clawemail/scripts/token.sh)
```

该脚本将令牌缓存 50 分钟。在进行 API 调用之前，始终将其分配给 `TOKEN`。

---

## Gmail

### 搜索电子邮件

```bash
TOKEN=$(~/.openclaw/skills/clawemail/scripts/token.sh)
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://gmail.googleapis.com/gmail/v1/users/me/messages?q=newer_than:7d&maxResults=10" | python3 -m json.tool
```

常用查询运算符：`from:`、`to:`、`subject:`、`newer_than:`、`older_than:`、`is:unread`、`has:attachment`、`label:`、`in:inbox`。

### 阅读邮件

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://gmail.googleapis.com/gmail/v1/users/me/messages/MESSAGE_ID?format=full" | python3 -m json.tool
```

对于纯文本正文，请使用 `format=minimal` 并解码 payload。对于可读的输出：

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://gmail.googleapis.com/gmail/v1/users/me/messages/MESSAGE_ID?format=full" \
  | python3 -c "
import json,sys,base64
m=json.load(sys.stdin)
hdrs={h['name']:h['value'] for h in m['payload']['headers']}
print(f\"From: {hdrs.get('From','')}\nTo: {hdrs.get('To','')}\nSubject: {hdrs.get('Subject','')}\nDate: {hdrs.get('Date','')}\n\")
def get_body(part):
    if part.get('body',{}).get('data'):
        return base64.urlsafe_b64decode(part['body']['data']).decode('utf-8','replace')
    for p in part.get('parts',[]):
        if p['mimeType']=='text/plain': return get_body(p)
    for p in part.get('parts',[]):
        b=get_body(p)
        if b: return b
    return ''
print(get_body(m['payload']))
"
```

### 发送电子邮件

```bash
TOKEN=$(~/.openclaw/skills/clawemail/scripts/token.sh)
python3 -c "
import base64,json
raw = base64.urlsafe_b64encode(
    b'To: recipient@example.com\r\nSubject: Hello\r\nContent-Type: text/plain; charset=utf-8\r\n\r\nMessage body here'
).decode()
print(json.dumps({'raw': raw}))
" | curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d @- \
  "https://gmail.googleapis.com/gmail/v1/users/me/messages/send"
```

对于 HTML 电子邮件，将 `Content-Type: text/plain` 替换为 `Content-Type: text/html` 并在正文中使用 HTML。

### 回复邮件

与发送相同，但添加来自原始邮件的 `In-Reply-To:` 和 `References:` 头，并在 JSON 正文中包含 `threadId`：

```bash
python3 -c "
import base64,json
raw = base64.urlsafe_b64encode(
    b'To: recipient@example.com\r\nSubject: Re: Original Subject\r\nIn-Reply-To: <original-message-id>\r\nReferences: <original-message-id>\r\nContent-Type: text/plain; charset=utf-8\r\n\r\nReply body'
).decode()
print(json.dumps({'raw': raw, 'threadId': 'THREAD_ID'}))
" | curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d @- \
  "https://gmail.googleapis.com/gmail/v1/users/me/messages/send"
```

### 列出标签

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://gmail.googleapis.com/gmail/v1/users/me/labels" | python3 -m json.tool
```

### 添加/移除标签

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"addLabelIds":["LABEL_ID"],"removeLabelIds":["INBOX"]}' \
  "https://gmail.googleapis.com/gmail/v1/users/me/messages/MESSAGE_ID/modify"
```

---

## Google Drive

### 列出文件

```bash
TOKEN=$(~/.openclaw/skills/clawemail/scripts/token.sh)
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/drive/v3/files?pageSize=20&fields=files(id,name,mimeType,modifiedTime,size)&orderBy=modifiedTime desc" | python3 -m json.tool
```

### 搜索文件

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/drive/v3/files?q=name+contains+'report'&fields=files(id,name,mimeType,modifiedTime)" | python3 -m json.tool
```

查询运算符：`name contains 'term'`、`mimeType='application/vnd.google-apps.document'`、`'FOLDER_ID' in parents`、`trashed=false`、`modifiedTime > '2025-01-01'`。

常用 MIME 类型：
- 文档：`application/vnd.google-apps.document`
- 表格：`application/vnd.google-apps.spreadsheet`
- 演示文稿：`application/vnd.google-apps.presentation`
- 文件夹：`application/vnd.google-apps.folder`
- 表单：`application/vnd.google-apps.form`

### 创建文件夹

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"My Folder","mimeType":"application/vnd.google-apps.folder"}' \
  "https://www.googleapis.com/drive/v3/files?fields=id,name" | python3 -m json.tool
```

### 上传文件

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -F "metadata={\"name\":\"report.pdf\"};type=application/json" \
  -F "file=@/path/to/report.pdf;type=application/pdf" \
  "https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart&fields=id,name" | python3 -m json.tool
```

### 下载文件

对于 Google Docs/Sheets/Slides（导出）：

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/drive/v3/files/FILE_ID/export?mimeType=application/pdf" -o output.pdf
```

导出格式：`text/plain`、`text/html`、`application/pdf`、`application/vnd.openxmlformats-officedocument.wordprocessingml.document` (docx)、`text/csv` (sheets)。

对于二进制文件（下载）：

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/drive/v3/files/FILE_ID?alt=media" -o output.file
```

### 共享文件

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"role":"writer","type":"user","emailAddress":"user@example.com"}' \
  "https://www.googleapis.com/drive/v3/files/FILE_ID/permissions"
```

角色：`reader`、`commenter`、`writer`、`owner`。类型：`user`、`group`、`domain`、`anyone`。

### 删除文件

```bash
curl -s -X DELETE -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/drive/v3/files/FILE_ID"
```

---

## Google Docs

### 创建文档

```bash
TOKEN=$(~/.openclaw/skills/clawemail/scripts/token.sh)
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"My Document"}' \
  "https://docs.googleapis.com/v1/documents" | python3 -m json.tool
```

### 阅读文档

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://docs.googleapis.com/v1/documents/DOCUMENT_ID" | python3 -m json.tool
```

对于纯文本提取：

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://docs.googleapis.com/v1/documents/DOCUMENT_ID" \
  | python3 -c "
import json,sys
doc=json.load(sys.stdin)
text=''
for el in doc.get('body',{}).get('content',[]):
    for p in el.get('paragraph',{}).get('elements',[]):
        text+=p.get('textRun',{}).get('content','')
print(text)
"
```

### 向文档追加文本

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"requests":[{"insertText":{"location":{"index":1},"text":"Hello, world!\n"}}]}' \
  "https://docs.googleapis.com/v1/documents/DOCUMENT_ID:batchUpdate"
```

### 替换文档中的文本

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"requests":[{"replaceAllText":{"containsText":{"text":"OLD_TEXT","matchCase":true},"replaceText":"NEW_TEXT"}}]}' \
  "https://docs.googleapis.com/v1/documents/DOCUMENT_ID:batchUpdate"
```

### 插入标题

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"requests":[{"insertText":{"location":{"index":1},"text":"My Heading\n"}},{"updateParagraphStyle":{"range":{"startIndex":1,"endIndex":12},"paragraphStyle":{"namedStyleType":"HEADING_1"},"fields":"namedStyleType"}}]}' \
  "https://docs.googleapis.com/v1/documents/DOCUMENT_ID:batchUpdate"
```

标题样式：`HEADING_1` 到 `HEADING_6`、`TITLE`、`SUBTITLE`、`NORMAL_TEXT`。

---

## Google Sheets

### 创建电子表格

```bash
TOKEN=$(~/.openclaw/skills/clawemail/scripts/token.sh)
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"properties":{"title":"My Spreadsheet"}}' \
  "https://sheets.googleapis.com/v4/spreadsheets" | python3 -m json.tool
```

### 读取单元格

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://sheets.googleapis.com/v4/spreadsheets/SPREADSHEET_ID/values/Sheet1!A1:D10" | python3 -m json.tool
```

### 写入单元格

```bash
curl -s -X PUT -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"values":[["Name","Age","City"],["Alice","30","NYC"],["Bob","25","LA"]]}' \
  "https://sheets.googleapis.com/v4/spreadsheets/SPREADSHEET_ID/values/Sheet1!A1:C3?valueInputOption=USER_ENTERED" | python3 -m json.tool
```

### 追加行

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"values":[["Charlie","35","Chicago"]]}' \
  "https://sheets.googleapis.com/v4/spreadsheets/SPREADSHEET_ID/values/Sheet1!A:C:append?valueInputOption=USER_ENTERED&insertDataOption=INSERT_ROWS" | python3 -m json.tool
```

### 清除范围

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  "https://sheets.googleapis.com/v4/spreadsheets/SPREADSHEET_ID/values/Sheet1!A1:D10:clear"
```

### 获取电子表格元数据

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://sheets.googleapis.com/v4/spreadsheets/SPREADSHEET_ID?fields=properties.title,sheets.properties" | python3 -m json.tool
```

---

## Google Slides

### 创建演示文稿

```bash
TOKEN=$(~/.openclaw/skills/clawemail/scripts/token.sh)
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"My Presentation"}' \
  "https://slides.googleapis.com/v1/presentations" | python3 -m json.tool
```

### 获取演示文稿信息

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://slides.googleapis.com/v1/presentations/PRESENTATION_ID" | python3 -m json.tool
```

### 添加新幻灯片

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"requests":[{"createSlide":{"slideLayoutReference":{"predefinedLayout":"TITLE_AND_BODY"}}}]}' \
  "https://slides.googleapis.com/v1/presentations/PRESENTATION_ID:batchUpdate" | python3 -m json.tool
```

布局：`BLANK`、`TITLE`、`TITLE_AND_BODY`、`TITLE_AND_TWO_COLUMNS`、`TITLE_ONLY`、`SECTION_HEADER`、`ONE_COLUMN_TEXT`、`MAIN_POINT`、`BIG_NUMBER`。

### 向幻灯片添加文本

首先获取幻灯片的页面对象 ID，然后将文本插入占位符：

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"requests":[{"insertText":{"objectId":"PLACEHOLDER_OBJECT_ID","text":"Hello from ClawEmail!","insertionIndex":0}}]}' \
  "https://slides.googleapis.com/v1/presentations/PRESENTATION_ID:batchUpdate"
```

### 向幻灯片添加图片

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"requests":[{"createImage":{"url":"https://example.com/image.png","elementProperties":{"pageObjectId":"SLIDE_ID","size":{"width":{"magnitude":3000000,"unit":"EMU"},"height":{"magnitude":2000000,"unit":"EMU"}},"transform":{"scaleX":1,"scaleY":1,"translateX":1000000,"translateY":1500000,"unit":"EMU"}}}}]}' \
  "https://slides.googleapis.com/v1/presentations/PRESENTATION_ID:batchUpdate"
```

---

## Google Calendar

### 列出即将发生的事件

```bash
TOKEN=$(~/.openclaw/skills/clawemail/scripts/token.sh)
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/calendar/v3/calendars/primary/events?timeMin=$(date -u +%Y-%m-%dT%H:%M:%SZ)&maxResults=10&singleEvents=true&orderBy=startTime" | python3 -m json.tool
```

### 获取日期范围内的事件

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/calendar/v3/calendars/primary/events?timeMin=2025-03-01T00:00:00Z&timeMax=2025-03-31T23:59:59Z&singleEvents=true&orderBy=startTime" | python3 -m json.tool
```

### 创建事件

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "summary": "Team Meeting",
    "description": "Weekly standup",
    "start": {"dateTime": "2025-03-15T10:00:00-05:00", "timeZone": "America/New_York"},
    "end": {"dateTime": "2025-03-15T11:00:00-05:00", "timeZone": "America/New_York"},
    "attendees": [{"email": "colleague@example.com"}]
  }' \
  "https://www.googleapis.com/calendar/v3/calendars/primary/events" | python3 -m json.tool
```

### 更新事件

```bash
curl -s -X PATCH -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"summary":"Updated Meeting Title","location":"Conference Room A"}' \
  "https://www.googleapis.com/calendar/v3/calendars/primary/events/EVENT_ID" | python3 -m json.tool
```

### 删除事件

```bash
curl -s -X DELETE -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/calendar/v3/calendars/primary/events/EVENT_ID"
```

### 列出日历

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/calendar/v3/users/me/calendarList" | python3 -m json.tool
```

---

## Google Forms

### 创建表单

```bash
TOKEN=$(~/.openclaw/skills/clawemail/scripts/token.sh)
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"info":{"title":"Feedback Form"}}' \
  "https://forms.googleapis.com/v1/forms" | python3 -m json.tool
```

### 添加问题

```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"requests":[{"createItem":{"item":{"title":"How would you rate this?","questionItem":{"question":{"required":true,"scaleQuestion":{"low":1,"high":5,"lowLabel":"Poor","highLabel":"Excellent"}}}},"location":{"index":0}}}]}' \
  "https://forms.googleapis.com/v1/forms/FORM_ID:batchUpdate"
```

### 获取表单回复

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://forms.googleapis.com/v1/forms/FORM_ID/responses" | python3 -m json.tool
```

---

## 提示

- **始终先刷新令牌：** 在每个序列开始时使用 `TOKEN=$(~/.openclaw/skills/clawemail/scripts/token.sh)`
- **JSON 输出：** 通过 `python3 -m json.tool` 管道传输以获得可读的输出，或通过 `| python3 -c "import json,sys;..."` 进行提取
- **分页：** 大多数列表端点返回 `nextPageToken`。将其作为 `?pageToken=TOKEN` 传递以获取下一页
- **批量操作：** Docs、Sheets 和 Slides 支持 `batchUpdate` — 在一个请求中发送多个操作
- **错误 401：** 令牌已过期。重新运行 `token.sh` 以刷新
- **错误 403：** 范围未授权。ClawEmail OAuth 包括 Gmail、Drive、Docs、Sheets、Slides、Calendar 和 Forms 范围
- **速率限制：** Google API 有每个用户的速率限制。在快速连续调用之间添加短暂的延迟
- **文件 ID：** Google Docs/Sheets/Slides URL 包含文件 ID：`https://docs.google.com/document/d/FILE_ID/edit`

## 何时使用

- 用户要求发送、阅读或搜索电子邮件
- 用户想要创建或编辑文档、电子表格或演示文稿
- 用户需要管理 Google Drive 中的文件
- 用户想要安排或检查日历事件
- 用户要求创建表单或查看表单回复
- 任何涉及 Google Workspace 服务的任务
