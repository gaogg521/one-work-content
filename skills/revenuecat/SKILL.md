---
name: revenuecat
description: RevenueCat 指标、客户数据和文档搜索。在查询订阅分析、MRR、流失率、客户或 RevenueCat 文档时使用。
license: MIT
metadata:
  openclaw:
    emoji: 😻
    requires:
      bins:
      - curl
      env:
      - RC_API_KEY
    primaryEnv: RC_API_KEY
tags:
- 数据
- 文档
---

# RevenueCat

查询 RevenueCat 指标并搜索文档。

## 配置

设置 `RC_API_KEY` 环境变量，该变量应为 v2 密钥 API 密钥。

## 上下文

查询 RevenueCat API (`GET /projects`) 以获取你有权访问的项目信息。你的 RevenueCat API 密钥允许访问单个项目。在后续 API 调用中使用项目 ID。

## API 访问

```bash
{baseDir}/scripts/rc-api.sh <endpoint>
```

示例：`{baseDir}/scripts/rc-api.sh /projects` 列出项目。

## 本地 API 参考

从 `{baseDir}/references/api-v2.md` 开始了解认证、分页和通用模式。然后加载你需要的域文件：

| 域             | 文件                               | 覆盖内容                                                                                                   |
| ------------------ | ---------------------------------- | -------------------------------------------------------------------------------------------------------- |
| 客户          | `references/customers.md`          | CRUD、属性、别名、权益、订阅、购买、发票、虚拟货币、操作 |
| 订阅      | `references/subscriptions.md`      | 列表、获取、交易、取消、退款、管理 URL                                                  |
| 产品           | `references/products.md`           | CRUD、在商店中创建、测试价格                                                                       |
| 套餐          | `references/offerings.md`          | 套餐、包、包产品                                                                    |
| 权益       | `references/entitlements.md`       | CRUD、附加/分离产品                                                                             |
| 购买          | `references/purchases.md`          | 列表、获取、退款、权益                                                                          |
| 项目           | `references/projects.md`           | 项目、应用、API 密钥、StoreKit 配置                                                                |
| 指标            | `references/metrics.md`            | 概览指标、图表、图表选项                                                                  |
| 付费墙           | `references/paywalls.md`           | 付费墙创建                                                                                         |
| 集成       | `references/integrations.md`       | 集成 CRUD                                                                                        |
| 虚拟货币 | `references/virtual-currencies.md` | 虚拟货币 CRUD                                                                                  |
| 错误处理     | `references/error-handling.md`     | 错误处理                                                                                           |
| 速率限制        | `references/rate-limits.md`        | 速率限制                                                                                              |

只加载与当前任务相关的参考文件 —— 不要全部加载。

## 远程文档搜索

RevenueCat 文档可在 https://www.revenuecat.com/docs 获取。使用 https://www.revenuecat.com/docs/llms.txt 和 /sitemap.xml 作为可用内容的指南。在文档 URL 末尾添加 .md 以获取页面的 markdown 版本。
