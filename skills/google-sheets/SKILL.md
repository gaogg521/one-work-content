---
name: google-sheets
description: Google Sheets API 集成，支持托管 OAuth。读取和写入电子表格数据、创建表格、应用格式化和管理范围。 当用户想要读取或写入 Google Sheets 时使用此技能。对于其他第三方应用，请使用 api-gateway 技能 (https://clawhub.ai/byungkyu/api-gateway)。
compatibility: Requires network access and valid Maton API key
metadata: None
author: maton
clawdbot: None
emoji: 🧠
requires: None
env:
- MATON_API_KEY
version: 1.0
tags:
- AI
- API
---

# Google Sheets

通过托管 OAuth 认证访问 Google Sheets API。读取和写入电子表格值、创建表格、应用格式化和执行批量操作。

## 快速开始

```bash
# 从电子表格读取值（注意：range 需要 URL 编码）
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://gateway.maton.ai/google-sheets/v4/spreadsheets/SPREADSHEET_ID/values/Sheet1%21A1%3AD10')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

## 基础 URL

```
https://gateway.maton.ai/google-sheets/{native-api-path}
```

将 `{native-api-path}` 替换为实际的 Google Sheets API 端点路径。网关将请求代理到 `sheets.googleapis.com` 并自动注入你的 OAuth token。

## 认证

所有请求都需要在 Authorization header 中提供 Maton API key：

```
Authorization: Bearer $MATON_API_KEY
```

**环境变量：** 将你的 API key 设置为 `MATON_API_KEY`：

```bash
export MATON_API_KEY="YOUR_API_KEY"
```

### 获取你的 API Key

1. 在 [maton.ai](https://maton.ai) 登录或创建账户
2. 前往 [maton.ai/settings](https://maton.ai/settings)
3. 复制你的 API key

## 连接管理

在 `https://ctrl.maton.ai` 管理你的 Google OAuth 连接。

### 列出连接

```bash
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://ctrl.maton.ai/connections?app=google-sheets&status=ACTIVE')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

### 创建连接

```bash
python <<'EOF'
import urllib.request, os, json
data = json.dumps({'app': 'google-sheets'}).encode()
req = urllib.request.Request('https://ctrl.maton.ai/connections', data=data, method='POST')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
req.add_header('Content-Type', 'application/json')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

### 获取连接

```bash
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://ctrl.maton.ai/connections/{connection_id}')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

**响应：**
```json
{
  "connection": {
    "connection_id": "21fd90f9-5935-43cd-b6c8-bde9d915ca80",
    "status": "ACTIVE",
    "creation_time": "2025-12-08T07:20:53.488460Z",
    "last_updated_time": "2026-01-31T20:03:32.593153Z",
    "url": "https://connect.maton.ai/?session_token=...",
    "app": "google-sheets",
    "metadata": {}
  }
}
```

在浏览器中打开返回的 `url` 以完成 OAuth 授权。

### 删除连接

```bash
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://ctrl.maton.ai/connections/{connection_id}', method='DELETE')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

### 指定连接

如果你有多个 Google 账户已连接，使用 `Maton-Connection` header 指定使用哪一个：

```bash
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://gateway.maton.ai/google-sheets/v4/spreadsheets/SPREADSHEET_ID')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
req.add_header('Maton-Connection', '21fd90f9-5935-43cd-b6c8-bde9d915ca80')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

如果省略，网关将使用默认的（最早的）活跃连接。

## API 参考

### 获取电子表格元数据

```bash
GET /google-sheets/v4/spreadsheets/{spreadsheetId}
```

### 获取值

```bash
GET /google-sheets/v4/spreadsheets/{spreadsheetId}/values/{range}
```

示例：

```bash
GET /google-sheets/v4/spreadsheets/SHEET_ID/values/Sheet1%21A1%3AD10
```

### 获取多个范围

```bash
GET /google-sheets/v4/spreadsheets/{spreadsheetId}/values:batchGet?ranges=Sheet1%21A1%3AB10&ranges=Sheet2%21A1%3AC5
```

### 更新值

```bash
PUT /google-sheets/v4/spreadsheets/{spreadsheetId}/values/{range}?valueInputOption=USER_ENTERED
Content-Type: application/json

{
  "values": [
    ["A1", "B1", "C1"],
    ["A2", "B2", "C2"]
  ]
}
```

### 追加值

```bash
POST /google-sheets/v4/spreadsheets/{spreadsheetId}/values/{range}:append?valueInputOption=USER_ENTERED
Content-Type: application/json

{
  "values": [
    ["New Row 1", "Data", "More Data"],
    ["New Row 2", "Data", "More Data"]
  ]
}
```

### 批量更新值

```bash
POST /google-sheets/v4/spreadsheets/{spreadsheetId}/values:batchUpdate
Content-Type: application/json

{
  "valueInputOption": "USER_ENTERED",
  "data": [
    {"range": "Sheet1!A1:B2", "values": [["A1", "B1"], ["A2", "B2"]]},
    {"range": "Sheet1!D1:E2", "values": [["D1", "E1"], ["D2", "E2"]]}
  ]
}
```

### 清除值

```bash
POST /google-sheets/v4/spreadsheets/{spreadsheetId}/values/{range}:clear
```

### 创建电子表格

```bash
POST /google-sheets/v4/spreadsheets
Content-Type: application/json

{
  "properties": {"title": "New Spreadsheet"},
  "sheets": [{"properties": {"title": "Sheet1"}}]
}
```

### 批量更新（格式化、添加工作表等）

```bash
POST /google-sheets/v4/spreadsheets/{spreadsheetId}:batchUpdate
Content-Type: application/json

{
  "requests": [
    {"addSheet": {"properties": {"title": "New Sheet"}}}
  ]
}
```

## 常见 batchUpdate 请求

### 使用格式化更新单元格

```json
{
  "updateCells": {
    "rows": [
      {"values": [{"userEnteredValue": {"stringValue": "Name"}}, {"userEnteredValue": {"numberValue": 100}}]}
    ],
    "fields": "userEnteredValue",
    "start": {"sheetId": 0, "rowIndex": 0, "columnIndex": 0}
  }
}
```

### 格式化标题行（粗体 + 背景色）

```json
{
  "repeatCell": {
    "range": {"sheetId": 0, "startRowIndex": 0, "endRowIndex": 1, "startColumnIndex": 0, "endColumnIndex": 3},
    "cell": {
      "userEnteredFormat": {
        "backgroundColor": {"red": 0.2, "green": 0.6, "blue": 0.9},
        "textFormat": {"bold": true}
      }
    },
    "fields": "userEnteredFormat(backgroundColor,textFormat)"
  }
}
```

### 自动调整列宽

```json
{
  "autoResizeDimensions": {
    "dimensions": {"sheetId": 0, "dimension": "COLUMNS", "startIndex": 0, "endIndex": 3}
  }
}
```

### 重命名工作表

```json
{
  "updateSheetProperties": {
    "properties": {"sheetId": 0, "title": "NewName"},
    "fields": "title"
  }
}
```

### 插入行/列

```json
{
  "insertDimension": {
    "range": {"sheetId": 0, "dimension": "ROWS", "startIndex": 1, "endIndex": 3},
    "inheritFromBefore": true
  }
}
```

### 排序范围

```json
{
  "sortRange": {
    "range": {"sheetId": 0, "startRowIndex": 1, "endRowIndex": 10, "startColumnIndex": 0, "endColumnIndex": 3},
    "sortSpecs": [{"dimensionIndex": 1, "sortOrder": "DESCENDING"}]
  }
}
```

### 添加筛选器

```json
{
  "setBasicFilter": {
    "filter": {
      "range": {"sheetId": 0, "startRowIndex": 0, "endRowIndex": 100, "startColumnIndex": 0, "endColumnIndex": 5}
    }
  }
}
```

### 删除工作表

```json
{
  "deleteSheet": {"sheetId": 123456789}
}
```

## 值输入选项

- `RAW` - 值按原样存储
- `USER_ENTERED` - 值像输入到 UI 中一样被解析（公式执行、数字解析）

## 范围表示法

- `Sheet1!A1:D10` - 特定范围
- `Sheet1!A:D` - 整列 A 到 D
- `Sheet1!1:10` - 整行 1 到 10
- `Sheet1` - 整个工作表
- `A1:D10` - 第一个工作表中的范围

## 代码示例

### JavaScript

```javascript
// 读取值
const response = await fetch(
  'https://gateway.maton.ai/google-sheets/v4/spreadsheets/SHEET_ID/values/Sheet1!A1:D10',
  {
    headers: {
      'Authorization': `Bearer ${process.env.MATON_API_KEY}`
    }
  }
);

// 写入值
await fetch(
  'https://gateway.maton.ai/google-sheets/v4/spreadsheets/SHEET_ID/values/Sheet1!A1:B2?valueInputOption=USER_ENTERED',
  {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${process.env.MATON_API_KEY}`
    },
    body: JSON.stringify({
      values: [['A1', 'B1'], ['A2', 'B2']]
    })
  }
);
```

### Python

```python
import os
import requests

# 读取值
response = requests.get(
    'https://gateway.maton.ai/google-sheets/v4/spreadsheets/SHEET_ID/values/Sheet1!A1:D10',
    headers={'Authorization': f'Bearer {os.environ["MATON_API_KEY"]}'}
)

# 写入值
response = requests.put(
    'https://gateway.maton.ai/google-sheets/v4/spreadsheets/SHEET_ID/values/Sheet1!A1:B2',
    headers={'Authorization': f'Bearer {os.environ["MATON_API_KEY"]}'},
    params={'valueInputOption': 'USER_ENTERED'},
    json={'values': [['A1', 'B1'], ['A2', 'B2']]}
)
```

## 注意事项

- 使用 curl 时，URL 路径中的范围必须进行 URL 编码（! -> %21, : -> %3A）。JavaScript fetch 和 Python requests 会自动处理编码。
- 使用 `valueInputOption=USER_ENTERED` 来解析公式和数字
- 通过 Google Drive API 删除电子表格，而不是 Sheets API
- Sheet ID 是数字，可在电子表格元数据中找到
- IMPORTANT: 使用 curl 命令时，如果 URL 包含括号（`fields[]`, `sort[]`, `records[]`），请使用 `curl -g` 禁用 glob 解析
- IMPORTANT: 将 curl 输出通过管道传输到 `jq` 或其他命令时，环境变量如 `$MATON_API_KEY` 在某些 shell 环境中可能无法正确展开。你可能会在管道传输时收到 "Invalid API key" 错误。

## 错误处理

| Status | Meaning |
|--------|---------|
| 400 | Missing Google Sheets connection |
| 401 | Invalid or missing Maton API key |
| 429 | Rate limited (10 req/sec per account) |
| 4xx/5xx | Passthrough error from Google Sheets API |

### 故障排除：API Key 问题

1. 检查 `MATON_API_KEY` 环境变量是否已设置：

```bash
echo $MATON_API_KEY
```

2. 通过列出连接来验证 API key 是否有效：

```bash
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://ctrl.maton.ai/connections')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

### 故障排除：无效的应用名称

1. 确保你的 URL 路径以 `google-sheets` 开头。例如：

- Correct: `https://gateway.maton.ai/google-sheets/v4/spreadsheets/SPREADSHEET_ID`
- Incorrect: `https://gateway.maton.ai/v4/spreadsheets/SPREADSHEET_ID`

## 资源

- [Sheets API Overview](https://developers.google.com/workspace/sheets/api/reference/rest)
- [Get Spreadsheet](https://developers.google.com/workspace/sheets/api/reference/rest/v4/spreadsheets/get)
- [Create Spreadsheet](https://developers.google.com/workspace/sheets/api/reference/rest/v4/spreadsheets/create)
- [Batch Update](https://developers.google.com/workspace/sheets/api/reference/rest/v4/spreadsheets/batchUpdate)
- [Batch Update Request Types](https://developers.google.com/workspace/sheets/api/reference/rest/v4/spreadsheets/request)
- [Get Values](https://developers.google.com/workspace/sheets/api/reference/rest/v4/spreadsheets.values/get)
- [Update Values](https://developers.google.com/workspace/sheets/api/reference/rest/v4/spreadsheets.values/update)
- [Append Values](https://developers.google.com/workspace/sheets/api/reference/rest/v4/spreadsheets.values/append)
- [Batch Get Values](https://developers.google.com/workspace/sheets/api/reference/rest/v4/spreadsheets.values/batchGet)
- [Batch Update Values](https://developers.google.com/workspace/sheets/api/reference/rest/v4/spreadsheets.values/batchUpdate)
- [Maton Community](https://discord.com/invite/dBfFAcefs2)
- [Maton Support](mailto:support@maton.ai)
