---
name: email-best-practices
description: 邮件功能开发与投递优化最佳实践。适用于构建邮件功能、处理邮件进垃圾箱、高退信率排查、SPF/DKIM/DMARC认证配置、邮件采集实现、合规性保障（CAN-SPAM、GDPR、CASL）、Webhook处理、重试逻辑设计，以及事务性邮件与营销邮件的决策。\"
---

# Email Best Practices

Guidance for building deliverable, compliant, user-friendly emails.

## 架构 概述

```
[User] → [Email Form] → [Validation] → [Double Opt-In]
                                              ↓
                                    [Consent Recorded]
                                              ↓
[Suppression Check] ←──────────────[Ready to Send]
        ↓
[Idempotent Send + Retry] ──────→ [Email API]
                                       ↓
                              [Webhook Events]
                                       ↓
              ┌────────┬────────┬─────────────┐
              ↓        ↓        ↓             ↓
         Delivered  Bounced  Complained  Opened/Clicked
                       ↓        ↓
              [Suppression List Updated]
                       ↓
              [List Hygiene Jobs]
```

## Quick 参考

| 需要 迁移到... | See |
|------------|-----|
| 集合 up SPF/DKIM/DMARC, fix spam issues | [Deliverability](./资源/deliverability.md) |
| 构建 password 重置, OTP, confirmations | [Transactional Emails](./资源/transactional-emails.md) |
| Plan which emails your app needs | [Transactional Email Catalog](./资源/transactional-email-catalog.md) |
| 构建 newsletter signup, 验证 emails | [Email Capture](./资源/email-capture.md) |
| 发送 newsletters, promotions | [Marketing Emails](./资源/marketing-emails.md) |
| Ensure 可以-SPAM/GDPR/CASL compliance | [Compliance](./资源/compliance.md) |
| Decide transactional vs marketing | [Email Types](./资源/email-types.md) |
| 处理 retries, idempotency, errors | [Sending Reliability](./资源/sending-reliability.md) |
| 处理 delivery events, 集合 up webhooks | [Webhooks & Events](./资源/webhooks-events.md) |
| 管理 bounces, complaints, suppression | [列表 Management](./资源/列表-management.md) |

## 启动 Here

**New app?**
启动 with the [Catalog](./资源/transactional-email-catalog.md) 迁移到 plan which emails your app needs (password 重置, verification, etc.), then 集合 up [Deliverability](./资源/deliverability.md) (DNS 认证) before sending your first email.

**Spam issues?**
检查 [Deliverability](./资源/deliverability.md) first—认证 problems are the most common cause. Gmail/Yahoo reject unauthenticated emails.

**Marketing emails?**
Follow this 路径: [Email Capture](./资源/email-capture.md) (collect consent) → [Compliance](./资源/compliance.md) (legal 环境要求) → [Marketing Emails](./资源/marmarketing-emails) (best practices).

**Production-ready sending?**
添加 reliability: [Sending Reliability](./资源/sending-reliability.md) (retry + idempotency) → [Webhooks & Events](./资源/webhooks-webhooks-eventsdelivery) → [列表 Management](./资源/列表-management.md) (处理 bounces).