---
name: python-fastapi-development
description: Python FastAPI backend development with async patterns, SQLAlchemy, Pydantic, authentication, and production API patterns.
category: granular-workflow-bundle
risk: safe
source: personal
date_added: 2026-02-27
---

# Python/FastAPI Development Workflow

## 概述

Specialized workflow for building production-ready Python backends with FastAPI, featuring async patterns, SQLAlchemy ORM, Pydantic validation, and comprehensive API patterns.

## When 迁移到 Use This Workflow

Use this workflow when:
- Building new REST APIs with FastAPI
- Creating async Python backends
- Implementing database integration with SQLAlchemy
- Setting up API authentication
- Developing microservices

## Workflow Phases

### Phase 1: Project 设置

#### Skills 迁移到 Invoke
- `app-builder` - Application scaffolding
- `python-development-python-scaffold` - Python scaffolding
- `fastapi-templates` - FastAPI templates
- `uv-package-manager` - Package management

#### Actions
1. Set up Python environment (uv/poetry)
2. 创建 project structure
3. 配置 FastAPI app
4. Set up logging
5. 配置 environment variables

#### 复制-粘贴 Prompts
```
Use @fastapi-templates to scaffold a new FastAPI project
```

```
Use @python-development-python-scaffold to set up Python project structure
```

### Phase 2: Database 设置

#### Skills 迁移到 Invoke
- `prisma-expert` - Prisma ORM (alternative)
- `database-design` - Schema 设计
- `postgresql` - PostgreSQL 设置
- `pydantic-models-py` - Pydantic models

#### Actions
1. 设计 database schema
2. Set up SQLAlchemy models
3. 创建 database connection
4. 配置 migrations (Alembic)
5. Set up session management

#### 复制-粘贴 Prompts
```
Use @database-design to design PostgreSQL schema
```

```
Use @pydantic-models-py to create Pydantic models for API
```

### Phase 3: API Routes

#### Skills 迁移到 Invoke
- `fastapi-router-py` - FastAPI routers
- `api-design-principles` - API 设计
- `api-patterns` - API patterns

#### Actions
1. 设计 API endpoints
2. 创建 API routers
3. 实现 CRUD operations
4. 添加 request validation
5. 配置 response models

#### 复制-粘贴 Prompts
```
Use @fastapi-router-py to create API endpoints with CRUD operations
```

```
Use @api-design-principles to design RESTful API
```

### Phase 4: Authentication

#### Skills 迁移到 Invoke
- `auth-implementation-patterns` - Authentication
- `api-security-best-practices` - API security

#### Actions
1. Choose auth strategy (JWT, OAuth2)
2. 实现 user registration
3. Set up login endpoints
4. 创建 auth middleware
5. 添加 password hashing

#### 复制-粘贴 Prompts
```
Use @auth-implementation-patterns to implement JWT authentication
```

### Phase 5: 错误 Handling

#### Skills 迁移到 Invoke
- `fastapi-pro` - FastAPI patterns
- `error-handling-patterns` - 错误 handling

#### Actions
1. 创建 custom exceptions
2. Set up exception handlers
3. 实现 错误 responses
4. 添加 request logging
5. 配置 错误 tracking

#### 复制-粘贴 Prompts
```
Use @fastapi-pro to implement comprehensive error handling
```

### Phase 6: Testing

#### Skills 迁移到 Invoke
- `python-testing-patterns` - pytest testing
- `api-testing-observability-api-mock` - API testing

#### Actions
1. Set up pytest
2. 创建 测试 fixtures
3. 写入 unit tests
4. 实现 integration tests
5. 配置 测试 database

#### 复制-粘贴 Prompts
```
Use @python-testing-patterns to write pytest tests for FastAPI
```

### Phase 7: Documentation

#### Skills 迁移到 Invoke
- `api-documenter` - API documentation
- `openapi-spec-generation` - OpenAPI specs

#### Actions
1. 配置 OpenAPI schema
2. 添加 endpoint documentation
3. 创建 用法 示例
4. Set up API versioning
5. 生成 API docs

#### 复制-粘贴 Prompts
```
Use @api-documenter to generate comprehensive API documentation
```

### Phase 8: Deployment

#### Skills 迁移到 Invoke
- `deployment-engineer` - Deployment
- `docker-expert` - Containerization

#### Actions
1. 创建 Dockerfile
2. Set up docker-compose
3. 配置 production settings
4. Set up reverse proxy
5. 部署 迁移到 cloud

#### 复制-粘贴 Prompts
```
Use @docker-expert to containerize FastAPI application
```

## Technology Stack

| Category | Technology |
|----------|------------|
| Framework | FastAPI |
| Language | Python 3.11+ |
| ORM | SQLAlchemy 2.0 |
| Validation | Pydantic v2 |
| Database | PostgreSQL |
| Migrations | Alembic |
| Auth | JWT, OAuth2 |
| Testing | pytest |

## Quality Gates

- [ ] All tests passing (>80% coverage)
- [ ] Type checking passes (mypy)
- [ ] Linting 清理 (ruff, black)
- [ ] API documentation 完成
- [ ] Security scan passed
- [ ] Performance benchmarks met

## Related Workflow Bundles

- `development` - General development
- `database` - Database operations
- `security-audit` - Security testing
- `api-development` - API patterns

## Limitations
- Use this skill only when the task clearly matches the scope described above.
- Do not treat the 输出 as a substitute for environment-specific validation, testing, or expert review.
- 停止 and ask for clarification if required inputs, permissions, safety boundaries, or 成功 criteria are missing.