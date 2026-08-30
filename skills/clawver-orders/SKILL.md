---
name: clawver-orders
description: 管理 Clawver 商店订单，支持订单列表、状态跟踪、退款处理与下载链接生成。适用于客户订单查询、履约(fulfillment)、退款或订单历史查看。触发词：Clawver、订单(orders)、退款(refund)、履约(fulfillment)、下载链接(download link)。
version: 1.3.0
homepage: https://clawver.store
metadata:
  openclaw:
    emoji: 📦
    homepage: https://clawver.store
    requires:
      env:
      - CLAW_API_KEY
    primaryEnv: CLAW_API_KEY
tags:
- 电商
- 管理
---

# Clawver Orders

管理你的 Clawver 商店订单——查看订单历史、跟踪履约、处理退款并生成下载链接。

## 前置条件

- `CLAW_API_KEY` 环境变量
- 有活跃订单的商店

如需来自 `claw-social` 的平台特定 API 良好和不良模式，请使用 `references/api-examples.md`。

## 列出订单

### 获取所有订单

```bash
curl https://api.clawver.store/v1/orders \
  -H "Authorization: Bearer $CLAW_API_KEY"
```

### 按状态筛选

```bash
# 已确认（已支付）订单
curl "https://api.clawver.store/v1/orders?status=confirmed" \
  -H "Authorization: Bearer $CLAW_API_KEY"

# 进行中的 POD 订单
curl "https://api.clawver.store/v1/orders?status=processing" \
  -H "Authorization: Bearer $CLAW_API_KEY"

# 已发货订单
curl "https://api.clawver.store/v1/orders?status=shipped" \
  -H "Authorization: Bearer $CLAW_API_KEY"

# 已送达订单
curl "https://api.clawver.store/v1/orders?status=delivered" \
  -H "Authorization: Bearer $CLAW_API_KEY"
```

**订单状态：**

| 状态 | 描述 |
|--------|-------------|
| `pending` | 订单已创建，等待支付 |
| `confirmed` | 支付已确认 |
| `processing` | 正在履约 |
| `shipped` | 运输中（仅限 POD） |
| `delivered` | 已完成 |
| `cancelled` | 已取消 |

`paymentStatus` 单独报告，可以是 `pending`、`paid`、`failed`、`partially_refunded` 或 `refunded`。

### 分页

```bash
curl "https://api.clawver.store/v1/orders?limit=20" \
  -H "Authorization: Bearer $CLAW_API_KEY"
```

支持 `limit`。此端点当前不暴露基于游标的分页。

## 获取订单详情

```bash
curl https://api.clawver.store/v1/orders/{orderId} \
  -H "Authorization: Bearer $CLAW_API_KEY"
```

对于按需打印商品，订单载荷包含：
- `variantId`（必需——履约变体标识符，必须匹配商品变体）
- `variantName`（人类可读的选择尺寸/变体标签）

注意：自 2026 年 2 月起，`variantId` 是所有 POD 结账商品的必填项。缺货变体将被拒绝。

## 生成下载链接

### 所有者下载链接（数字商品）

```bash
curl "https://api.clawver.store/v1/orders/{orderId}/download/{itemId}" \
  -H "Authorization: Bearer $CLAW_API_KEY"
```

当客户报告下载问题或请求新链接时使用此功能。

### 客户下载链接（数字商品）

```bash
curl "https://api.clawver.store/v1/orders/{orderId}/download/{itemId}/public?token={downloadToken}"
```

下载令牌按订单商品发放，可在结账收据中返回（`GET /v1/checkout/{checkoutId}/receipt`）。

### 客户订单状态（公开）

```bash
curl "https://api.clawver.store/v1/orders/{orderId}/public?token={orderStatusToken}"
```

### 结账收据（成功页面/支持）

```bash
curl "https://api.clawver.store/v1/checkout/{checkoutId}/receipt"
```

## 处理退款

### 全额退款

```bash
curl -X POST https://api.clawver.store/v1/orders/{orderId}/refund \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "amountInCents": 2499,
    "reason": "Customer requested refund"
  }'
```

### 部分退款

```bash
curl -X POST https://api.clawver.store/v1/orders/{orderId}/refund \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "amountInCents": 500,
    "reason": "Partial refund for missing item"
  }'
```

**注意：**
- `amountInCents` 是必需的，必须是正整数
- `reason` 是必需的
- `amountInCents` 不能超过剩余可退款金额
- 退款通过 Stripe 处理（1-5 个工作日到达客户）
- 订单必须具有 `paid` 或 `partially_refunded` 的 `paymentStatus`

## POD 订单跟踪

对于按需打印订单，发货后跟踪信息可用：

```bash
curl https://api.clawver.store/v1/orders/{orderId} \
  -H "Authorization: Bearer $CLAW_API_KEY"
```

检查响应中的 `trackingUrl`、`trackingNumber` 和 `carrier` 字段。

### 发货更新 Webhook

```bash
curl -X POST https://api.clawver.store/v1/webhooks \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-server.com/webhook",
    "events": ["order.shipped", "order.fulfilled"],
    "secret": "your-secret-min-16-chars"
  }'
```

## 订单 Webhooks

接收实时通知：

```bash
curl -X POST https://api.clawver.store/v1/webhooks \
  -H "Authorization: Bearer $CLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-server.com/webhook",
    "events": ["order.created", "order.paid", "order.refunded"],
    "secret": "your-webhook-secret-16chars"
  }'
```

**签名格式：**
```
X-Claw-Signature: sha256=abc123...
```

**验证（Node.js）：**
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

## 常见工作流

### 每日订单检查

```python
# 获取新支付/已确认订单
response = api.get("/v1/orders?status=confirmed")
orders = response["data"]["orders"]
print(f"New orders: {len(orders)}")

for order in orders:
    print(f"  - {order['id']}: ${order['totalInCents']/100:.2f}")
```

### 处理退款请求

```python
def process_refund(order_id, amount_cents, reason):
    # 获取订单详情
    response = api.get(f"/v1/orders/{order_id}")
    order = response["data"]["order"]
    
    # 检查是否可退款
    if order["paymentStatus"] not in ["paid", "partially_refunded"]:
        return "Order cannot be refunded"
    
    # 处理退款
    result = api.post(f"/v1/orders/{order_id}/refund", {
        "amountInCents": amount_cents,
        "reason": reason
    })
    
    return f"Refunded ${amount_cents/100:.2f}"
```

### 尺寸错误支持手册

```python
def handle_wrong_size(order_id):
    response = api.get(f"/v1/orders/{order_id}")
    order = response["data"]["order"]

    for item in order["items"]:
        if item.get("productType") == "print_on_demand":
            print("Variant ID:", item.get("variantId"))
            print("Variant Name:", item.get("variantName"))

    # 在发起退款/更换工作流之前确认所选变体。
```

### 重新发送下载链接

```python
def resend_download(order_id, item_id):
    # 生成新的下载链接
    response = api.get(f"/v1/orders/{order_id}/download/{item_id}")
    
    return response["data"]["downloadUrl"]
```

## 订单生命周期

```
pending → confirmed → processing → shipped → delivered
               ↓
      cancelled / refunded (paymentStatus)
```

**数字产品：** `confirmed` → `delivered`（即时履约）
**POD 产品：** `confirmed` → `processing` → `shipped` → `delivered`
