---
name: google-docs-skill
description: 使用 OAuth 2.0 访问 Google Docs API，创建文档、插入和格式化文本以及管理文档内容
tags:
- API
---

# Google Docs API 集成

使用 OAuth 2.0 直接访问 Google Docs API。创建文档、插入和格式化文本，以及管理文档内容。

## 先决条件

1. **Google Cloud 项目设置**
   - 前往 [Google Cloud Console](https://console.cloud.google.com)
   - 创建新项目或选择现有项目
   - 启用 Google Docs API
   - 创建 OAuth 2.0 凭据（桌面应用或 Web 应用）
   - 下载凭据 JSON 文件

2. **环境设置**
   ```bash
   export GOOGLE_CLIENT_ID="your-client-id"
   export GOOGLE_CLIENT_SECRET="your-client-secret"
   export GOOGLE_REFRESH_TOKEN="your-refresh-token"
   ```

## 认证

### 获取 OAuth 令牌

使用以下 Python 脚本获取您的 refresh token（一次性设置）：

```python
import urllib.request
import urllib.parse
import json
import webbrowser
from http.server import HTTPServer, BaseHTTPRequestHandler
import os

# OAuth configuration
CLIENT_ID = os.environ.get('GOOGLE_CLIENT_ID')
CLIENT_SECRET = os.environ.get('GOOGLE_CLIENT_SECRET')
REDIRECT_URI = 'http://localhost:8080'
SCOPES = 'https://www.googleapis.com/auth/documents'

# Step 1: Get authorization code
auth_url = (
    f"https://accounts.google.com/o/oauth2/v2/auth?"
    f"client_id={CLIENT_ID}&"
    f"redirect_uri={REDIRECT_URI}&"
    f"response_type=code&"
    f"scope={urllib.parse.quote(SCOPES)}&"
    f"access_type=offline&"
    f"prompt=consent"
)

print(f"Opening browser for authorization...")
webbrowser.open(auth_url)

# Step 2: Capture authorization code
auth_code = None

class OAuthHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        global auth_code
        query = urllib.parse.urlparse(self.path).query
        params = urllib.parse.parse_qs(query)
        auth_code = params.get('code', [None])[0]
        
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(b'<html><body><h1>Authorization successful!</h1><p>You can close this window.</p></body></html>')

server = HTTPServer(('localhost', 8080), OAuthHandler)
server.handle_request()

# Step 3: Exchange code for tokens
if auth_code:
    data = urllib.parse.urlencode({
        'code': auth_code,
        'client_id': CLIENT_ID,
        'client_secret': CLIENT_SECRET,
        'redirect_uri': REDIRECT_URI,
        'grant_type': 'authorization_code'
    }).encode()
    
    req = urllib.request.Request('https://oauth2.googleapis.com/token', data=data)
    response = json.load(urllib.request.urlopen(req))
    
    print(f"\nRefresh Token: {response['refresh_token']}")
    print(f"Access Token: {response['access_token']}")
    print(f"\nSet your refresh token:")
    print(f"export GOOGLE_REFRESH_TOKEN=\"{response['refresh_token']}\"")
```

### 获取 Access Token

在进行 API 调用之前，获取一个新的 access token：

```python
import urllib.request
import urllib.parse
import json
import os

def get_access_token():
    """Get a fresh access token using refresh token"""
    data = urllib.parse.urlencode({
        'client_id': os.environ['GOOGLE_CLIENT_ID'],
        'client_secret': os.environ['GOOGLE_CLIENT_SECRET'],
        'refresh_token': os.environ['GOOGLE_REFRESH_TOKEN'],
        'grant_type': 'refresh_token'
    }).encode()
    
    req = urllib.request.Request('https://oauth2.googleapis.com/token', data=data)
    response = json.load(urllib.request.urlopen(req))
    return response['access_token']

# Store for reuse
access_token = get_access_token()
print(f"Access Token: {access_token}")
```

## Base URL

```
https://docs.googleapis.com/v1
```

所有请求都需要在 Authorization header 中携带 access token：

```
Authorization: Bearer {access_token}
```

## 快速开始

```python
import urllib.request
import json
import os

# Get access token first (using function from above)
access_token = get_access_token()

# Get document
req = urllib.request.Request('https://docs.googleapis.com/v1/documents/{documentId}')
req.add_header('Authorization', f'Bearer {access_token}')
doc = json.load(urllib.request.urlopen(req))
print(json.dumps(doc, indent=2))
```

## API 参考

### 创建文档

```python
import urllib.request
import json

access_token = get_access_token()

data = json.dumps({'title': 'My New Document'}).encode()
req = urllib.request.Request(
    'https://docs.googleapis.com/v1/documents',
    data=data,
    method='POST'
)
req.add_header('Authorization', f'Bearer {access_token}')
req.add_header('Content-Type', 'application/json')

response = json.load(urllib.request.urlopen(req))
doc_id = response['documentId']
print(f"Created document: {doc_id}")
print(f"URL: https://docs.google.com/document/d/{doc_id}/edit")
```

### 获取文档

```python
req = urllib.request.Request(
    f'https://docs.googleapis.com/v1/documents/{doc_id}'
)
req.add_header('Authorization', f'Bearer {access_token}')
doc = json.load(urllib.request.urlopen(req))
```

### 批量更新文档

```python
# Insert text at beginning
updates = {
    'requests': [
        {
            'insertText': {
                'location': {'index': 1},
                'text': 'Hello, World!\n\n'
            }
        }
    ]
}

data = json.dumps(updates).encode()
req = urllib.request.Request(
    f'https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate',
    data=data,
    method='POST'
)
req.add_header('Authorization', f'Bearer {access_token}')
req.add_header('Content-Type', 'application/json')

response = json.load(urllib.request.urlopen(req))
```

## 常见操作

### 插入文本

```python
{
    'requests': [
        {
            'insertText': {
                'location': {'index': 1},
                'text': 'Your text here'
            }
        }
    ]
}
```

### 格式化文本（粗体、斜体、字号）

```python
{
    'requests': [
        {
            'updateTextStyle': {
                'range': {
                    'startIndex': 1,
                    'endIndex': 10
                },
                'textStyle': {
                    'bold': True,
                    'italic': True,
                    'fontSize': {
                        'magnitude': 14,
                        'unit': 'PT'
                    }
                },
                'fields': 'bold,italic,fontSize'
            }
        }
    ]
}
```

### 插入表格

```python
{
    'requests': [
        {
            'insertTable': {
                'location': {'index': 1},
                'rows': 3,
                'columns': 4
            }
        }
    ]
}
```

### 替换文本

```python
{
    'requests': [
        {
            'replaceAllText': {
                'containsText': {
                    'text': '{{placeholder}}',
                    'matchCase': True
                },
                'replaceText': 'Actual value'
            }
        }
    ]
}
```

### 删除内容

```python
{
    'requests': [
        {
            'deleteContentRange': {
                'range': {
                    'startIndex': 1,
                    'endIndex': 50
                }
            }
        }
    ]
}
```

### 插入分页符

```python
{
    'requests': [
        {
            'insertPageBreak': {
                'location': {'index': 1}
            }
        }
    ]
}
```

### 更新段落样式

```python
{
    'requests': [
        {
            'updateParagraphStyle': {
                'range': {
                    'startIndex': 1,
                    'endIndex': 50
                },
                'paragraphStyle': {
                    'namedStyleType': 'HEADING_1',
                    'alignment': 'CENTER'
                },
                'fields': 'namedStyleType,alignment'
            }
        }
    ]
}
```

## 完整示例：创建并格式化文档

```python
import urllib.request
import urllib.parse
import json
import os

def get_access_token():
    """Get fresh access token"""
    data = urllib.parse.urlencode({
        'client_id': os.environ['GOOGLE_CLIENT_ID'],
        'client_secret': os.environ['GOOGLE_CLIENT_SECRET'],
        'refresh_token': os.environ['GOOGLE_REFRESH_TOKEN'],
        'grant_type': 'refresh_token'
    }).encode()
    
    req = urllib.request.Request('https://oauth2.googleapis.com/token', data=data)
    response = json.load(urllib.request.urlopen(req))
    return response['access_token']

def create_document(title, access_token):
    """Create a new document"""
    data = json.dumps({'title': title}).encode()
    req = urllib.request.Request(
        'https://docs.googleapis.com/v1/documents',
        data=data,
        method='POST'
    )
    req.add_header('Authorization', f'Bearer {access_token}')
    req.add_header('Content-Type', 'application/json')
    
    response = json.load(urllib.request.urlopen(req))
    return response['documentId']

def batch_update(doc_id, requests, access_token):
    """Apply batch updates to document"""
    data = json.dumps({'requests': requests}).encode()
    req = urllib.request.Request(
        f'https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate',
        data=data,
        method='POST'
    )
    req.add_header('Authorization', f'Bearer {access_token}')
    req.add_header('Content-Type', 'application/json')
    
    return json.load(urllib.request.urlopen(req))

# Main workflow
access_token = get_access_token()

# Create document
doc_id = create_document('Project Report', access_token)
print(f"Created: https://docs.google.com/document/d/{doc_id}/edit")

# Add content with formatting
requests = [
    # Insert title
    {
        'insertText': {
            'location': {'index': 1},
            'text': 'Project Report\n\n'
        }
    },
    # Format title as heading
    {
        'updateParagraphStyle': {
            'range': {'startIndex': 1, 'endIndex': 15},
            'paragraphStyle': {'namedStyleType': 'HEADING_1'},
            'fields': 'namedStyleType'
        }
    },
    # Add body text
    {
        'insertText': {
            'location': {'index': 16},
            'text': 'Executive Summary\n\nThis report covers...\n\n'
        }
    },
    # Format section header
    {
        'updateParagraphStyle': {
            'range': {'startIndex': 16, 'endIndex': 34},
            'paragraphStyle': {'namedStyleType': 'HEADING_2'},
            'fields': 'namedStyleType'
        }
    }
]

batch_update(doc_id, requests, access_token)
print("Document updated successfully!")
```

## 关键概念

### 文档索引
- 索引从 1 开始（文档从索引 1 开始）
- 每个字符（包括换行符）占据一个索引
- 要在末尾追加，使用 `endOfSegmentLocation`

### 批量更新
- 多个请求原子性应用
- 请求按顺序处理
- 先获取文档以确定正确的索引

### 字段掩码
- 更新样式时，指定要更新的字段
- 使用逗号分隔的字段名：`'fields': 'bold,fontSize,foregroundColor'`

## 错误处理

| 状态码 | 含义 |
|-------------|---------|
| 400 | Bad Request - 请求格式无效 |
| 401 | Unauthorized - access token 无效或已过期 |
| 403 | Forbidden - 权限不足 |
| 404 | Not Found - 文档不存在 |
| 429 | Rate Limited - 请求过多 |

### Token 刷新
Access token 在 1 小时后过期。如果收到 401 错误，请刷新 token：

```python
try:
    # Make API call
    req = urllib.request.Request(url)
    req.add_header('Authorization', f'Bearer {access_token}')
    response = urllib.request.urlopen(req)
except urllib.error.HTTPError as e:
    if e.code == 401:
        # Refresh token and retry
        access_token = get_access_token()
        req = urllib.request.Request(url)
        req.add_header('Authorization', f'Bearer {access_token}')
        response = urllib.request.urlopen(req)
```

## 最佳实践

1. **Token 管理**
   - 缓存 access token（有效期 1 小时）
   - 安全存储 refresh token
   - 实现自动 token 刷新

2. **批量操作**
   - 将多个更新合并为单个批量调用
   - 减少 API 调用并提高性能

3. **索引计算**
   - 更新前始终获取当前文档状态
   - 考虑换行符 (\n)
   - 使用 `endOfSegmentLocation` 进行追加

4. **速率限制**
   - Google Docs API 有配额（请查看 Cloud Console）
   - 对速率限制错误实现指数退避

## 资源

- [Google Docs API Documentation](https://developers.google.com/docs/api)
- [OAuth 2.0 for Desktop Apps](https://developers.google.com/identity/protocols/oauth2/native-app)
- [Document Structure](https://developers.google.com/docs/api/concepts/structure)
- [Request Types Reference](https://developers.google.com/docs/api/reference/rest/v1/documents/request)
- [Python Quickstart](https://developers.google.com/docs/api/quickstart/python)
