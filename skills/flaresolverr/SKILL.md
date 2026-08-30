---
name: flaresolverr
description: 绕过 Cloudflare 保护 —— 当 curl/summarize 得到 403 或 Cloudflare 拦截时使用
metadata: None
openclaw: None
emoji: 🛡️
primaryEnv: FLARESOLVERR_URL
requires: None
bins:
- curl
- jq
env:
- FLARESOLVERR_URL
tags:
- 云服务
---

# FlareSolverr — Cloudflare 绕过

当 direct curl requests 因 403 或 Cloudflare challenge pages 失败时，使用 FlareSolverr 绕过 Cloudflare 保护。

## 设置

1. **运行 FlareSolverr**（推荐 Docker）：

```bash
docker run -d --name flaresolverr -p 8191:8191 ghcr.io/flaresolverr/flaresolverr:latest
```

2. **设置环境变量：**

```bash
export FLARESOLVERR_URL="http://localhost:8191"
```

3. **验证：**

```bash
curl -s "$FLARESOLVERR_URL/health" | jq '.'
# 预期输出: {"status":"ok","version":"3.x.x"}
```

## 何时使用

- **Direct curl fails** 并返回 403 Forbidden
- **Cloudflare challenge page** 出现（JS challenge、captcha、"Checking your browser"）
- **Bot detection** 阻止自动化请求
- **Rate limiting** 或 anti-scraping 措施

## 工作流

1. **先尝试 direct curl**（更快更简单）
2. **如果被拦截**：使用 FlareSolverr 获取 cookies/user-agent
3. **Reuse session** 用于后续请求（可选，为了性能）

## 基本用法

### Simple GET Request

```bash
curl -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "request.get",
    "url": "https://example.com/protected-page",
    "maxTimeout": 60000
  }' | jq '.'
```

### Response Structure

```json
{
  "status": "ok",
  "message": "Challenge solved!",
  "solution": {
    "url": "https://example.com/protected-page",
    "status": 200,
    "headers": {},
    "response": "<html>...</html>",
    "cookies": [
      {
        "name": "cf_clearance",
        "value": "...",
        "domain": ".example.com"
      }
    ],
    "userAgent": "Mozilla/5.0 ..."
  },
  "startTimestamp": 1234567890,
  "endTimestamp": 1234567895,
  "version": "3.3.2"
}
```

### Extract Page Content

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "request.get",
    "url": "https://example.com/protected-page"
  }' | jq -r '.solution.response'
```

### Extract Cookies

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "request.get",
    "url": "https://example.com"
  }' | jq -r '.solution.cookies[] | "\(.name)=\(.value)"'
```

## Session Management

Sessions 允许为多个请求复用 browser context（cookies、user-agent），提高性能。

### Create Session

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{"cmd": "sessions.create"}' | jq -r '.session'
```

### Use Session for Request

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "request.get",
    "url": "https://example.com/page1",
    "session": "SESSION_ID"
  }' | jq -r '.solution.response'
```

### List Active Sessions

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{"cmd": "sessions.list"}' | jq '.sessions'
```

### Destroy Session

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "sessions.destroy",
    "session": "SESSION_ID"
  }'
```

## POST Requests

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "request.post",
    "url": "https://example.com/api/endpoint",
    "postData": "key1=value1&key2=value2",
    "maxTimeout": 60000
  }' | jq '.'
```

对于 JSON POST data：

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "request.post",
    "url": "https://example.com/api/endpoint",
    "postData": "{\"key\":\"value\"}",
    "headers": {
      "Content-Type": "application/json"
    }
  }' | jq '.'
```

## 高级选项

### Custom User-Agent

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "request.get",
    "url": "https://example.com",
    "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
  }' | jq '.'
```

### Custom Headers

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "request.get",
    "url": "https://example.com",
    "headers": {
      "Accept-Language": "en-US,en;q=0.9",
      "Referer": "https://google.com"
    }
  }' | jq '.'
```

### Proxy Support

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "request.get",
    "url": "https://example.com",
    "proxy": {
      "url": "http://proxy.example.com:8080"
    }
  }' | jq '.'
```

### Download Binary Content

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "request.get",
    "url": "https://example.com/file.pdf",
    "download": true
  }' | jq -r '.solution.response' | base64 -d > file.pdf
```

## 错误处理

### 常见错误

- **`"status": "error"`**: Request failed（检查 `message` 字段）
- **`"status": "timeout"`**: maxTimeout 超出（增加 timeout）
- **`"status": "captcha"`**: 需要手动 captcha（罕见，通常自动解决）

### Check Status

```bash
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{"cmd": "request.get", "url": "https://example.com"}' | \
  jq -r '.status'
```

## 示例工作流

### 绕过 Cloudflare 并提取数据

```bash
# Step 1: 通过 FlareSolverr 获取页面
RESPONSE=$(curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "request.get",
    "url": "https://example.com/protected-page"
  }')

# Step 2: 检查是否成功
STATUS=$(echo "$RESPONSE" | jq -r '.status')
if [ "$STATUS" != "ok" ]; then
  echo "Failed: $(echo "$RESPONSE" | jq -r '.message')"
  exit 1
fi

# Step 3: 提取并解析 HTML
echo "$RESPONSE" | jq -r '.solution.response'
```

### Multi-Page Session

```bash
# Create session
SESSION=$(curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d '{"cmd": "sessions.create"}' | jq -r '.session')

# Page 1
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d "{\"cmd\": \"request.get\", \"url\": \"https://example.com/page1\", \"session\": \"$SESSION\"}" | \
  jq -r '.solution.response'

# Page 2（reuses cookies from page 1）
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d "{\"cmd\": \"request.get\", \"url\": \"https://example.com/page2\", \"session\": \"$SESSION\"}" | \
  jq -r '.solution.response'

# Cleanup
curl -s -X POST "$FLARESOLVERR_URL/v1" \
  -H "Content-Type: application/json" \
  -d "{\"cmd\": \"sessions.destroy\", \"session\": \"$SESSION\"}"
```

## Health Check

```bash
curl -s "$FLARESOLVERR_URL/health" | jq '.'
```

## 性能提示

1. **对同一 domain 的多个请求使用 sessions**（reuses cookies/context）
2. **为慢速站点增加 maxTimeout**（默认：60000ms）
3. **尽可能回退到 direct curl**（FlareSolverr 因 browser overhead 更慢）
4. **完成后销毁 sessions** 以释放资源

## 限制

- **比 direct curl 慢**（启动 headless browser）
- **资源密集**（限制并发请求）
- **可能无法解决所有 captchas**（大多数 Cloudflare challenges 有效）
- **响应中仅 HTML**（fetch 后无 client-side JS 执行）

## 最佳实践

1. **始终先尝试 direct curl**
2. **对 multi-page 工作流使用 sessions**
3. **设置合适的 maxTimeout**（默认 60s，为慢速站点增加）
4. **完成后清理 sessions**
5. **优雅地处理错误**（检查 `status` 字段）
6. **Rate limit** 你的请求（不要压垮 FlareSolverr 或目标站点）
