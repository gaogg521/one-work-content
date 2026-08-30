---
name: clickup-observability
description: SaaS 可观测性专家 - ClickUp 监控、API 链路追踪、任务系统分析
allowed-tools: Read, Write, Edit
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
tags:
- saas
- productivity
- clickup
compatible-with: claude-code
---

# ClickUp 可观测性

## 概述

监控 ClickUp API v2 集成，包括指标（请求速率、延迟、错误、速率限制使用）、分布式链路追踪和告警。

## 关键指标

| 指标 | 类型 | 标签 | 描述 |
|--------|------|--------|-------------|
| `clickup_requests_total` | 计数器 | method, endpoint, status | API 请求总数 |
| `clickup_request_duration_seconds` | 直方图 | method, endpoint | 请求延迟 |
| `clickup_errors_total` | 计数器 | status_code, error_type | 按类型划分的错误 |
| `clickup_rate_limit_remaining` | 仪表盘 | token_hash | 速率限制余量 |
| `clickup_rate_limit_resets_total` | 计数器 | | 触发 429 的次数 |

## Prometheus 插桩

```typescript
import { Registry, Counter, Histogram, Gauge } from 'prom-client';

const registry = new Registry();

const requestCounter = new Counter({
  name: 'clickup_requests_total',
  help: 'ClickUp API v2 请求总数',
  labelNames: ['method', 'endpoint', 'status'] as const,
  registers: [registry],
});

const requestDuration = new Histogram({
  name: 'clickup_request_duration_seconds',
  help: 'ClickUp API 请求持续时间（秒）',
  labelNames: ['method', 'endpoint'] as const,
  buckets: [0.05, 0.1, 0.25, 0.5, 1, 2, 5],
  registers: [registry],
});

const rateLimitGauge = new Gauge({
  name: 'clickup_rate_limit_remaining',
  help: 'ClickUp 速率限制剩余请求数',
  registers: [registry],
});

const errorCounter = new Counter({
  name: 'clickup_errors_total',
  help: '按状态码划分的 ClickUp API 错误',
  labelNames: ['status_code', 'error_type'] as const,
  registers: [registry],
});
```

## 插桩客户端

```typescript
async function instrumentedClickUpRequest<T>(
  path: string,
  options: RequestInit = {}
): Promise<T> {
  const method = options.method ?? 'GET';
  // 为基数控制规范化端点（替换 UUID）
  const endpoint = path.replace(/\/[a-zA-Z0-9]{6,}(?=\/|$|\?)/g, '/:id');
  const timer = requestDuration.startTimer({ method, endpoint });

  try {
    const response = await fetch(`https://api.clickup.com/api/v2${path}`, {
      ...options,
      headers: {
        'Authorization': process.env.CLICKUP_API_TOKEN!,
        'Content-Type': 'application/json',
        ...options.headers,
      },
    });

    // 更新速率限制仪表盘
    const remaining = response.headers.get('X-RateLimit-Remaining');
    if (remaining) rateLimitGauge.set(parseInt(remaining));

    const status = response.ok ? 'success' : `${response.status}`;
    requestCounter.inc({ method, endpoint, status });

    if (!response.ok) {
      const body = await response.json().catch(() => ({}));
      errorCounter.inc({
        status_code: String(response.status),
        error_type: body.ECODE ?? 'unknown',
      });
      throw new Error(`ClickUp ${response.status}: ${body.err}`);
    }

    return response.json();
  } catch (error) {
    if (!(error instanceof Error && error.message.startsWith('ClickUp'))) {
      errorCounter.inc({ status_code: 'network', error_type: 'fetch_error' });
    }
    throw error;
  } finally {
    timer();
  }
}
```

## OpenTelemetry 链路追踪

```typescript
import { trace, SpanStatusCode } from '@opentelemetry/api';

const tracer = trace.getTracer('clickup-integration', '1.0.0');

async function tracedClickUpCall<T>(
  operationName: string,
  path: string,
  fn: () => Promise<T>
): Promise<T> {
  return tracer.startActiveSpan(`clickup.${operationName}`, async (span) => {
    span.setAttribute('clickup.path', path);
    span.setAttribute('clickup.method', 'GET');

    try {
      const result = await fn();
      span.setStatus({ code: SpanStatusCode.OK });
      return result;
    } catch (error: any) {
      span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
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

const logger = pino({ name: 'clickup', level: process.env.LOG_LEVEL ?? 'info' });

function logClickUpCall(data: {
  method: string;
  path: string;
  status: number;
  durationMs: number;
  rateLimitRemaining?: number;
  error?: string;
}): void {
  const level = data.status >= 500 ? 'error' : data.status >= 400 ? 'warn' : 'info';
  logger[level]({
    service: 'clickup',
    ...data,
  }, `ClickUp ${data.method} ${data.path} -> ${data.status} (${data.durationMs}ms)`);
}
```

## 告警规则

```yaml
# prometheus/clickup_alerts.yaml
groups:
  - name: clickup
    rules:
      - alert: ClickUpHighErrorRate
        expr: rate(clickup_errors_total[5m]) / rate(clickup_requests_total[5m]) > 0.05
        for: 5m
        labels: { severity: warning }
        annotations:
          summary: "ClickUp API 错误率 > 5%"

      - alert: ClickUpHighLatency
        expr: histogram_quantile(0.95, rate(clickup_request_duration_seconds_bucket[5m])) > 3
        for: 5m
        labels: { severity: warning }
        annotations:
          summary: "ClickUp P95 延迟 > 3秒"

      - alert: ClickUpRateLimitLow
        expr: clickup_rate_limit_remaining < 10
        for: 1m
        labels: { severity: critical }
        annotations:
          summary: "ClickUp 速率限制即将耗尽"

      - alert: ClickUpAuthFailures
        expr: increase(clickup_errors_total{status_code="401"}[5m]) > 0
        labels: { severity: critical }
        annotations:
          summary: "检测到 ClickUp 认证失败"
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
| 高基数 | 标签中的动态 ID | 规范化路径（将 ID 替换为 `:id`） |
| 指标缺失 | 未插桩的代码路径 | 通过插桩客户端包装所有 API 调用 |
| 告警风暴 | 阈值过于敏感 | 调整 `for` 持续时间和阈值 |
| 链路追踪断层 | 缺少上下文传播 | 确保传递 span 上下文 |

## 资源

- [Prometheus 最佳实践](https://prometheus.io/docs/practices/naming/)
- [OpenTelemetry JS SDK](https://opentelemetry.io/docs/languages/js/)
- [ClickUp 速率限制](https://developer.clickup.com/docs/rate-limits)

## 下一步

有关事件响应，请参阅 `clickup-incident-runbook`。
