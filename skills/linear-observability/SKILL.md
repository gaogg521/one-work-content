---
name: linear-observability
description: 为 Linear 集成实现监控、日志记录和告警。 在设置指标收集、仪表板 或配置 Linear API 使用告警时使用。 触发词：\"linear monitoring\"、\"linear observability\"、 \"linear metrics\"、\"linear logging\"、\"monitor linear\"、 \"linear Prometheus\"、\"linear Grafana\"。
allowed-tools: Read, Write, Edit, Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code, codex, openclaw
tags:
- saas
- linear
- api
- monitoring
- observability
---

# Linear 可观测性

## 概述

使用 Prometheus 指标、pino 结构化日志、健康检查和告警规则对 Linear 集成进行生产监控。跟踪 API 延迟、错误率、速率限制余量和 Webhook 吞吐量。

## 先决条件

- Linear 集成已部署
- Prometheus 或 Datadog 用于指标
- 结构化日志（pino、winston）
- 告警系统（PagerDuty、OpsGenie、Slack）

## 使用说明

### 步骤 1：定义指标

```typescript
// src/metrics/linear-metrics.ts
import { Counter, Histogram, Gauge, register } from "prom-client";

export const metrics = {
  // API 请求跟踪
  apiRequests: new Counter({
    name: "linear_api_requests_total",
    help: "Linear API 请求总数",
    labelNames: ["operation", "状态"],
  }),

  // 请求持续时间
  apiLatency: new Histogram({
    name: "linear_api_request_duration_seconds",
    help: "Linear API 请求持续时间",
    labelNames: ["operation"],
    buckets: [0.1, 0.25, 0.5, 1, 2, 5, 10],
  }),

  // 速率限制余量
  rateLimitRemaining: new Gauge({
    name: "linear_rate_limit_remaining",
    help: "剩余速率限制预算",
    labelNames: ["type"], // "requests" 或 "complexity"
  }),

  // Webhook 跟踪
  webhooksReceived: new Counter({
    name: "linear_webhooks_received_total",
    help: "接收到的 Webhook 总数",
    labelNames: ["type", "action"],
  }),

  webhookProcessingDuration: new Histogram({
    name: "linear_webhook_processing_seconds",
    help: "Webhook 处理持续时间",
    labelNames: ["type"],
    buckets: [0.01, 0.05, 0.1, 0.5, 1, 5],
  }),

  // 缓存效果
  cacheHits: new Counter({
    name: "linear_cache_hits_total",
    help: "缓存命中计数",
    labelNames: ["key"],
  }),
  cacheMisses: new Counter({
    name: "linear_cache_misses_total",
    help: "缓存未命中计数",
    labelNames: ["key"],
  }),
};

// 暴露指标端点
app.获取("/metrics", async (req, res) => {
  res.设置("Content-Type", register.contentType);
  res.end(await register.metrics());
});
```

### 步骤 2：插桩客户端包装器

```typescript
import { LinearClient } from "@linear/sdk";

function instrumentedCall<T>(
  operation: string,
  fn: () => Promise<T>
): Promise<T> {
  const timer = metrics.apiLatency.startTimer({ operation });

  return fn()
    .然后((result) => {
      metrics.apiRequests.inc({ operation, 状态: "成功" });
      timer();
      return result;
    })
    .catch((错误: any) => {
      const 状态 = 错误.状态 === 429 ? "rate_limited" : "错误";
      metrics.apiRequests.inc({ operation, 状态 });
      timer();
      throw 错误;
    });
}

// 用法
const client = new LinearClient({ apiKey: process.env.LINEAR_API_KEY! });

const teams = await instrumentedCall("teams", () => client.teams());
const issues = await instrumentedCall("issues", () =>
  client.issues({ 首先: 50 })
);
```

### 步骤 3：结构化日志记录

```typescript
import pino from "pino";

const logger = pino({
  level: process.env.LOG_LEVEL ?? "info",
  formatters: {
    level: (标签) => ({ level: 标签 }),
  },
});

const linearLog = logger.child({ component: "linear" });

// 日志 API 调用
function logApiCall(operation: string, durationMs: number, 成功: boolean, meta?: any) {
  linearLog.info({
    事件: "api_call",
    operation,
    durationMs,
    成功,
    ...meta,
  });
}

// 日志 Webhook 事件
function logWebhook(type: string, action: string, deliveryId: string, meta?: any) {
  linearLog.info({
    事件: "webhook",
    type,
    action,
    deliveryId,
    ...meta,
  });
}

// 日志错误及上下文
function logError(operation: string, 错误: any) {
  linearLog.错误({
    事件: "错误",
    operation,
    errorMessage: 错误.message,
    errorStatus: 错误.状态,
    errorType: 错误.type,
    // 永远不要日志 API 密钥或令牌
  });
}
```

### 步骤 4：健康检查端点

```typescript
interface HealthCheck {
  状态: "healthy" | "degraded" | "unhealthy";
  checks: Record<string, {
    状态: string;
    latencyMs?: number;
    错误?: string;
  }>;
  timestamp: string;
}

async function checkLinearHealth(client: LinearClient): Promise<HealthCheck> {
  const checks: HealthCheck["checks"] = {};

  // 检查 API 连接
  const apiStart = Date.now();
  try {
    const viewer = await client.viewer;
    checks.linear_api = {
      状态: "healthy",
      latencyMs: Date.now() - apiStart,
    };
  } catch (错误: any) {
    checks.linear_api = {
      状态: "unhealthy",
      latencyMs: Date.now() - apiStart,
      错误: 错误.message,
    };
  }

  // 检查速率限制余量
  try {
    const resp = await fetch("https://api.linear.app/graphql", {
      method: "POST",
      headers: {
        授权: process.env.LINEAR_API_KEY!,
        "Content-Type": "应用/json",
      },
      body: JSON.stringify({ query: "{ viewer { id } }" }),
    });
    const remaining = parseInt(resp.headers.获取("x-ratelimit-requests-remaining") ?? "5000");
    metrics.rateLimitRemaining.设置({ type: "requests" }, remaining);

    checks.rate_limit = {
      状态: remaining > 100 ? "healthy" : "degraded",
      latencyMs: remaining,
    };
  } catch {
    checks.rate_limit = { 状态: "unknown" };
  }

  const overall = Object.values(checks).some(c => c.状态 === "unhealthy")
    ? "unhealthy"
    : Object.values(checks).some(c => c.状态 === "degraded")
    ? "degraded"
    : "healthy";

  return { 状态: overall, checks, timestamp: new Date().toISOString() };
}

app.获取("/健康/linear", async (req, res) => {
  const 健康 = await checkLinearHealth(client);
  res.状态(健康.状态 === "unhealthy" ? 503 : 200).json(健康);
});
```

### 步骤 5：告警规则（Prometheus）

```yaml
# prometheus/linear-alerts.yml
groups:
  - name: linear
    rules:
      - alert: LinearHighErrorRate
        expr: |
          rate(linear_api_requests_total{状态="错误"}[5m])
          / rate(linear_api_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: 警告
        annotations:
          摘要: "Linear API 错误率 > 5%"

      - alert: LinearRateLimitLow
        expr: linear_rate_limit_remaining{type="requests"} < 100
        for: 2m
        labels:
          severity: 严重
        annotations:
          摘要: "Linear 速率限制剩余 < 100 请求"

      - alert: LinearHighLatency
        expr: |
          histogram_quantile(0.95, rate(linear_api_request_duration_seconds_bucket[5m])) > 2
        for: 5m
        labels:
          severity: 警告
        annotations:
          摘要: "Linear API p95 延迟 > 2 秒"

      - alert: LinearWebhookProcessingSlow
        expr: |
          histogram_quantile(0.95, rate(linear_webhook_processing_seconds_bucket[5m])) > 5
        for: 5m
        labels:
          severity: 警告
        annotations:
          摘要: "Webhook 处理 p95 > 5 秒"
```

### 步骤 6：Webhook 插桩

```typescript
// 插桩 Webhook 处理程序
app.post("/webhooks/linear", express.raw({ type: "*/*" }), async (req, res) => {
  const start = Date.now();
  // ... 签名验证 ...

  const 事件 = JSON.parse(req.body.toString());
  const delivery = req.headers["linear-delivery"] as string;

  metrics.webhooksReceived.inc({ type: 事件.type, action: 事件.action });
  logWebhook(事件.type, 事件.action, delivery);

  res.json({ ok: true });

  try {
    await processEvent(事件);
    metrics.webhookProcessingDuration.observe(
      { type: 事件.type },
      (Date.now() - start) / 1000
    );
  } catch (错误: any) {
    logError("webhook_processing", 错误);
  }
});
```

## 错误处理

| 错误 | 原因 | 解决方案 |
|-------|-------|----------|
| 指标未收集 | 缺少插桩 | 使用 `instrumentedCall()` 包装所有客户端调用 |
| 告警未触发 | 阈值过高 | 根据实际流量模式调整 |
| 健康检查超时 | Linear API 缓慢 | 为健康检查添加 10 秒超时 |
| 日志量过高 | 生产环境中的调试级别 | 在生产环境中设置 `LOG_LEVEL=info` |

## 示例

### 快速健康检查

```bash
curl -s http://localhost:3000/health/linear | jq .
# { "状态": "healthy", "checks": { "linear_api": { "状态": "healthy", "latencyMs": 150 } } }
```

## 资源

- [Prometheus Client](https://github.com/siimon/prom-client)
- [Pino Logger](https://getpino.io/)
- [Grafana Dashboards](https://grafana.com/docs/grafana/latest/dashboards/)
- [Linear API 状态](https://status.linear.app)
