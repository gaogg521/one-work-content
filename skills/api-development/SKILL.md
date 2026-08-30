---
name: api-development
model: reasoning
description: 编排完整 API 开发生命周期的元技能，协调设计、实现、测试、文档和部署等阶段，整合专项技能与命令形成无缝构建工作流。
tags:
- API
- 开发
---

# API Development

通过协调设计、实现、测试和文档，将完整的 API 开发生命周期编排为单一工作流。

## 何时使用此技能

- 从零开始构建新 API
- 向现有 API 添加端点
- 重新设计或重构 API
- 规划 API 版本控制和迁移
- 运行完整的 API 开发周期（设计 → 构建 → 测试 → 文档 → 部署）

---

## 编排流程

按顺序执行以下步骤。每个步骤路由到相应的技能或工具。

### 1. 设计 API

加载 `api-design` 技能以建立资源模型、URL 结构、HTTP 方法语义、错误格式和分页策略。

**交付物：** 资源列表、端点映射、请求/响应 schema、错误格式

### 2. 生成 OpenAPI 规范

从设计生成机器可读的 OpenAPI 3.x 规范。使用 `api-design/assets/openapi-template.yaml` 中的 OpenAPI 模板作为起点。

**交付物：** 包含所有端点、schema、认证方案和示例的 `openapi.yaml`

### 3. 搭建端点

为每个端点生成路由文件、请求/响应类型和验证 schema。按资源分组路由。

**交付物：** 每个资源的路由文件、类型定义、验证 schema

### 4. 实现业务逻辑

编写服务层逻辑，包含输入验证、授权检查、数据库查询和正确的错误传播。保持控制器精简 —— 业务逻辑位于服务层。

**交付物：** 服务模块、仓库层、中间件（认证、速率限制、CORS）

### 5. 测试

在三个层面编写测试：
- **单元测试** —— 服务逻辑、验证、错误处理
- **集成测试** —— 使用真实数据库的端点行为
- **契约测试** —— 响应形状匹配 OpenAPI 规范

**交付物：** 覆盖快乐路径、错误案例、边界情况和认证的测试套件

### 6. 文档

生成包含使用示例和 SDK 代码片段的人类可读 API 文档。确保每个端点都有描述、参数、请求/响应示例和错误代码。

**交付物：** API 文档、变更日志、认证指南

### 7. 版本控制和部署

应用版本控制策略、标记发布、更新变更日志，并通过流水线部署。遵循 `api-versioning` 技能进行弃用和迁移指导。

**交付物：** 版本标签、变更日志条目、部署确认

---

## API 设计决策表

为您的用例选择正确的范式。

| 标准 | REST | GraphQL | gRPC |
|----------|------|---------|------|
| **最适合** | 重度 CRUD 的公共 API | 复杂关系数据、客户端驱动查询 | 内部微服务、高吞吐量 |
| **数据获取** | 每个端点固定响应形状 | 客户端指定精确字段 | 强类型 protobuf 消息 |
| **过度/不足获取** | 常见问题 | 通过设计解决 | 最小 —— schema 是显式的 |
| **缓存** | 原生 HTTP 缓存（ETags、Cache-Control） | 需要自定义缓存 | 无内置 HTTP 缓存 |
| **实时** | 轮询或 WebSockets | Subscriptions（内置） | 双向流 |
| **工具** | 成熟 —— OpenAPI、Postman、curl | 发展中 —— Apollo、Relay、GraphiQL | 成熟 —— protoc、grpcurl、Buf |
| **学习曲线** | 低 | 中等 | 中-高 |
| **版本控制** | URL 或 header 版本控制 | 使用 `@deprecated` 进行 schema 演进 | `.proto` 中的包版本控制 |

**经验法则：** 公共 API 默认使用 REST。当客户端需要跨相关数据的灵活查询时使用 GraphQL。内部服务间通信使用 gRPC。

---

## API 检查清单

在将任何 API 工作标记为完成之前，运行此检查清单。

### 认证与授权

- [ ] 选择认证机制（JWT、OAuth2、API key）
- [ ] 在每个端点强制执行授权规则
- [ ] 令牌验证和范围正确
- [ ] 密钥安全存储（绝不在代码或日志中）

### 速率限制

- [ ] 按端点或消费者层级配置速率限制
- [ ] 响应中包含 `RateLimit-*` header
- [ ] 返回 `429 Too Many Requests` 并附带 `Retry-After` header
- [ ] 为消费者记录速率限制策略

### 分页

- [ ] 所有集合端点都分页
- [ ] 选择分页样式（基于游标或基于偏移）
- [ ] `page_size` 有合理上限
- [ ] 包含总数或 `hasNextPage` 指示器

### 过滤与排序

- [ ] 过滤参数经过验证和清理
- [ ] 排序字段已列入允许列表（无任意列排序）
- [ ] 默认排序顺序已定义并记录

### 错误处理

- [ ] 所有端点使用一致的错误响应 schema
- [ ] 正确的 HTTP 状态码（4xx 客户端、5xx 服务器）
- [ ] 验证错误返回字段级详情
- [ ] 内部错误绝不泄露堆栈跟踪或敏感数据

### 版本控制

- [ ] 版本控制策略已选择并统一应用
- [ ] 破坏性与非破坏性变更政策已记录
- [ ] 弃用时间线通过 `Sunset` header 传达

### CORS

- [ ] 配置允许的源（在生产环境中带凭证时不使用通配符 `*`）
- [ ] 明确列出允许的方法和 header
- [ ] 正确处理预检（`OPTIONS`）请求

### 文档

- [ ] 生成并保持 OpenAPI / Swagger 规范最新
- [ ] 每个端点都有描述、参数和示例响应
- [ ] 认证需求已记录
- [ ] 错误代码和含义已列出
- [ ] 为每个版本维护变更日志

### 安全

- [ ] 所有字段都有输入验证
- [ ] SQL 注入防护
- [ ] 强制使用 HTTPS
- [ ] 敏感数据绝不在 URL 或日志中
- [ ] CORS 配置正确

### 监控

- [ ] 带请求 ID 的结构化日志
- [ ] 配置错误跟踪（Sentry、Datadog 等）
- [ ] 收集性能指标（延迟、错误率）
- [ ] 健康检查端点可用（`/health`）
- [ ] 为错误率峰值配置告警

---

## 技能路由表

| 需求 | 技能 | 用途 |
|------|-------|---------|
| API 设计原则 | `api-design` | 资源建模、HTTP 语义、分页、错误格式 |
| 版本控制策略 | `api-versioning` | 版本生命周期、弃用、迁移模式 |
| 认证 | `auth-patterns` | JWT、OAuth2、sessions、RBAC、MFA |
| 错误处理 | `error-handling` | 错误类型、重试模式、熔断器、HTTP 错误 |
| 速率限制 | `rate-limiting` | 算法、HTTP header、分层限制、分布式限制 |
| 缓存 | `caching` | 缓存策略、HTTP 缓存、失效、Redis 模式 |
| 数据库迁移 | `database-migrations` | Schema 演进、零停机模式、回滚策略 |

---

## NEVER Do

1. **NEVER skip the design phase** —— 直接跳到代码会产生不一致的 API，修复成本高昂
2. **NEVER expose database schema directly** —— API 资源不是数据库表；围绕消费者用例设计
3. **NEVER ship without authentication** —— 每个生产端点都必须有认证策略
4. **NEVER return inconsistent error formats** —— 每个错误响应都必须遵循相同的 schema
5. **NEVER break a published API without a versioning plan** —— 破坏性变更需要新版本、迁移指南和弃用时间线
6. **NEVER deploy without tests and documentation** —— 未经测试的 API 会发布 bug，未记录的 API 会让开发者沮丧
