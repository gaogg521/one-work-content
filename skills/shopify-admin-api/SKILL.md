---
name: shopify-admin-api
description: 对 Shopify Admin REST API 的完整读/写访问，用于管理订单、产品、客户、库存、履行、退款、退货和交易
slug: shopify-admin-api
display_name: Shopify Admin API
version: 1.0.0
tags:
- latest
---

# Shopify Admin API

## 描述

对 Shopify Admin REST API 的完整读/写访问，用于管理订单、产品、客户、库存、履行、退款、退货和交易。

## 设置

### 环境变量

- `SHOPIFY_STORE_DOMAIN` - 你的商店的 myshopify.com 域名（例如，`my-store.myshopify.com`）
- `SHOPIFY_ACCESS_TOKEN` - 来自自定义应用的 Admin API 访问令牌

### 所需的 API 范围

| 范围 | 访问权限 |
|-------|--------|
| `read_orders` / `write_orders` | 订单、履行、废弃结账 |
| `read_products` / `write_products` | 产品、变体、系列 |
| `read_customers` / `write_customers` | 客户、细分 |
| `read_inventory` / `write_inventory` | 库存水平、项目 |
| `read_returns` / `write_returns` | 退货 |
| `read_all_orders` | 超过 60 天的订单（需要批准） |

### 认证

所有请求都需要此 header：

```
X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN
```

### 获取访问令牌

1. 前往你的 Shopify 管理员 > 设置 > 应用和销售渠道
2. 点击 "开发应用" > "创建应用"
3. 根据你需要的内容配置 Admin API 范围
4. 将应用安装到你的商店
5. 复制 Admin API 访问令牌

## API 参考

基础 URL: `https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10`

### 订单

#### 列出订单

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

查询参数：`ids`、`limit`、`since_id`、`created_at_min`、`created_at_max`、`updated_at_min`、`updated_at_max`、`processed_at_min`、`processed_at_max`、`status`（open、closed、cancelled、any）、`financial_status`、`fulfillment_status`、`fields`

#### 获取订单

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取订单数量

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/count.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 更新订单

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{"order":{"id":{ORDER_ID},"note":"Updated note","tags":"priority,vip"}}'
```

#### 关闭订单

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/close.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -X POST
```

#### 重新打开订单

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/open.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -X POST
```

#### 取消订单

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/cancel.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"reason":"customer","email":true,"restock":true}'
```

取消原因：`customer`、`fraud`、`inventory`、`declined`、`other`

### 产品

#### 列出产品

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/products.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

查询参数：`ids`、`limit`、`since_id`、`title`、`vendor`、`handle`、`product_type`、`collection_id`、`created_at_min`、`created_at_max`、`updated_at_min`、`updated_at_max`、`published_at_min`、`published_at_max`、`published_status`（published、unpublished、any）、`fields`、`status`（active、archived、draft）

#### 获取产品

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/products/{PRODUCT_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取产品数量

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/products/count.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 创建产品

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/products.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"product":{"title":"Burton Custom Freestyle","body_html":"<strong>Good snowboard!</strong>","vendor":"Burton","product_type":"Snowboard","status":"draft","variants":[{"price":"99.99","sku":"BOARD-001"}]}}'
```

#### 更新产品

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/products/{PRODUCT_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{"product":{"id":{PRODUCT_ID},"title":"Updated Product Title","status":"active"}}'
```

#### 删除产品

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/products/{PRODUCT_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -X DELETE
```

### 产品变体

#### 列出产品的变体

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/products/{PRODUCT_ID}/variants.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取变体

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/variants/{VARIANT_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 创建变体

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/products/{PRODUCT_ID}/variants.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"variant":{"option1":"Blue","price":"19.99","sku":"BLUE-001","inventory_management":"shopify"}}'
```

#### 更新变体

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/variants/{VARIANT_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{"variant":{"id":{VARIANT_ID},"price":"24.99","compare_at_price":"29.99"}}'
```

#### 删除变体

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/products/{PRODUCT_ID}/variants/{VARIANT_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -X DELETE
```

### 客户

#### 列出客户

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/customers.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

查询参数：`ids`、`limit`、`since_id`、`created_at_min`、`created_at_max`、`updated_at_min`、`updated_at_max`、`fields`

#### 搜索客户

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/customers/search.json?query=email:customer@example.com" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

搜索字段：`email`、`phone`、`first_name`、`last_name`、`company`、`orders_count`、`total_spent`、`country`、`state`

#### 获取客户

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/customers/{CUSTOMER_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取客户数量

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/customers/count.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 创建客户

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/customers.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"customer":{"first_name":"John","last_name":"Doe","email":"john.doe@example.com","phone":"+15551234567","addresses":[{"address1":"123 Main St","city":"Ottawa","province":"ON","country":"CA","zip":"K1A 0B1"}]}}'
```

#### 更新客户

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/customers/{CUSTOMER_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{"customer":{"id":{CUSTOMER_ID},"tags":"vip,wholesale","note":"Important customer"}}'
```

#### 删除客户

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/customers/{CUSTOMER_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -X DELETE
```

#### 获取客户订单

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/customers/{CUSTOMER_ID}/orders.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

### 库存

#### 列出库存水平

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/inventory_levels.json?inventory_item_ids={ITEM_ID}" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

查询参数：`inventory_item_ids`（必需）、`location_ids`、`limit`、`updated_at_min`

#### 调整库存水平

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/inventory_levels/adjust.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"location_id":{LOCATION_ID},"inventory_item_id":{ITEM_ID},"available_adjustment":5}'
```

#### 设置库存水平

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/inventory_levels/set.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"location_id":{LOCATION_ID},"inventory_item_id":{ITEM_ID},"available":100}'
```

#### 将库存连接到位置

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/inventory_levels/connect.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"location_id":{LOCATION_ID},"inventory_item_id":{ITEM_ID}}'
```

### 库存项目

#### 列出库存项目

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/inventory_items.json?ids={ITEM_IDS}" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取库存项目

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/inventory_items/{ITEM_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 更新库存项目

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/inventory_items/{ITEM_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{"inventory_item":{"id":{ITEM_ID},"cost":"25.00","tracked":true}}'
```

### 位置

#### 列出位置

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/locations.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取位置

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/locations/{LOCATION_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取位置数量

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/locations/count.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取位置的库存水平

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/locations/{LOCATION_ID}/inventory_levels.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

### 履行

#### 列出订单的履行

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/fulfillments.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取履行

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/fulfillments/{FULFILLMENT_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取履行数量

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/fulfillments/count.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 创建履行

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/fulfillments.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"fulfillment":{"line_items_by_fulfillment_order":[{"fulfillment_order_id":{FULFILLMENT_ORDER_ID}}],"tracking_info":{"number":"1Z999AA10123456784","url":"https://www.ups.com/track?tracknum=1Z999AA10123456784","company":"UPS"}}}'
```

#### 更新跟踪

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/fulfillments/{FULFILLMENT_ID}/update_tracking.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"fulfillment":{"tracking_info":{"number":"1Z999AA10123456784","url":"https://www.ups.com/track?tracknum=1Z999AA10123456784","company":"UPS"},"notify_customer":true}}'
```

#### 取消履行

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/fulfillments/{FULFILLMENT_ID}/cancel.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -X POST
```

### 履行订单

#### 列出订单的履行订单

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/fulfillment_orders.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取履行订单

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/fulfillment_orders/{FULFILLMENT_ORDER_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

### 退款

#### 列出订单的退款

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/refunds.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取退款

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/refunds/{REFUND_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 计算退款

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/refunds/calculate.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"refund":{"shipping":{"full_refund":true},"refund_line_items":[{"line_item_id":{LINE_ITEM_ID},"quantity":1,"restock_type":"return"}]}}'
```

补货类型：`no_restock`、`cancel`、`return`、`legacy_restock`

#### 创建退款

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/refunds.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"refund":{"currency":"USD","notify":true,"note":"Customer returned item","shipping":{"full_refund":true},"refund_line_items":[{"line_item_id":{LINE_ITEM_ID},"quantity":1,"restock_type":"return"}],"transactions":[{"parent_id":{TRANSACTION_ID},"amount":"10.00","kind":"refund","gateway":"shopify_payments"}]}}'
```

### 退货

#### 列出退货

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/returns.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

查询参数：`limit`、`status`（open、closed、cancelled、requested、in_progress）

#### 获取退货

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/returns/{RETURN_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 创建退货

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/returns.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"return":{"order_id":{ORDER_ID},"return_line_items":[{"fulfillment_line_item_id":{FULFILLMENT_LINE_ITEM_ID},"quantity":1,"return_reason":"WRONG_ITEM"}]}}'
```

退货原因：`UNKNOWN`、`SIZE_TOO_SMALL`、`SIZE_TOO_LARGE`、`UNWANTED`、`NOT_AS_DESCRIBED`、`WRONG_ITEM`、`DEFECTIVE`、`STYLE`、`COLOR`、`OTHER`

### 交易

#### 列出订单的交易

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/transactions.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取交易

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/transactions/{TRANSACTION_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取交易数量

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/transactions/count.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 创建交易（捕获）

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/orders/{ORDER_ID}/transactions.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"transaction":{"kind":"capture","amount":"10.00","parent_id":{AUTHORIZATION_ID}}}'
```

交易类型：`authorization`、`capture`、`sale`、`void`、`refund`

### 系列

#### 列出自定义系列

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/custom_collections.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取自定义系列

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/custom_collections/{COLLECTION_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 创建自定义系列

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/custom_collections.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"custom_collection":{"title":"Summer Collection","body_html":"<p>Summer products</p>"}}'
```

#### 更新自定义系列

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/custom_collections/{COLLECTION_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{"custom_collection":{"id":{COLLECTION_ID},"title":"Updated Collection"}}'
```

#### 删除自定义系列

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/custom_collections/{COLLECTION_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -X DELETE
```

#### 列出智能系列

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/smart_collections.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 获取智能系列

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/smart_collections/{COLLECTION_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

#### 创建智能系列

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/smart_collections.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"smart_collection":{"title":"Sale Items","rules":[{"column":"compare_at_price","relation":"greater_than","condition":"0"}]}}'
```

规则列：`title`、`type`、`vendor`、`variant_price`、`tag`、`compare_at_price`、`weight`、`inventory_stock`、`variant_compare_at_price`、`variant_weight`、`variant_inventory`、`variant_title`

规则关系：`equals`、`not_equals`、`greater_than`、`less_than`、`starts_with`、`ends_with`、`contains`、`not_contains`

### 收集（产品-系列链接）

#### 列出收集

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/collects.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

查询参数：`product_id`、`collection_id`、`limit`、`since_id`、`fields`

#### 创建收集

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/collects.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"collect":{"product_id":{PRODUCT_ID},"collection_id":{COLLECTION_ID}}}'
```

#### 删除收集

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/collects/{COLLECT_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -X DELETE
```

### 废弃结账

#### 列出废弃结账

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/checkouts.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

查询参数：`limit`、`since_id`、`created_at_min`、`created_at_max`、`updated_at_min`、`updated_at_max`、`status`（open、closed）

#### 获取废弃结账数量

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/checkouts/count.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

## 状态参考

### 订单状态

| Field | Values |
|-------|--------|
| `financial_status` | pending, authorized, partially_paid, paid, partially_refunded, refunded, voided |
| `fulfillment_status` | null (unfulfilled), partial, fulfilled, restocked |

### 产品状态

| Status | Description |
|--------|-------------|
| `active` | Available for sale |
| `archived` | No longer available, hidden from admin lists |
| `draft` | Not ready for sale |

### 退货状态

| Status | Description |
|--------|-------------|
| `requested` | Return requested by customer |
| `in_progress` | Return being processed |
| `open` | Return accepted, awaiting items |
| `closed` | Return completed |
| `cancelled` | Return cancelled |

## 分页

Shopify 使用基于游标的分页，通过 `Link` header。

### 使用页面信息

```bash
# 第一个请求
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/products.json?limit=50" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -i
```

响应包括 `Link` header：
```
Link: <https://store.myshopify.com/admin/api/2024-10/products.json?page_info=abc123&limit=50>; rel="next"
```

```bash
# 下一页
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/products.json?page_info=abc123&limit=50" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

注意：使用 `page_info` 时，除了 `limit` 和 `fields` 之外，你不能使用其他查询参数。

## 速率限制

Shopify 使用漏桶算法：

- **桶大小**：40 个请求
- **泄漏速率**：每秒 2 个请求
- **恢复**：完整桶约 20 秒

响应 headers：
- `X-Shopify-Shop-Api-Call-Limit`：当前使用量（例如，`32/40`）
- `Retry-After`：等待秒数（在 429 响应时）

### 最佳实践

1. 检查 `X-Shopify-Shop-Api-Call-Limit` header
2. 如果接近限制，在请求之间添加延迟
3. 在 429 响应时，等待 `Retry-After` 秒
4. 对大数据集使用批量操作

## Webhooks

### 列出 Webhooks

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/webhooks.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN"
```

### 创建 Webhook

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/webhooks.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"webhook":{"topic":"orders/create","address":"https://example.com/webhooks/orders","format":"json"}}'
```

常见主题：`orders/create`、`orders/updated`、`orders/fulfilled`、`orders/cancelled`、`products/create`、`products/update`、`products/delete`、`customers/create`、`customers/update`、`inventory_levels/update`、`refunds/create`

### 删除 Webhook

```bash
curl "https://$SHOPIFY_STORE_DOMAIN/admin/api/2024-10/webhooks/{WEBHOOK_ID}.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -X DELETE
```

## 变更日志

### v1.0.0

- 初始发布，包含完整的 Admin REST API 覆盖
- 订单、产品、变体、客户
- 库存管理（水平、项目、位置）
- 履行和履行订单
- 退款、退货、交易
- 系列（自定义、智能）和收集
- 废弃结账
- Webhooks 管理
- 状态参考表
- 分页和速率限制文档
