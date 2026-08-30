---
name: afrexai-api-docs
description: 从端点描述生成生产就绪的 API 文档，输出 OpenAPI 3.0 规范、Markdown 参考文档和 SDK 快速入门指南
tags:
- API
---

# API 文档生成器

从端点描述生成生产就绪的 API 文档。输出 OpenAPI 3.0、markdown 参考文档和 SDK 快速入门指南。

## 用法

描述你的 API 端点，本技能生成：

1. **OpenAPI 3.0 规范** — 机器可读，可导入 Swagger/Postman
2. **Markdown 参考** — 带示例的人类可读端点文档
3. **SDK 快速入门** — 供开发者复制粘贴的集成指南

## 说明

当用户描述 API 端点（路由、方法、参数、响应）时：

1. 生成完整的 OpenAPI 3.0 YAML 规范
2. 创建 markdown 文档，包含：
   - 认证部分
   - 每个端点：方法、路径、描述、参数表、请求/响应示例
   - 错误代码参考
   - 速率限制说明
3. 生成快速入门指南，包含 curl 示例和常用语言代码片段（Python、JavaScript、Go）

### 输出格式

```yaml
# openapi.yaml — 完整的 OpenAPI 3.0 规范
```

```markdown
# API Reference — 人类可读文档
```

```markdown
# Quickstart Guide — 集成示例
```

### 质量标准
- 每个端点都必须有请求 AND 响应示例
- 使用真实的示例数据，而不是 "string" 或 "example"
- 包含错误响应（400、401、403、404、500）
- 记录分页、过滤和排序模式
- 注明破坏性变更和版本控制策略

## 技巧
- 粘贴你的路由文件或控制器代码以自动提取
- 支持 REST、GraphQL（schema-first）和 gRPC（proto-first）
- 与 CI/CD 结合，在每次部署时自动生成文档

---

由 [AfrexAI](https://afrexai-cto.github.io/context-packs/) 构建 — 为快速交付团队提供的 AI 驱动业务工具。
