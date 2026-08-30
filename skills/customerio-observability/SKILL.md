---
name: customerio-observability
description: SaaS 监控专家 - Customer.io 可观测性、邮件指标监控、客户行为分析
allowed-tools: Read, Write, Edit, Bash(npm:*), Bash(npx:*), Glob, Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code, codex, openclaw
tags:
- saas
- customer-io
- monitoring
- observability
- prometheus
---

# Customer.io 可观测性

## 概述

为 Customer.io 集成实现全面的可观测性：Prometheus 指标（延迟、错误率、交付漏斗）、具有 PII 脱敏功能的结构化 JSON 日志记录、OpenTelemetry 链路追踪和 Grafana 仪表板定义。

## 先决条件

- Customer.io 集成已部署
- Prometheus + Grafana（或兼容的指标堆栈）
- 结构化日志系统（推荐 pino）

## 要跟踪的关键指标

| 指标 | 类型 | 描述 | 告警阈值 |
|--------|------|-------------|----------------|
| `cio_api_duration_ms` | 直方图 | API 调用延迟 | p99 > 5000ms |
| `cio_api_requests_total` | 计数器 | 按操作划分的 API 请求总数 | N/A（速率） |
| `cio_api_errors_total` | 计数器 | 按状态码划分的 API 错误 | > 1% 错误率 |
| `cio_email_sent_total` | 计数器 | 事务性 + 活动邮件 | N/A |
| `cio_email_bounced_total` | 计数器 | 退信计数 | > 5% 的发送 |
| `cio_email_complained_total` | 计数器 | 垃圾邮件投诉 | > 0.1% 的发送 |
| `cio_webhook_received_total` | 计数器 | 按指标类型划分的 Webhook 事件 | N/A |
| `cio_queue_depth` | 仪表盘 | 事件队列中的待处理项目 | > 10K |

## 使用说明

### 步骤 1：Prometheus 指标

```typescript
// lib/customerio-metrics.ts
import { Counter, Histogram, Gauge, Registry } from "prom-client";

const registry = new Registry();

export const cioMetrics = {
  apiDuration: new Histogram({
    name: "cio_api_duration_ms",
    help: "Customer.io API 调用持续时间（毫秒）",
    labelNames: ["operation", "status"] as const,
    buckets: [10, 25, 50, 100, 250, 500, 1000, 2500, 5000],
    registers: [registry],
  }),

  apiRequests: new Counter({
    name: "cio_api_requests_total",
    help: "Customer.io API 请求总数",
    labelNames: ["operation"] as const,
    registers: [registry],
  }),

  apiErrors: new Counter({
    name: "cio_api_errors_total",
    help: "Customer.io API 错误",
    labelNames: ["operation", "status_code"] as const,
    registers: [registry],
  }),

  emailSent: new Counter({
    name: "cio_email_sent_total",
    help: "通过 Customer.io 发送的邮件",
    labelNames: ["type"] as const,  // "transactional" 或 "campaign"
    registers: [registry],
  }),

  emailBounced: new Counter({
    name: "cio_email_bounced_total",
    help: "来自 Customer.io webhook 的退信",
    registers: [registry],
  }),

  emailComplained: new Counter({
    name: "cio_email_complained_total",
    help: "来自 Customer.io webhook 的垃圾邮件投诉",
    registers: [registry],
  }),

  webhookReceived: new Counter({
    name: "cio_webhook_received_total",
    help: "接收到的 Webhook 事件",
    labelNames: ["metric"] as const,
    registers: [registry],
  }),

  queueDepth: new Gauge({
    name: "cio_queue_depth",
    help: "Customer.io 事件队列中的待处理项目",
    labelNames: ["queue"] as const,
    registers: [registry],
  }),
};

export { registry };
```

### 步骤 2：插桩客户端

```typescript
// lib/customerio-instrumented.ts
import { TrackClient, APIClient, SendEmailRequest, RegionUS } from "customerio-node";
import { cioMetrics } from "./customerio-metrics";

export class InstrumentedCioClient {
  private track: TrackClient;
  private app: APIClient;

  constructor(siteId: string, trackKey: string, appKey: string) {
    this.track = new TrackClient(siteId, trackKey, { region: RegionUS });
    this.app = new APIClient(appKey, { region: RegionUS });
  }

  async identify(userId: string, attrs: Record<string, any>): Promise<void> {
    const timer = cioMetrics.apiDuration.startTimer({ operation: "identify" });
    cioMetrics.apiRequests.inc({ operation: "identify" });

    try {
      await this.track.identify(userId, attrs);
      timer({ status: "success" });
    } catch (err: any) {
      const code = String(err.statusCode ?? "unknown");
      timer({ status: "error" });
      cioMetrics.apiErrors.inc({ operation: "identify", status_code: code });
      throw err;
    }
  }

  async trackEvent(
    userId: string,
    name: string,
    data?: Record<string, any>
  ): Promise<void> {
    const timer = cioMetrics.apiDuration.startTimer({ operation: "track" });
    cioMetrics.apiRequests.inc({ operation: "track" });

    try {
      await this.track.track(userId, { name, data });
      timer({ status: "success" });
    } catch (err: any) {
      timer({ status: "error" });
      cioMetrics.apiErrors.inc({
        operation: "track",
        status_code: String(err.statusCode ?? "unknown"),
      });
      throw err;
    }
  }

  async sendEmail(request: SendEmailRequest): Promise<any> {
    const timer = cioMetrics.apiDuration.startTimer({ operation: "send_email" });
    cioMetrics.apiRequests.inc({ operation: "send_email" });

    try {
      const result = await this.app.sendEmail(request);
      timer({ status: "success" });
      cioMetrics.emailSent.inc({ type: "transactional" });
      return result;
    } catch (err: any) {
      timer({ status: "error" });
      cioMetrics.apiErrors.inc({
        operation: "send_email",
        status_code: String(err.statusCode ?? "unknown"),
      });
      throw err;
    }
  }
}
```

### 步骤 3：具有 PII 脱敏功能的结构化日志记录

```typescript
// lib/customerio-logger.ts
import pino from "pino";

const logger = pino({
  name: "customerio",
  level: process.env.CUSTOMERIO_LOG_LEVEL ?? "info",
  redact: {
    paths: [
      "*.email",
      "*.phone",
      "*.ip_address",
      "*.password",
      "attrs.email",
      "attrs.phone",
    ],
    censor: "[已脱敏]",
  },
});

export function logCioOperation(
  operation: string,
  data: {
    userId?: string;
    event?: string;
    latencyMs?: number;
    statusCode?: number;
    error?: string;
    attrs?: Record<string, any>;
  }
): void {
  if (data.error) {
    logger.error({ operation, ...data }, `CIO ${operation} 失败`);
  } else {
    logger.info({ operation, ...data }, `CIO ${operation} 完成`);
  }
}

// 用法：
// logCioOperation("identify", {
//   userId: "user-123",
//   latencyMs: 85,
//   attrs: { email: "user@example.com", plan: "pro" }
// });
// 输出：{"level":"info","operation":"identify","userId":"user-123",
//          "latencyMs":85,"attrs":{"email":"[已脱敏]","plan":"pro"},
//          "msg":"CIO identify 完成"}
```

### 步骤 4：Webhook 指标收集

```typescript
// 与 webhook 处理程序集成（参见 customerio-webhooks-events 技能）
function recordWebhookMetrics(event: { metric: string }): void {
  cioMetrics.webhookReceived.inc({ metric: event.metric });

  switch (event.metric) {
    case "bounced":
      cioMetrics.emailBounced.inc();
      break;
    case "spammed":
      cioMetrics.emailComplained.inc();
      break;
    case "sent":
      cioMetrics.emailSent.inc({ type: "campaign" });
      break;
  }
}
```

### 步骤 5：Prometheus 指标端点

```typescript
// routes/metrics.ts
import { Router } from "express";
import { registry } from "../lib/customerio-metrics";

const router = Router();

router.get("/metrics", async (_req, res) => {
  res.set("Content-Type", registry.contentType);
  res.end(await registry.metrics());
});

export default router;
```

### 步骤 6：Grafana 仪表板（JSON 模型）

```json
{
  "title": "Customer.io 集成",
  "panels": [
    {
      "title": "API 延迟 (p50/p95/p99)",
      "type": "timeseries",
      "targets": [
        { "expr": "histogram_quantile(0.50, rate(cio_api_duration_ms_bucket[5m]))" },
        { "expr": "histogram_quantile(0.95, rate(cio_api_duration_ms_bucket[5m]))" },
        { "expr": "histogram_quantile(0.99, rate(cio_api_duration_ms_bucket[5m]))" }
      ]
    },
    {
      "title": "按操作划分的请求速率",
      "type": "timeseries",
      "targets": [
        { "expr": "rate(cio_api_requests_total[5m])" }
      ]
    },
    {
      "title": "错误率 %",
      "type": "stat",
      "targets": [
        { "expr": "rate(cio_api_errors_total[5m]) / rate(cio_api_requests_total[5m]) * 100" }
      ]
    },
    {
      "title": "邮件交付漏斗",
      "type": "bargauge",
      "targets": [
        { "expr": "cio_email_sent_total" },
        { "expr": "cio_email_bounced_total" },
        { "expr": "cio_email_complained_total" }
      ]
    }
  ]
}
```

### 步骤 7：告警规则

```yaml
# prometheus/customerio-alerts.yml
groups:
  - name: customerio
    rules:
      - alert: CioHighErrorRate
        expr: rate(cio_api_errors_total[5m]) / rate(cio_api_requests_total[5m]) > 0.05
        for: 5m
        labels: { severity: critical }
        annotations:
          summary: "Customer.io API 错误率 > 5%"

      - alert: CioHighLatency
        expr: histogram_quantile(0.99, rate(cio_api_duration_ms_bucket[5m])) > 5000
        for: 5m
        labels: { severity: warning }
        annotations:
          summary: "Customer.io p99 延迟 > 5 秒"

      - alert: CioHighBounceRate
        expr: rate(cio_email_bounced_total[1h]) / rate(cio_email_sent_total[1h]) > 0.05
        for: 15m
        labels: { severity: warning }
        annotations:
          summary: "邮件退信率 > 5%"

      - alert: CioSpamComplaints
        expr: rate(cio_email_complained_total[1h]) / rate(cio_email_sent_total[1h]) > 0.001
        for: 5m
        labels: { severity: critical }
        annotations:
          summary: "垃圾邮件投诉率 > 0.1% — 发件人声誉面临风险"
```

## 错误处理

| 问题 | 解决方案 |
|-------|----------|
| 高基数指标 | 不要将 userId 用作标签 — 仅使用 operation + status |
| 日志量过高 | 在生产环境中设置 `CUSTOMERIO_LOG_LEVEL=warn` |
| 指标缺失 | 检查指标注册和抓取配置 |
| 日志中的 PII | 验证 pino 脱敏路径覆盖所有敏感字段 |

## 资源

- [Prometheus 最佳实践](https://prometheus.io/docs/practices/)
- [Grafana 仪表板配置](https://grafana.com/docs/grafana/latest/dashboards/)
- [pino 日志记录器](https://getpino.io/)

## 下一步

在可观测性设置完成后，继续执行 `customerio-advanced-troubleshooting` 进行调试。
