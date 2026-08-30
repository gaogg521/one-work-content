---
name: salesforce-api
description: Salesforce CRM API 集成，支持托管 OAuth。使用 SOQL 查询记录，管理 sObjects（Contacts、Accounts、Leads、Opportunities），并执行批量操作。 当用户想要与 Salesforce 数据交互时使用此技能。对于其他第三方应用，请使用 api-gateway 技能 (https://clawhub.ai/byungkyu/api-gateway)。
---

# Salesforce

通过托管 OAuth 认证访问 Salesforce REST API。使用 SOQL 查询记录，管理 sObjects，并对你的 Salesforce 数据执行 CRUD 操作。

## 快速开始

```bash
# Query contacts
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://gateway.maton.ai/salesforce/services/data/v59.0/query?q=SELECT+Id,Name,Email+FROM+Contact+LIMIT+10')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

## 基础 URL

```
https://gateway.maton.ai/salesforce/{native-api-path}
```

将 `{native-api-path}` 替换为实际的 Salesforce REST API 端点路径。网关将请求代理到 `{instance}.salesforce.com`（自动替换为你的连接配置）并注入你的 access token。

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

在 `https://ctrl.maton.ai` 管理你的 Salesforce OAuth 连接。

### 列出连接

```bash
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://ctrl.maton.ai/connections?app=salesforce&status=ACTIVE')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

### 创建连接

```bash
python <<'EOF'
import urllib.request, os, json
data = json.dumps({'app': 'salesforce'}).encode()
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
    "app": "salesforce",
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

如果你有多个 Salesforce 连接，使用 `Maton-Connection` header 指定使用哪一个：

```bash
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://gateway.maton.ai/salesforce/services/data/v59.0/sobjects')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
req.add_header('Maton-Connection', '21fd90f9-5935-43cd-b6c8-bde9d915ca80')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

如果省略，网关将使用默认的（最早的）活跃连接。

## API 参考

### SOQL 查询

```bash
GET /salesforce/services/data/v59.0/query?q=SELECT+Id,Name+FROM+Contact+LIMIT+10
```

复杂查询：

```bash
GET /salesforce/services/data/v59.0/query?q=SELECT+Id,Name,Email+FROM+Contact+WHERE+Email+LIKE+'%example.com'+ORDER+BY+CreatedDate+DESC
```

### 获取对象

```bash
GET /salesforce/services/data/v59.0/sobjects/{objectType}/{recordId}
```

示例：

```bash
GET /salesforce/services/data/v59.0/sobjects/Contact/003XXXXXXXXXXXXXXX
```

### 创建对象

```bash
POST /salesforce/services/data/v59.0/sobjects/{objectType}
Content-Type: application/json

{
  "FirstName": "John",
  "LastName": "Doe",
  "Email": "john@example.com"
}
```

### 更新对象

```bash
PATCH /salesforce/services/data/v59.0/sobjects/{objectType}/{recordId}
Content-Type: application/json

{
  "Phone": "+1234567890"
}
```

### 删除对象

```bash
DELETE /salesforce/services/data/v59.0/sobjects/{objectType}/{recordId}
```

### 描述对象（获取 schema）

```bash
GET /salesforce/services/data/v59.0/sobjects/{objectType}/describe
```

### 列出对象

```bash
GET /salesforce/services/data/v59.0/sobjects
```

### 搜索（SOSL）

```bash
GET /salesforce/services/data/v59.0/search?q=FIND+{searchTerm}+IN+ALL+FIELDS+RETURNING+Contact(Id,Name)
```

### 复合请求（批量多个操作）

```bash
POST /salesforce/services/data/v59.0/composite
Content-Type: application/json

{
  "compositeRequest": [
    {
      "method": "GET",
      "url": "/services/data/v59.0/sobjects/Contact/003XXXXXXX",
      "referenceId": "contact1"
    },
    {
      "method": "GET",
      "url": "/services/data/v59.0/sobjects/Account/001XXXXXXX",
      "referenceId": "account1"
    }
  ]
}
```

### 复合批量请求

```bash
POST /salesforce/services/data/v59.0/composite/batch
Content-Type: application/json

{
  "batchRequests": [
    {"method": "GET", "url": "v59.0/sobjects/Contact/003XXXXXXX"},
    {"method": "GET", "url": "v59.0/sobjects/Account/001XXXXXXX"}
  ]
}
```

### sObject 集合创建（批量创建）

```bash
POST /salesforce/services/data/v59.0/composite/sobjects
Content-Type: application/json

{
  "allOrNone": true,
  "records": [
    {"attributes": {"type": "Contact"}, "FirstName": "John", "LastName": "Doe"},
    {"attributes": {"type": "Contact"}, "FirstName": "Jane", "LastName": "Smith"}
  ]
}
```

### sObject 集合删除（批量删除）

```bash
DELETE /salesforce/services/data/v59.0/composite/sobjects?ids=003XXXXX,003YYYYY&allOrNone=true
```

### 获取更新的记录

```bash
GET /salesforce/services/data/v59.0/sobjects/{objectType}/updated/?start=2026-01-30T00:00:00Z&end=2026-02-01T00:00:00Z
```

### 获取删除的记录

```bash
GET /salesforce/services/data/v59.0/sobjects/{objectType}/deleted/?start=2026-01-30T00:00:00Z&end=2026-02-01T00:00:00Z
```

### 获取 API 限制

```bash
GET /salesforce/services/data/v59.0/limits
```

### 列出 API 版本

```bash
GET /salesforce/services/data/
```

## 常见对象

- `Account` - 公司/组织
- `Contact` - 与账户关联的人员
- `Lead` - 潜在客户
- `Opportunity` - 销售交易
- `Case` - 支持案例
- `Task` - 待办事项
- `Event` - 日历事件

## 代码示例

### JavaScript

```javascript
const response = await fetch(
  'https://gateway.maton.ai/salesforce/services/data/v59.0/query?q=SELECT+Id,Name+FROM+Contact+LIMIT+5',
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
    'https://gateway.maton.ai/salesforce/services/data/v59.0/query',
    headers={'Authorization': f'Bearer {os.environ["MATON_API_KEY"]}'},
    params={'q': 'SELECT Id,Name FROM Contact LIMIT 5'}
)
```

## 注意事项

- 对 SOQL 查询使用 URL 编码（空格变为 `+`）
- Record ID 是 15 或 18 位的字母数字字符串
- API 版本（v59.0）可以调整；最新为 v65.0
- Update 和 Delete 操作成功时返回 HTTP 204（无内容）
- updated/deleted 查询的日期使用 ISO 8601 格式：`YYYY-MM-DDTHH:MM:SSZ`
- 在批量操作中使用 `allOrNone: true` 以实现原子事务
- 重要：使用 curl 命令时，如果 URL 包含括号（`fields[]`、`sort[]`、`records[]`），请使用 `curl -g` 以禁用 glob 解析
- 重要：将 curl 输出通过管道传给 `jq` 或其他命令时，环境变量如 `$MATON_API_KEY` 在某些 shell 环境中可能无法正确展开。你可能会在管道使用时收到 "Invalid API key" 错误。

## 错误处理

| 状态码 | 含义 |
|--------|---------|
| 400 | 缺少 Salesforce 连接 |
| 401 | 无效或缺失的 Maton API key |
| 429 | 速率限制（每个账户每秒 10 个请求） |
| 4xx/5xx | Salesforce API 透传错误 |

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

1. 确保你的 URL 路径以 `salesforce` 开头。例如：

- 正确：`https://gateway.maton.ai/salesforce/services/data/v59.0/query`
- 错误：`https://gateway.maton.ai/services/data/v59.0/query`

## 资源

- [REST API Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_rest.htm)
- [List sObjects](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_describeGlobal.htm)
- [Describe sObject](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_sobject_describe.htm)
- [Get Record](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_sobject_retrieve_get.htm)
- [Create Record](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/dome_sobject_create.htm)
- [Update Record](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/dome_update_fields.htm)
- [Delete Record](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/dome_delete_record.htm)
- [Query Records (SOQL)](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_query.htm)
- [Composite Request](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_composite_composite_post.htm)
- [sObject Collections](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_composite_sobjects_collections_create.htm)
- [SOQL Reference](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql.htm)
- [SOSL Reference](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_sosl.htm)
- [Maton Community](https://discord.com/invite/dBfFAcefs2)
- [Maton Support](mailto:support@maton.ai)
