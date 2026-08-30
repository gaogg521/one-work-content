---
name: instantly
description: Instantly API 集成，支持托管 OAuth。用于管理活动、线索、账户和分析的冷邮件外联平台。 当用户想要创建活动、管理线索、发送邮件或查看分析时使用此技能。 对于其他第三方应用，请使用 api-gateway 技能 (https://clawhub.ai/byungkyu/api-gateway)。
compatibility: Requires network access and valid Maton API key
metadata: None
author: maton
clawdbot: None
emoji: 🧠
homepage: https://maton.ai
requires: None
env:
- MATON_API_KEY
version: 1.0
tags:
- AI
- API
---

# Instantly

通过托管认证访问 Instantly API v2。管理冷邮件活动、线索、发送账户和查看分析。

## 快速开始

```bash
# 列出活动
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://gateway.maton.ai/instantly/api/v2/campaigns?limit=10')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

## Base URL

```
https://gateway.maton.ai/instantly/{native-api-path}
```

将 `{native-api-path}` 替换为实际的 Instantly API 端点路径。网关将请求代理到 `api.instantly.ai` 并自动注入你的 API key。

## 认证

所有请求都需要在 Authorization header 中提供 Maton API key：

```
Authorization: Bearer $MATON_API_KEY
```

**环境变量:** 将你的 API key 设置为 `MATON_API_KEY`：

```bash
export MATON_API_KEY="YOUR_API_KEY"
```

### 获取你的 API Key

1. 在 [maton.ai](https://maton.ai) 登录或创建账户
2. 前往 [maton.ai/settings](https://maton.ai/settings)
3. 复制你的 API key

## 连接管理

在 `https://ctrl.maton.ai` 管理你的 Instantly 连接。

### 列出连接

```bash
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://ctrl.maton.ai/connections?app=instantly&status=ACTIVE')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

### 创建连接

```bash
python <<'EOF'
import urllib.request, os, json
data = json.dumps({'app': 'instantly'}).encode()
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

**响应:**
```json
{
  "connection": {
    "connection_id": "e4dca622-b9cf-4ed6-b52e-fa681345f5ac",
    "status": "ACTIVE",
    "creation_time": "2026-02-11T22:19:35.798712Z",
    "last_updated_time": "2026-02-11T22:20:15.702846Z",
    "url": "https://connect.maton.ai/?session_token=...",
    "app": "instantly",
    "metadata": {}
  }
}
```

在浏览器中打开返回的 `url` 以完成授权。

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

如果你有多个 Instantly 连接，使用 `Maton-Connection` header 指定使用哪一个：

```bash
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://gateway.maton.ai/instantly/api/v2/campaigns')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
req.add_header('Maton-Connection', 'e4dca622-b9cf-4ed6-b52e-fa681345f5ac')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

如果省略，网关将使用默认的（最早的）活跃连接。

## API 参考

### 活动 (Campaigns)

#### 列出活动

```bash
GET /instantly/api/v2/campaigns?limit=10&status=1&search=keyword
```

查询参数:
- `limit` - 结果数量 (默认: 10)
- `status` - 活动状态过滤 (0=草稿, 1=进行中, 2=已暂停, 3=已完成)
- `search` - 按活动名称搜索
- `starting_after` - 分页游标

#### 获取活动

```bash
GET /instantly/api/v2/campaigns/{campaign_id}
```

#### 创建活动

```bash
POST /instantly/api/v2/campaigns
Content-Type: application/json

{
  "name": "My Campaign",
  "campaign_schedule": {
    "schedules": [
      {
        "name": "My Schedule",
        "timing": {
          "from": "09:00",
          "to": "17:00"
        },
        "days": {
          "0": true,
          "1": true,
          "2": true,
          "3": true,
          "4": true
        },
        "timezone": "Etc/GMT+5"
      }
    ]
  }
}
```

注意: 时区必须使用 Etc/GMT 格式 (例如, "Etc/GMT+5", "Etc/GMT-8", "Etc/GMT+12")。
```

#### 激活活动

```bash
POST /instantly/api/v2/campaigns/{campaign_id}/activate
```

#### 暂停活动

```bash
POST /instantly/api/v2/campaigns/{campaign_id}/pause
```

#### 删除活动

```bash
DELETE /instantly/api/v2/campaigns/{campaign_id}
```

#### 按线索邮箱搜索活动

```bash
GET /instantly/api/v2/campaigns/search-by-contact?search=lead@example.com
```

### 线索 (Leads)

#### 创建线索

```bash
POST /instantly/api/v2/leads
Content-Type: application/json

{
  "campaign_id": "019bb3bd-9963-789e-b776-6c6927ef3f79",
  "email": "lead@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "company_name": "Acme Inc",
  "variables": {
    "custom_field": "custom_value"
  }
}
```

#### 批量添加线索

```bash
POST /instantly/api/v2/leads
Content-Type: application/json

{
  "campaign_id": "019bb3bd-9963-789e-b776-6c6927ef3f79",
  "leads": [
    {
      "email": "lead1@example.com",
      "first_name": "John"
    },
    {
      "email": "lead2@example.com",
      "first_name": "Jane"
    }
  ]
}
```

#### 列出线索

注意: 由于复杂的过滤需求，这是一个 POST 端点。

```bash
POST /instantly/api/v2/leads/list
Content-Type: application/json

{
  "campaign_id": "019bb3bd-9963-789e-b776-6c6927ef3f79",
  "limit": 100
}
```

#### 获取线索

```bash
GET /instantly/api/v2/leads/{lead_id}
```

#### 删除线索

```bash
DELETE /instantly/api/v2/leads/{lead_id}
```

#### 移动线索

```bash
POST /instantly/api/v2/leads/move
Content-Type: application/json

{
  "lead_ids": ["lead_id_1", "lead_id_2"],
  "to_campaign_id": "target_campaign_id"
}
```

### 线索列表 (Lead Lists)

#### 列出线索列表

```bash
GET /instantly/api/v2/lead-lists?limit=10
```

#### 创建线索列表

```bash
POST /instantly/api/v2/lead-lists
Content-Type: application/json

{
  "name": "My Lead List"
}
```

#### 获取线索列表

```bash
GET /instantly/api/v2/lead-lists/{list_id}
```

#### 更新线索列表

```bash
PATCH /instantly/api/v2/lead-lists/{list_id}
Content-Type: application/json

{
  "name": "Updated List Name"
}
```

#### 删除线索列表

```bash
DELETE /instantly/api/v2/lead-lists/{list_id}
```

### 账户 (发送邮件账户)

#### 列出账户

```bash
GET /instantly/api/v2/accounts?limit=10
```

#### 获取账户

```bash
GET /instantly/api/v2/accounts/{email}
```

#### 创建账户

```bash
POST /instantly/api/v2/accounts
Content-Type: application/json

{
  "email": "sender@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "provider_code": "google",
  "smtp_host": "smtp.gmail.com",
  "smtp_port": 587,
  "smtp_username": "sender@example.com",
  "smtp_password": "app_password",
  "imap_host": "imap.gmail.com",
  "imap_port": 993,
  "imap_username": "sender@example.com",
  "imap_password": "app_password"
}
```

#### 更新账户

```bash
PATCH /instantly/api/v2/accounts/{email}
Content-Type: application/json

{
  "first_name": "Jane"
}
```

#### 删除账户

```bash
DELETE /instantly/api/v2/accounts/{email}
```

#### 启用预热

```bash
POST /instantly/api/v2/accounts/warmup/enable
Content-Type: application/json

{
  "emails": ["account1@example.com", "account2@example.com"]
}
```

#### 禁用预热

```bash
POST /instantly/api/v2/accounts/warmup/disable
Content-Type: application/json

{
  "emails": ["account1@example.com"]
}
```

### 邮件 (Unibox)

#### 列出邮件

```bash
GET /instantly/api/v2/emails?limit=20
```

#### 获取邮件

```bash
GET /instantly/api/v2/emails/{email_id}
```

#### 回复邮件

```bash
POST /instantly/api/v2/emails/reply
Content-Type: application/json

{
  "reply_to_uuid": "email_uuid",
  "body": "Thank you for your response!"
}
```

#### 转发邮件

```bash
POST /instantly/api/v2/emails/forward
Content-Type: application/json

{
  "email_uuid": "email_uuid",
  "to": "forward@example.com"
}
```

#### 将线程标记为已读

```bash
POST /instantly/api/v2/emails/threads/{thread_id}/mark-as-read
```

#### 获取未读数量

```bash
GET /instantly/api/v2/emails/unread/count
```

#### 更新邮件

```bash
PATCH /instantly/api/v2/emails/{email_id}
Content-Type: application/json

{
  "is_read": true
}
```

#### 删除邮件

```bash
DELETE /instantly/api/v2/emails/{email_id}
```

### 分析

#### 获取活动分析

```bash
GET /instantly/api/v2/campaigns/analytics?id={campaign_id}
```

查询参数:
- `id` - 活动 ID (留空表示所有活动)
- `start_date` - 过滤开始日期 (YYYY-MM-DD)
- `end_date` - 过滤结束日期 (YYYY-MM-DD)
- `exclude_total_leads_count` - 设置为 true 以获得更快的响应

#### 获取活动分析概览

```bash
GET /instantly/api/v2/campaigns/analytics/overview?id={campaign_id}
```

#### 获取每日活动分析

```bash
GET /instantly/api/v2/campaigns/analytics/daily?id={campaign_id}
```

#### 获取活动步骤分析

```bash
GET /instantly/api/v2/campaigns/analytics/steps?id={campaign_id}
```

#### 获取预热分析

```bash
POST /instantly/api/v2/accounts/warmup/analytics
Content-Type: application/json

{
  "emails": ["account@example.com"]
}
```

### 阻止列表

#### 列出阻止列表条目

```bash
GET /instantly/api/v2/block-lists-entries?limit=100
```

查询参数:
- `domains_only` - 仅过滤域名条目
- `search` - 搜索条目

#### 创建阻止列表条目

```bash
POST /instantly/api/v2/block-lists-entries
Content-Type: application/json

{
  "bl_value": "blocked@example.com"
}
```

或阻止一个域名:

```bash
POST /instantly/api/v2/block-lists-entries
Content-Type: application/json

{
  "bl_value": "blockeddomain.com"
}
```

#### 删除阻止列表条目

```bash
DELETE /instantly/api/v2/block-lists-entries/{entry_id}
```

### 邮件验证

#### 验证邮件

```bash
GET /instantly/api/v2/email-verification/{email}
```

如果验证时间超过 10 秒，状态将为 `pending`。轮询此端点以检查状态。

响应字段:
- `verification_status` - 使用此字段（而不是 `status`）来确定验证结果

### 后台任务

#### 获取后台任务状态

```bash
GET /instantly/api/v2/background-jobs/{job_id}
```

查询参数:
- `data_fields` - 逗号分隔的字段 (例如, `success_count,failed_count,total_to_process`)

### 工作区

#### 获取当前工作区

```bash
GET /instantly/api/v2/workspaces/current
```

### 自定义标签

#### 在资源上切换标签

```bash
POST /instantly/api/v2/custom-tags/toggle-resource
Content-Type: application/json

{
  "tag_id": "tag_uuid",
  "resource_id": "campaign_or_account_id",
  "resource_type": "campaign"
}
```

## 分页

Instantly 使用基于游标的分页，使用 `limit` 和 `starting_after`：

```bash
GET /instantly/api/v2/campaigns?limit=10&starting_after=cursor_value
```

响应包含分页信息：

```json
{
  "items": [...],
  "next_starting_after": "cursor_for_next_page"
}
```

在下一个请求的 `starting_after` 参数中使用 `next_starting_after` 值。

## 代码示例

### JavaScript

```javascript
const response = await fetch(
  'https://gateway.maton.ai/instantly/api/v2/campaigns?limit=10',
  {
    headers: {
      'Authorization': `Bearer ${process.env.MATON_API_KEY}`
    }
  }
);
const data = await response.json();
```

### Python

```python
import os
import requests

response = requests.get(
    'https://gateway.maton.ai/instantly/api/v2/campaigns',
    headers={'Authorization': f'Bearer {os.environ["MATON_API_KEY"]}'},
    params={'limit': 10}
)
data = response.json()
```

## 注意事项

- Instantly API v2 对所有字段名使用 snake_case
- 线索自定义变量必须是 string、number、boolean 或 null（不能是 objects/arrays）
- List Leads 端点是 POST（不是 GET），因为过滤需求复杂
- 活动状态值: 0=草稿, 1=进行中, 2=已暂停, 3=已完成
- 邮件验证如果超过 10 秒可能返回 `pending` 状态
- 预热操作返回 background job ID - 轮询 background jobs 端点获取状态
- 重要: 使用 curl 命令时，如果 URL 包含括号，请使用 `curl -g` 以禁用 glob 解析
- 重要: 将 curl 输出通过管道传递给 `jq` 时，环境变量可能无法正确展开。请改用 Python 示例。

## 错误处理

| 状态 | 含义 |
|--------|---------|
| 400 | 缺少 Instantly 连接 或 无效请求 |
| 401 | 无效 或 缺失 Maton API key |
| 403 | API key 权限不足 |
| 429 | 速率限制 |
| 4xx/5xx | 来自 Instantly API 的透传错误 |

### 故障排除: API Key 问题

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

### 故障排除: 无效应用名称

1. 确保你的 URL 路径以 `instantly` 开头。例如：

- 正确: `https://gateway.maton.ai/instantly/api/v2/campaigns`
- 错误: `https://gateway.maton.ai/api/v2/campaigns`

## 资源

- [Instantly API V2 文档](https://developer.instantly.ai/api/v2)
- [Instantly API 介绍](https://developer.instantly.ai/)
- [Instantly 帮助中心](https://help.instantly.ai/)
- [Maton 社区](https://discord.com/invite/dBfFAcefs2)
- [Maton 支持](mailto:support@maton.ai)
