---
name: coreweave-observability
description: GPU 可观测性专家 - CoreWeave 监控、DCGM 指标采集、推理性能追踪
allowed-tools: Read, Write, Edit, Bash(kubectl:*), Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
tags:
- saas
- gpu-cloud
- kubernetes
- inference
- coreweave
compatible-with: claude-code
---

# CoreWeave 可观测性

## GPU 指标 (DCGM Exporter)

CKS 集群预装了 DCGM exporter。关键指标：

| 指标 | 描述 |
|--------|-------------|
| `DCGM_FI_DEV_GPU_UTIL` | GPU 核心利用率 % |
| `DCGM_FI_DEV_FB_USED` | GPU 内存已使用 (MB) |
| `DCGM_FI_DEV_FB_FREE` | GPU 内存空闲 (MB) |
| `DCGM_FI_DEV_POWER_USAGE` | 功耗 (W) |
| `DCGM_FI_DEV_GPU_TEMP` | GPU 温度 (C) |

## Prometheus 告警规则

```yaml
groups:
  - name: coreweave-gpu
    rules:
      - alert: GPUUtilizationLow
        expr: avg(DCGM_FI_DEV_GPU_UTIL) < 20
        for: 30m
        labels: { severity: warning }
        annotations:
          summary: "GPU 利用率低于 20%，持续 30 分钟 — 考虑缩减规模"

      - alert: GPUMemoryHigh
        expr: DCGM_FI_DEV_FB_USED / (DCGM_FI_DEV_FB_USED + DCGM_FI_DEV_FB_FREE) > 0.95
        for: 5m
        labels: { severity: critical }
        annotations:
          summary: "GPU 内存 >95% — OOM 风险"

      - alert: InferencePodDown
        expr: kube_deployment_status_replicas_available{deployment=~".*inference.*"} == 0
        for: 2m
        labels: { severity: critical }
```

## 资源

- [CoreWeave 可观测性](https://www.coreweave.com/observability)

## 下一步

有关事件响应，请参阅 `coreweave-incident-runbook`。
