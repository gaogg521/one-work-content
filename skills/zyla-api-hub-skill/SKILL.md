---
name: zyla-api-hub-skill
description: Zyla API Hub Skill —— 将你的 OpenClaw AI 智能体转变为现实世界操作员。通过 Zyla API Hub 为其提供 10,000+ 生产就绪 API —— 即时访问天气、金融、翻译、邮箱验证、地理定位等。
metadata: None
openclaw: None
homepage: https://zylalabs.com/openclaw/connect
primaryEnv: ZYLA_API_KEY
requires: {}
tags:
- AI
- API
---

# Zyla API Hub Skill

将你的 OpenClaw AI 智能体转变为现实世界操作员。
通过 Zyla API Hub 为其提供 10,000+ 生产就绪 API —— 即时访问天气、金融、翻译、邮箱验证、地理定位等，全部通过一个统一的 API 密钥、按量付费定价和零供应商锁定。

## 设置

如果未配置 `ZYLA_API_KEY`，引导用户：

1. 访问 https://zylalabs.com/openclaw/connect 获取 API 密钥
2. 或如果插件已安装，运行 `/zyla connect`（自动打开浏览器）
3. 将密钥添加到 `~/.openclaw/openclaw.json` 的 `skills.entries.zyla-api-hub-skill.apiKey` 下

切勿要求用户在聊天中粘贴他们的 API 密钥。请他们通过配置设置，并在准备好后确认。

## 快速开始 —— 热门 API

直接使用这些，无需搜索目录。每个都包含 API ID、端点详情和参数。

<!-- POPULAR_APIS_START -->
<!-- 此部分由以下命令自动生成：npx tsx scripts/generate-popular.ts -->
<!-- 发布前运行以使用最新的前 20 个热门 API 更新 -->

### Weather by Zip API (ID: 781)
- **Use when**: user asks about weather, temperature, forecast, climate, conditions by zip code
- **Category**: Weather & Environment
- **Call**: `npx tsx {baseDir}/scripts/zyla-api.ts call --api 781 --endpoint <endpoint_id> --params '{"zip":"10001"}'`

### Currency Conversion API (example)
- **Use when**: user asks about currency exchange, conversion rates, forex
- **Category**: Finance
- **Call**: `npx tsx {baseDir}/scripts/zyla-api.ts call --api <id> --endpoint <endpoint_id> --params '{"from":"USD","to":"EUR","amount":"100"}'`

### Email Validation API (example)
- **Use when**: user asks to validate, verify, or check an email address
- **Category**: Data Validation
- **Call**: `npx tsx {baseDir}/scripts/zyla-api.ts call --api <id> --endpoint <endpoint_id> --params '{"email":"test@example.com"}'`

> **Note**: Run `npx tsx {baseDir}/scripts/generate-popular.ts` to regenerate this section with real API IDs and endpoints from the live catalog.

<!-- POPULAR_APIS_END -->

## 发现 API

对于上面未列出的 API，搜索目录：

```bash
# 按关键词搜索
npx tsx {baseDir}/scripts/zyla-catalog.ts search "recipe"

# 按分类列出 API
npx tsx {baseDir}/scripts/zyla-catalog.ts list --category "Finance"

# 获取特定 API 的端点
npx tsx {baseDir}/scripts/zyla-catalog.ts endpoints --api 781
```

## 调用 API

### 使用辅助脚本（推荐）

```bash
# 基本调用
npx tsx {baseDir}/scripts/zyla-api.ts call \
  --api <api_id> \
  --endpoint <endpoint_id> \
  --params '{"key":"value"}'

# 指定 HTTP 方法（默认：GET）
npx tsx {baseDir}/scripts/zyla-api.ts call \
  --api <api_id> \
  --endpoint <endpoint_id> \
  --method POST \
  --params '{"key":"value"}'

# 获取 API 信息
npx tsx {baseDir}/scripts/zyla-api.ts info --api <api_id>

# 检查健康和剩余配额
npx tsx {baseDir}/scripts/zyla-api.ts health
```

### 使用 curl（降级方案）

```bash
curl -H "Authorization: Bearer $ZYLA_API_KEY" \
  "https://zylalabs.com/api/{api_id}/{api_slug}/{endpoint_id}/{endpoint_slug}?param=value"
```

**URL 模式**: `https://zylalabs.com/api/{api_id}/{api_name_slug}/{endpoint_id}/{endpoint_name_slug}`

- `api_id` 和 `endpoint_id` 是数字 ID（这些负责实际路由）
- `api_name_slug` 和 `endpoint_name_slug` 是 URL 友好名称（用于可读性）

## 错误处理

- **401 Unauthorized**: API 密钥无效或已过期。要求用户运行 `/zyla connect` 或访问 https://zylalabs.com/openclaw/connect 获取新密钥。
- **403 Forbidden**: 订阅问题。按量付费套餐应自动处理此问题。如果持续存在，要求用户联系支持。
- **429 Too Many Requests**: 超出速率限制。检查响应头 `X-Zyla-RateLimit-Minute-Remaining`。重试前等待。
- **404 Not Found**: API 或端点不存在。使用目录验证 ID。
- **5xx Server Error**: 上游 API 问题。短暂延迟后重试（2-5 秒）。

## 速率限制头

每个 API 响应都包含这些头：
- `X-Zyla-RateLimit-Minute-Limit`: 每分钟最大请求数
- `X-Zyla-RateLimit-Minute-Remaining`: 本分钟剩余请求数
- `X-Zyla-API-Calls-Monthly-Used`: 本计费周期总调用数
- `X-Zyla-API-Calls-Monthly-Remaining`: 本周期剩余调用数

## 计费

- **按量付费**: 无月费。每个 API 调用按该 API 的每次调用费率计费。
- 计费在每个周期结束时通过 Stripe 进行。
- 使用健康端点检查当前使用量：`npx tsx {baseDir}/scripts/zyla-api.ts health`
