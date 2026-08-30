---
name: miro-observability
description: 为 Miro REST API v2 集成设置可观测性，包括 Prometheus 指标、 OpenTelemetry 链路追踪、结构化日志记录和 Grafana 仪表板。 使用短语如 \"miro monitoring\"、\"miro metrics\"、 \"miro observability\"、\"monitor miro\"、\"miro alerts\"、\"miro tracing\" 触发。
allowed-tools: Read, Write, Edit
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
tags:
- saas
- miro
- observability
- monitoring
compatible-with: claude-code
---

# Miro 可观测性

## 概述

为 Miro REST API v2 集成提供全面监控：Prometheus 指标用于请求速率和延迟、OpenTelemetry 链路追踪用于请求流、结构化日志记录，以及用于速率限制和错误条件的告警。

## 关键指标

| 指标 | 类型 | 标签 | 用途 |
|--------|------|--------|---------|
| `miro_requests_total` | 计数器 | method, endpoint, status | 请求量 |
| `miro_request_duration_seconds` | 直方图 | method, endpoint | 延迟分布 |
| `miro_errors_total` | 计数器 | error_type, endpoint | 错误跟踪 |
| `miro_rate_limit_remaining` | 仪表盘 | — | 额度余量 |
| `miro_rate_limit_credits_used` | 仪表盘 | — | 额度消耗 |
| `miro_webhook_events_total` | 计数器 | event_type, item_type | Webhook 量 |
| `miro_token_refresh_total` | 计数器 | status | OAuth 健康 |

## Prometheus 指标

```typescript
import { Registry, Counter, Histogram, Gauge } from 'prom-client';

const registry = new Registry();
registry.setDefaultLabels({ app: 'miro-integration' });

const requestCounter = new Counter({
  name: 'miro_requests_total',
  help: 'Miro REST API v2 请求总数',
  labelNames: ['method', 'endpoint', 'status'] as const,
  registers: [registry],
});

const requestDuration = new Histogram({
  name: 'miro_request_duration_seconds',
  help: 'Miro API 请求延迟',
  labelNames: ['method', 'endpoint'] as const,
  buckets: [0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10],
  registers: [registry],
});

const errorCounter = new Counter({
  name: 'miro_errors_total',
  help: '按类型划分的 Miro API 错误',
  labelNames: ['error_type', 'endpoint'] as const,
  registers: [registry],
});

const rateLimitRemaining = new Gauge({
  name: 'miro_rate_limit_remaining',
  help: '剩余 Miro 速率限制额度',
  registers: [registry],
});

const rateLimitUsed = new Gauge({
  name: 'miro_rate_limit_credits_used',
  help: '当前窗口中使用的 Miro 速率限制额度',
  registers: [registry],
});

const webhookCounter = new Counter({
  name: 'miro_webhook_events_total',
  help: '接收到的 Miro Webhook 事件',
  labelNames: ['event_type', 'item_type'] as const,
  registers: [registry],
});
```

## 插桩 API 客户端

```typescript
class InstrumentedMiroClient {
  async fetch<T>(路径: string, method = 'GET', body?: unknown): Promise<T> {
    const endpoint = this.normalizeEndpoint(路径);
    const timer = requestDuration.startTimer({ method, endpoint });

    try {
      const response = await fetch(`https://api.miro.com${path}`, {
        method,
        headers: {
          'Authorization': `Bearer ${this.token}`,
          'Content-Type': 'application/json',
        },
        ...(body ? { body: JSON.stringify(body) } : {}),
      });

      // 从响应头更新速率限制指标
      const remaining = response.headers.get('X-RateLimit-Remaining');
      const limit = response.headers.get('X-RateLimit-Limit');
      if (remaining) rateLimitRemaining.set(parseInt(remaining));
      if (remaining && limit) {
        rateLimitUsed.set(parseInt(limit) - parseInt(remaining));
      }

      requestCounter.inc({ method, endpoint, status: String(response.status) });

      if (!response.ok) {
        const errorType = response.status === 429 ? 'rate_limit'
          : response.status === 401 ? 'auth'
          : response.status >= 500 ? 'server'
          : 'client';
        errorCounter.inc({ error_type: errorType, endpoint });
        throw new MiroApiError(response.status, await response.text());
      }

      return response.status === 204 ? null as T : await response.json();
    } catch (error) {
      if (!(error instanceof MiroApiError)) {
        errorCounter.inc({ error_type: 'network', endpoint });
      }
      throw error;
    } finally {
      timer();
    }
  }

  // 为指标基数控制规范化端点
  // /v2/boards/uXjVN123/items/345 → /v2/boards/{id}/items/{id}
  private normalizeEndpoint(路径: string): string {
    return path
      .replace(/\/boards\/[^/]+/, '/boards/{id}')
      .replace(/\/items\/[^/]+/, '/items/{id}')
      .replace(/\/sticky_notes\/[^/]+/, '/sticky_notes/{id}')
      .replace(/\/shapes\/[^/]+/, '/shapes/{id}')
      .replace(/\/connectors\/[^/]+/, '/connectors/{id}')
      .replace(/\?.*$/, '');
  }
}
```

## OpenTelemetry 链路追踪

```typescript
import { trace, SpanStatusCode, context } from '@opentelemetry/api';

const tracer = trace.getTracer('miro-client', '1.0.0');

async function tracedMiroFetch<T>(
  路径: string,
  method: string,
  body?: unknown,
): Promise<T> {
  const endpoint = normalizeEndpoint(路径);

  return tracer.startActiveSpan(`miro.${method} ${endpoint}`, async (span) => {
    span.setAttribute('miro.method', method);
    span.setAttribute('miro.endpoint', endpoint);
    span.setAttribute('miro.api_version', 'v2');

    try {
      const result = await instrumentedClient.fetch<T>(路径, method, body);
      span.setStatus({ code: SpanStatusCode.OK });
      return result;
    } catch (error: any) {
      span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
      span.setAttribute('miro.error_status', error.status ?? 0);
      span.recordException(error);
      throw error;
    } finally {
      span.end();
    }
  });
}
```

## 结构化日志记录

```typescript
import pino from 'pino';

const logger = pino({
  name: 'miro-integration',
  level: process.env.LOG_LEVEL ?? 'info',
  redact: ['token', 'accessToken', 'refreshToken', 'authorization'],
});

function logMiroRequest(method: string, 路径: string, status: number, durationMs: number) {
  logger.info({
    service: 'miro',
    event: 'api_request',
    method,
    path: normalizeEndpoint(路径),
    status,
    durationMs: Math.round(durationMs),
    rateLimitRemaining: currentRateLimitRemaining,
  });
}

function logWebhookEvent(event: MiroBoardEvent) {
  logger.info({
    service: 'miro',
    event: 'webhook_received',
    eventType: event.type,           // created | updated | deleted
    itemType: event.item.type,       // sticky_note | shape | card | etc.
    boardId: event.boardId,
    itemId: event.item.id,
  });
}
```

## 告警规则（Prometheus AlertManager）

```yaml
# alerts/miro.yaml
groups:
  - name: miro_alerts
    rules:
      - alert: MiroHighErrorRate
        expr: |
          rate(miro_errors_total[5m]) /
          rate(miro_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Miro API 错误率 > 5%"
          dashboard: "https://grafana.myapp.com/d/miro"

      - alert: MiroHighLatency
        expr: |
          histogram_quantile(0.95,
            rate(miro_request_duration_seconds_bucket[5m])
          ) > 3
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Miro API P95 延迟 > 3 秒"

      - alert: MiroRateLimitLow
        expr: miro_rate_limit_remaining < 5000
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "剩余 Miro 速率限制额度 < 5000"
          runbook: "立即降低请求速率。参见 miro-rate-limits 技能。"

      - alert: MiroAuthFailures
        expr: rate(miro_errors_total{error_type="auth"}[5m]) > 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "检测到 Miro 认证失败"
          runbook: "检查令牌过期。验证 OAuth 范围。"

      - alert: MiroDown
        expr: |
          sum(rate(miro_requests_total{status=~"5.."}[5m])) /
          sum(rate(miro_requests_total[5m])) > 0.5
        for: 3m
        labels:
          severity: critical
        annotations:
          summary: "Miro API >50% 服务器错误 — 检查 status.miro.com"
```

## Grafana 仪表板面板

```json
{
  "panels": [
    {
      "title": "Miro 请求速率 (req/s)",
      "targets": [{ "expr": "sum(rate(miro_requests_total[1m]))" }]
    },
    {
      "title": "Miro 延迟 P50/P95/P99",
      "targets": [
        { "expr": "histogram_quantile(0.50, rate(miro_request_duration_seconds_bucket[5m]))", "legendFormat": "P50" },
        { "expr": "histogram_quantile(0.95, rate(miro_request_duration_seconds_bucket[5m]))", "legendFormat": "P95" },
        { "expr": "histogram_quantile(0.99, rate(miro_request_duration_seconds_bucket[5m]))", "legendFormat": "P99" }
      ]
    },
    {
      "title": "剩余速率限制额度",
      "targets": [{ "expr": "miro_rate_limit_remaining" }]
    },
    {
      "title": "按类型划分的错误率",
      "targets": [{ "expr": "sum by(error_type) (rate(miro_errors_total[5m]))" }]
    },
    {
      "title": "按类型划分的 Webhook 事件",
      "targets": [{ "expr": "sum by(event_type, item_type) (rate(miro_webhook_events_total[5m]))" }]
    }
  ]
}
```

## 指标端点

```typescript
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', registry.contentType);
  res.send(await registry.metrics());
});
```

## 错误处理

| 问题 | 原因 | 解决方案 |
|-------|-------|----------|
| 高基数指标 | 标签中的看板/项目 ID | 规范化端点路径 |
| 链路追踪缺失 | 无上下文传播 | 检查 OpenTelemetry SDK 初始化 |
| 日志中的令牌 | 脱敏不充分 | 使用 pino `redact` 选项 |
| 告警风暴 | 阈值过于敏感 | 增加 `for` 持续时间 |

## 资源

- [Prometheus Client (prom-client)](https://github.com/siimon/prom-client)
- [OpenTelemetry JS](https://opentelemetry.io/docs/languages/js/)
- [Pino Logger](https://getpino.io/)
- [Miro 速率限制](https://developers.miro.com/reference/rate-limiting)

## 下一步

有关事件响应，请参见 `miro-incident-runbook`。
