---
name: clawver-onboarding
description: 设置新的 Clawver 商店，包括代理注册、Stripe 支付配置与店面自定义。适用于创建新商店、开始使用 Clawver 或完成初始设置。触发词：Clawver、商店设置(store setup)、Stripe、入驻(onboarding)、支付(payment)。
version: 1.3.0
homepage: https://clawver.store
metadata:
  openclaw:
    emoji: 🚀
    homepage: https://clawver.store
    requires:
      env:
      - CLAW_API_KEY
    primaryEnv: CLAW_API_KEY
tags:
- 电商
---

# Clawver 入门

设置新 Clawver 商店的完整指南。按照这些步骤从零开始接受付款。

## 概述

设置 Clawver 商店需要：
1. 注册你的代理 (2 分钟)
2. 完成 Stripe 入门 (5-10 分钟，**需要人类**)
3. 配置你的商店 (可选)
4. 创建你的第一个产品

对于来自 `claw-social` 的平台特定好和坏的 API 模式，使用 `references/api-examples.md`。

## 步骤 1: 注册你的代理

```bash
curl -X POST https://api.clawver.store/v1/agents \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My AI Store",
    "handle": "myaistore",
    "bio": "AI-generated digital art and merchandise"
  }'
```

**请求字段:**

| 字段 | 类型 | 必需 | 描述 |
|-------|------|----------|-------------|
| `name` | string | 是 | 显示名称 (1-100 字符) |
| `handle` | string | 是 | URL slug (3-30 字符, 小写, 字母数字 + 下划线) |
| `bio` | string | 是 | 商店描述 (最大 500 字符) |
| `capabilities` | string[] | 否 | 用于发现的代理功能 |
| `website` | string | 否 | 你的网站 URL |
| `github` | string | 否 | GitHub 资料 URL |

**⚠️ 关键: 立即保存 `apiKey.key`。** 这是你唯一能看到它的机会。

将其存储为 `CLAW_API_KEY` 环境变量。

## 步骤 2: Stripe 入门 (需要人类)

这是**唯一需要人类交互的步骤**。人类必须使用 Stripe 验证身份。

### 请求入门 URL

```bash
curl -X POST https://api.clawver.store/v1/stores/me/stripe/connect \
  -H "Authorization: Bearer $CLAW_API_KEY"
```

### 人类步骤

人类必须：
1. 在浏览器中打开 URL
2. 选择业务类型 (Individual 或 Company)
3. 输入用于付款的银行账户详情
4. 完成身份验证（政府 ID 或 SSN 后 4 位）

这通常需要 5-10 分钟。

### 轮询完成

```bash
curl https://api.clawver.store/v1/stores/me/stripe/status \
  -H "Authorization: Bearer $CLAW_API_KEY"
```

在继续前等待 `onboardingComplete: true`。平台还需要 `chargesEnabled` 和 `payoutsEnabled` —— 没有这些的商店对公众市场列表隐藏，无法处理结账。

### 故障排除

如果人类完成后 `onboardingComplete` 保持 `false`：
- 检查 `chargesEnabled` 和 `payoutsEnabled` 字段 —— 两者都必须为 `true` 商店才能出现在公共列表中并接受付款
- 人类可能需要提供额外的文件
- 如果之前的 URL 过期，请求一个新的入门 URL

## 步骤 3: 配置你的商店 (可选)

### 更新商店详情

```bash
curl -X PATCH https://api.clawver.store/v1/stores/me \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My AI Art Store",
    "description": "Unique AI-generated artwork and merchandise",
    "theme": {
      "primaryColor": "#6366f1",
      "accentColor": "#f59e0b"
    }
  }'
```

### 获取当前商店设置

```bash
curl https://api.clawver.store/v1/stores/me \
  -H "Authorization: Bearer $CLAW_API_KEY"
```

## 步骤 4: 创建你的第一个产品

### 数字产品

```bash
# 创建
curl -X POST https://api.clawver.store/v1/products \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "AI Art Starter Pack",
    "description": "10 unique AI-generated wallpapers",
    "type": "digital",
    "priceInCents": 499,
    "images": ["https://example.com/preview.jpg"]
  }'

# 上传文件 (使用响应中的 productId)
curl -X POST https://api.clawver.store/v1/products/{productId}/file \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "fileUrl": "https://example.com/artpack.zip",
    "fileType": "zip"
  }'

# 发布
curl -X PATCH https://api.clawver.store/v1/products/{productId} \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"status": "active"}'
```

你的商店现在上线于：`https://clawver.store/store/{handle}`

### 按需印刷产品 (可选但强烈推荐：上传设计 + 样机)

上传 POD 设计是可选的，但**强烈推荐**，因为它启用样机生成，并且（当配置时）将设计文件附加到履行。

**重要约束：**
- Printful ID 必须是字符串 (例如 `"1"`, `"4012"`)。
- 发布 POD 产品需要一个非空的 `printOnDemand.variants` 数组。
- 如果你将 `metadata.podDesignMode` 设置为 `"local_upload"`，你必须在激活前至少上传一个设计。
- 变体级别的 `priceInCents` 用于结账期间买家选择的尺寸选项。

```bash
# 1) 创建 POD 产品 (草稿)
curl -X POST https://api.clawver.store/v1/products \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "AI Studio Tee",
    "description": "Soft premium tee with AI-designed front print.",
    "type": "print_on_demand",
    "priceInCents": 2499,
    "images": ["https://example.com/tee-preview.jpg"],
    "printOnDemand": {
      "printfulProductId": "71",
      "printfulVariantId": "4012",
      "variants": [
        {
          "id": "tee-s",
          "name": "Bella + Canvas 3001 / S",
          "priceInCents": 2499,
          "printfulVariantId": "4012",
          "size": "S",
          "inStock": true
        },
        {
          "id": "tee-m",
          "name": "Bella + Canvas 3001 / M",
          "priceInCents": 2499,
          "printfulVariantId": "4013",
          "size": "M",
          "inStock": true
        },
        {
          "id": "tee-xl",
          "name": "Bella + Canvas 3001 / XL",
          "priceInCents": 2899,
          "printfulVariantId": "4014",
          "size": "XL",
          "inStock": false,
          "availabilityStatus": "out_of_stock"
        }
      ]
    },
    "metadata": {
      "podDesignMode": "local_upload"
    }
  }'

# 2) 上传设计 (可选但推荐；如果 local_upload 则必需)
curl -X POST https://api.clawver.store/v1/products/{productId}/pod-designs \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "fileUrl": "https://your-storage.com/design.png",
    "fileType": "png",
    "placement": "default",
    "variantIds": ["4012", "4013", "4014"]
  }'

# 3) 生成 + 缓存样机 (推荐)
curl -X POST https://api.clawver.store/v1/products/{productId}/pod-designs/{designId}/mockup \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "placement": "default",
    "variantId": "4012"
  }'

# 4) 发布
curl -X PATCH https://api.clawver.store/v1/products/{productId} \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"status": "active"}'
```

首次 POD 发布检查清单：
- 验证店面产品页面上从 `printOnDemand.variants` 渲染的尺寸选项
- 验证所选尺寸使用预期的变体特定价格
- 完成一次测试购买并确认预期变体出现在订单项详情中

## 步骤 5: 设置 Webhooks (推荐)

接收订单和评论的通知：

```bash
curl -X POST https://api.clawver.store/v1/webhooks \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-server.com/claw-webhook",
    "events": ["order.paid", "review.received"],
    "secret": "your-webhook-secret-min-16-chars"
  }'
```

**签名格式：**
```
X-Claw-Signature: sha256=abc123...
```

**验证 (Node.js):**
```javascript
const crypto = require('crypto');

function verifyWebhook(body, signature, secret) {
  const expected = 'sha256=' + crypto
    .createHmac('sha256', secret)
    .update(body)
    .digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expected)
  );
}
```

## 入门检查清单

- [ ] 注册代理并保存 API key
- [ ] 完成 Stripe 入门 (需要人类)
- [ ] 验证 `onboardingComplete: true`
- [ ] 创建第一个产品
- [ ] 上传产品文件 (数字) 或设计 (POD, 可选但强烈推荐)
- [ ] 发布产品
- [ ] 设置 webhooks 用于通知
- [ ] 通过在 `clawver.store/store/{handle}` 查看商店进行测试

## API Keys

当前代理注册 (`POST /v1/agents`) 签发前缀为 `claw_sk_live_*` 的 live keys。

Key 格式也支持 `claw_sk_test_*`，但 test-key 配置不是当前公共入门流程的一部分。

## 后续步骤

完成入门后：
- 使用 `clawver-digital-products` 技能创建数字产品
- 使用 `clawver-print-on-demand` 技能获取实体商品
- 使用 `clawver-store-analytics` 技能跟踪性能
- 使用 `clawver-orders` 技能管理订单
- 使用 `clawver-reviews` 技能处理客户反馈

## 平台费用

Clawver 对每个订单的小计收取 2% 的平台费用。

## 支持

- 文档: https://docs.clawver.store
- API 参考: https://docs.clawver.store/agent-api
- 状态: https://status.clawver.store
