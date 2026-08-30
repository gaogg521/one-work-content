---
name: api-design-principles
description: 设计直观、可扩展的 REST 和 GraphQL API，涵盖资源建模、HTTP 语义、分页、错误处理、版本控制和 Schema 模式
model: reasoning
tags:
- API
- Schema
- 架构
---

# API 设计原则

## WHAT
设计直观、可扩展的 REST 和 GraphQL API，让开发者喜爱。涵盖资源建模、HTTP 语义、分页、错误处理、版本控制和 GraphQL schema 模式。

## WHEN
- 设计新的 REST 或 GraphQL API
- 在实现前审查 API 规范
- 为团队建立 API 设计标准
- 重构 API 以提高可用性
- 在不同 API 范式之间迁移

## KEYWORDS
REST, GraphQL, API design, HTTP methods, pagination, error handling, versioning, OpenAPI, HATEOAS, schema design

---

## 决策框架：REST vs GraphQL

| 选择 REST 当... | 选择 GraphQL 当... |
|---------------------|------------------------|
| 简单的 CRUD 操作 | 复杂的嵌套数据需求 |
| 面向广泛受众的公共 API | 移动应用需要带宽优化 |
| 重度缓存需求 | 客户端需要指定精确的数据形状 |
| 团队不熟悉 GraphQL | 聚合多个数据源 |
| 简单的响应结构 | 快速演进的前端需求 |

---

## REST API 设计

### 资源命名规则

```
✓ 集合使用复数名词
  GET /api/users
  GET /api/orders
  GET /api/products

✗ 避免动词（让 HTTP 方法充当动词）
  POST /api/createUser     ← 错误
  POST /api/users          ← 正确

✓ 嵌套资源（最多 2 层）
  GET /api/users/{id}/orders
  
✗ 避免深层嵌套
  GET /api/users/{id}/orders/{orderId}/items/{itemId}/reviews  ← 太深
  GET /api/order-items/{id}/reviews                            ← 更好
```

### HTTP 方法和状态码

| 方法 | 用途 | 成功 | 常见错误 |
|--------|---------|---------|---------------|
| GET | 检索 | 200 OK | 404 Not Found |
| POST | 创建 | 201 Created | 400/422 Validation |
| PUT | 替换 | 200 OK | 404 Not Found |
| PATCH | 部分更新 | 200 OK | 404 Not Found |
| DELETE | 移除 | 204 No Content | 404/409 Conflict |

### 完整状态码参考

```python
SUCCESS = {
    200: "OK",           # GET, PUT, PATCH 成功
    201: "Created",      # POST 成功
    204: "No Content",   # DELETE 成功
}

CLIENT_ERROR = {
    400: "Bad Request",           # 格式错误
    401: "Unauthorized",          # 缺少/无效认证
    403: "Forbidden",             # 认证有效，但无权限
    404: "Not Found",             # 资源不存在
    409: "Conflict",              # 状态冲突（重复 email）
    422: "Unprocessable Entity",  # 验证错误
    429: "Too Many Requests",     # 速率限制
}

SERVER_ERROR = {
    500: "Internal Server Error",
    503: "Service Unavailable",   # 临时停机
}
```

### 分页

#### 基于偏移（简单）

```python
GET /api/users?page=2&page_size=20

{
  "items": [...],
  "page": 2,
  "page_size": 20,
  "total": 150,
  "pages": 8
}
```

#### 基于游标（用于大型数据集）

```python
GET /api/users?limit=20&cursor=eyJpZCI6MTIzfQ

{
  "items": [...],
  "next_cursor": "eyJpZCI6MTQzfQ",
  "has_more": true
}
```

### 过滤和排序

```
# 过滤
GET /api/users?status=active&role=admin

# 排序（- 前缀表示降序）
GET /api/users?sort=-created_at,name

# 搜索
GET /api/users?search=john

# 字段选择
GET /api/users?fields=id,name,email
```

### 错误响应格式

始终使用一致的结构：

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "请求验证失败",
    "details": [
      {"field": "email", "message": "Invalid email format"}
    ],
    "timestamp": "2025-10-16T12:00:00Z"
  }
}
```

### FastAPI 实现

```python
from fastapi import FastAPI, Query, Path, HTTPException, status
from pydantic import BaseModel, Field, EmailStr
from typing import Optional, List
from datetime import datetime

app = FastAPI(title="API", version="1.0.0")

# Models
class UserCreate(BaseModel):
    email: EmailStr
    name: str = Field(..., min_length=1, max_length=100)

class User(BaseModel):
    id: str
    email: str
    name: str
    created_at: datetime

class PaginatedResponse(BaseModel):
    items: List[User]
    total: int
    page: int
    page_size: int
    pages: int

# Endpoints
@app.get("/api/users", response_model=PaginatedResponse)
async def list_users(
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    status: Optional[str] = Query(None),
    search: Optional[str] = Query(None)
):
    """List users with pagination and filtering."""
    total = await count_users(status=status, search=search)
    offset = (page - 1) * page_size
    users = await fetch_users(limit=page_size, offset=offset, status=status, search=search)
    
    return PaginatedResponse(
        items=users,
        total=total,
        page=page,
        page_size=page_size,
        pages=(total + page_size - 1) // page_size
    )

@app.post("/api/users", response_model=User, status_code=status.HTTP_201_CREATED)
async def create_user(user: UserCreate):
    """Create new user."""
    if await user_exists(user.email):
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail={"code": "EMAIL_EXISTS", "message": "Email already registered"}
        )
    return await save_user(user)

@app.get("/api/users/{user_id}", response_model=User)
async def get_user(user_id: str = Path(...)):
    """Get user by ID."""
    user = await fetch_user(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@app.delete("/api/users/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(user_id: str):
    """Delete user."""
    if not await fetch_user(user_id):
        raise HTTPException(status_code=404, detail="User not found")
    await remove_user(user_id)
```

---

## GraphQL API 设计

### Schema 结构

```graphql
# Types
type User {
  id: ID!
  email: String!
  name: String!
  createdAt: DateTime!
  orders(first: Int = 20, after: String): OrderConnection!
}

# Pagination (Relay-style)
type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type OrderEdge {
  node: Order!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

# Queries
type Query {
  user(id: ID!): User
  users(first: Int = 20, after: String, search: String): UserConnection!
}

# Mutations with Input/Payload pattern
input CreateUserInput {
  email: String!
  name: String!
  password: String!
}

type CreateUserPayload {
  user: User
  errors: [Error!]
}

type Error {
  field: String
  message: String!
  code: String!
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
}
```

### DataLoader（防止 N+1）

```python
from aiodataloader import DataLoader

class UserLoader(DataLoader):
    async def batch_load_fn(self, user_ids: List[str]) -> List[Optional[dict]]:
        """Load multiple users in single query."""
        users = await fetch_users_by_ids(user_ids)
        user_map = {user["id"]: user for user in users}
        return [user_map.get(uid) for uid in user_ids]

# In resolver
@user_type.field("orders")
async def resolve_orders(user: dict, info):
    loader = info.context["loaders"]["orders_by_user"]
    return await loader.load(user["id"])
```

### 查询保护

```python
# Depth limiting
MAX_QUERY_DEPTH = 5

# Complexity limiting
MAX_QUERY_COMPLEXITY = 100

# Timeout
QUERY_TIMEOUT_SECONDS = 10
```

---

## 版本控制策略

### URL 版本控制（推荐）

```
/api/v1/users
/api/v2/users
```

**优点**: Clear, easy to route, cacheable
**缺点**: Multiple URLs for same resource

### Header 版本控制

```
GET /api/users
Accept: application/vnd.api+json; version=2
```

**优点**: Clean URLs
**缺点**: Less visible, harder to test

### 弃用策略

1. Add deprecation headers: `Deprecation: true`
2. Document migration path
3. Give 6-12 months notice
4. Monitor usage before removal

---

## 速率限制

### Headers

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 742
X-RateLimit-Reset: 1640000000

# When limited:
429 Too Many Requests
Retry-After: 3600
```

### 实现

```python
from datetime import datetime, timedelta

class RateLimiter:
    def __init__(self, calls: int, period: int):
        self.calls = calls
        self.period = period
        self.cache = {}
    
    def check(self, key: str) -> tuple[bool, dict]:
        now = datetime.now()
        if key not in self.cache:
            self.cache[key] = []
        
        # Remove old requests
        cutoff = now - timedelta(seconds=self.period)
        self.cache[key] = [ts for ts in self.cache[key] if ts > cutoff]
        
        remaining = self.calls - len(self.cache[key])
        
        if remaining <= 0:
            return False, {"limit": self.calls, "remaining": 0}
        
        self.cache[key].append(now)
        return True, {"limit": self.calls, "remaining": remaining - 1}
```

---

## 实现前检查清单

### 资源
- [ ] 名词，而非动词
- [ ] 集合使用复数
- [ ] 最多 2 层嵌套

### HTTP
- [ ] 每个动作使用正确的方法
- [ ] 正确的状态码
- [ ] 幂等操作是幂等的

### 数据
- [ ] 所有集合都分页
- [ ] 支持过滤/排序
- [ ] 错误格式一致

### 安全
- [ ] 认证已定义
- [ ] 速率限制已配置
- [ ] 所有字段都有输入验证
- [ ] 强制使用 HTTPS

### 文档
- [ ] 生成 OpenAPI 规范
- [ ] 所有端点都有文档
- [ ] 提供示例

---

## NEVER

- **URL 中的动词**: `/api/getUser` → 使用 `/api/users/{id}` 配合 GET
- **POST 用于检索**: 对安全、幂等的读取使用 GET
- **不一致的错误**: 始终使用相同的错误格式
- **无界列表**: 始终对集合进行分页
- **URL 中的密钥**: 查询参数会被记录
- **没有版本控制的破坏性变更**: 从第一天起就规划演进
- **将数据库 Schema 作为 API**: 即使 schema 变化，API 也应保持稳定
- **忽略 HTTP 语义**: 状态码和方法有其含义
