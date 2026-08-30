---
name: nexus-sql-builder
description: 用自然语言描述您的数据查询，获取优化的、生产就绪的 SQL，包含适当的 JOIN、窗口函数、CTE 和索引建议。支持 PostgreSQL、MySQL、SQLite 和 SQL Server
version: 1.0.2
capabilities:
- id: invoke-sql-builder
  description: 用自然语言描述您的数据查询，获取优化的、生产就绪的 SQL，包含适当的 JOIN、窗口函数、CTE 和索引建议。支持 PostgreSQL、MySQL、SQLite
    和 SQL Server
permissions:
  network: true
  filesystem: false
  shell: false
inputs:
- name: input
  type: string
  required: true
  description: 服务的主要输入
outputs:
  type: object
  properties:
    result:
      type: string
      description: 处理结果
requires:
  env:
  - NEXUS_PAYMENT_PROOF
metadata: "{\"openclaw\":{\"emoji\":\"\u26a1\",\"requires\":{\"env\":[\"NEXUS_PAYMENT_PROOF\"]},\"primaryEnv\":\"NEXUS_PAYMENT_PROOF\"}}"
---

# NEXUS SQL 架构师

> Cardano 原生 AI 服务，用于自主智能体 | NEXUS AaaS 平台

## 何时使用

当您的智能体需要查询数据库但使用自然语言输入时。提供类似"查找上季度收入最高的客户"的描述，即可获取带有性能优化提示的可执行 SQL。处理复杂的多表 JOIN、聚合和子查询。

## 与众不同之处

方言感知：生成特定于您数据库的语法（PostgreSQL 数组、MySQL LIMIT、SQL Server TOP）。包含查询执行计划分析、索引创建建议，并警告潜在的 N+1 查询模式。处理 CTE、窗口函数和递归查询。

## 步骤

1. 将您的输入准备为 JSON 载荷。
2. 使用 `X-Payment-Proof` 请求头 POST 到 NEXUS API。
3. 解析结构化的 JSON 响应。

### API 调用

```bash
curl -X POST https://ai-service-hub-15.emergent.host/api/original-services/sql-builder \
  -H "Content-Type: application/json" \
  -H "X-Payment-Proof: sandbox_test" \
  -d '{"description": "Find top 10 customers by total spend in last 90 days with their most recent order date", "dialect": "postgresql", "tables": "customers, orders, order_items"}'
```

**端点：** `https://ai-service-hub-15.emergent.host/api/original-services/sql-builder`
**方法：** POST
**请求头：**
- `Content-Type: application/json`
- `X-Payment-Proof: <masumi_payment_id>` （免费沙盒使用 `sandbox_test`）

## 外部端点

| URL | 方法 | 发送数据 |
|-----|--------|-----------|
| `https://ai-service-hub-15.emergent.host/api/original-services/sql-builder` | POST | 输入参数作为 JSON 请求体 |

## 安全与隐私

所有请求通过 HTTPS/TLS 加密到 `https://ai-service-hub-15.emergent.host`。数据不会永久存储 —— 在内存中处理并丢弃。通过 Cardano 上的 Masumi 协议进行支付验证（非托管托管）。不需要文件系统或 shell 权限。

## 模型调用说明

此技能调用 NEXUS AI 服务 API，该 API 使用大型语言模型在服务器端处理请求。您可以通过不安装此技能来选择退出。

## 信任声明

通过安装此技能，输入数据将被传输到 NEXUS (https://ai-service-hub-15.emergent.host) 进行 AI 处理。所有支付均通过 Cardano 进行非托管。访问 https://ai-service-hub-15.emergent.host 查看文档和条款。
