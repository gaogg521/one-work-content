---
name: nodejs-patterns
model: standard
description: 生产级 Node.js 后端模式 —— Express/Fastify 搭建、分层架构、中间件、错误处理、 验证、数据库集成、认证和缓存。 当用户构建 REST API、搭建 Node.js 服务器、实现认证、集成数据库、 添加验证/缓存或构建后端应用时使用。
tags:
- API
- 模式
---

# Node.js 后端模式

使用 TypeScript 构建可扩展、可维护的 Node.js 后端应用程序的模式。

## 绝对禁止

- **绝不在代码中存储 secrets** —— 使用环境变量，切勿硬编码凭证
- **绝不跳过输入验证** —— 在 middleware 层使用 Zod/Joi 验证所有输入
- **绝不在生产环境暴露错误详情** —— 返回通用消息，在服务端记录详情
- **绝不使用 `any` 类型** —— TypeScript 类型可防止运行时错误
- **绝不跳过错误处理** —— 始终包装异步处理程序，使用全局错误中间件
- **绝不使用同步操作** —— 对 I/O 使用 async/await，绝不在处理程序中使用 `fs.readFileSync`
- **绝不信任客户端输入** —— 清理、验证并对所有查询进行参数化

## 何时使用

- 使用 Express 或 Fastify 构建 REST API
- 设置中间件流水线和错误处理
- 实现认证和授权
- 集成数据库，使用连接池和事务
- 添加验证、缓存和速率限制

## 项目结构 —— 分层架构

```
src/
├── controllers/     # 处理 HTTP 请求/响应
├── services/        # 业务逻辑
├── repositories/    # 数据访问层
├── models/          # 数据模型和类型
├── middleware/      # 认证、验证、日志、错误处理
├── routes/          # 路由定义
├── config/          # 数据库、缓存、环境配置
└── utils/           # 辅助函数、自定义错误、响应格式化
```

控制器处理 HTTP 相关事务，服务包含业务逻辑，仓库抽象数据访问。每一层只调用其下一层。

## Express 搭建

```typescript
import express from "express";
import helmet from "helmet";
import cors from "cors";
import compression from "compression";

const app = express();

app.use(helmet());
app.use(cors({ origin: process.env.ALLOWED_ORIGINS?.split(",") }));
app.use(compression());
app.use(express.json({ limit: "10mb" }));
app.use(express.urlencoded({ extended: true, limit: "10mb" }));
```

## Fastify 搭建

```typescript
import Fastify from "fastify";
import helmet from "@fastify/helmet";
import cors from "@fastify/cors";

const fastify = Fastify({
  logger: { level: process.env.LOG_LEVEL || "info" },
});

await fastify.register(helmet);
await fastify.register(cors, { origin: true });

// 内置模式验证的类型安全路由
fastify.post<{ Body: { name: string; email: string } }>(
  "/users",
  {
    schema: {
      body: {
        type: "object",
        required: ["name", "email"],
        properties: {
          name: { type: "string", minLength: 1 },
          email: { type: "string", format: "email" },
        },
      },
    },
  },
  async (request) => {
    const { name, email } = request.body;
    return { id: "123", name };
  },
);
```

## 错误处理

### 自定义错误类

```typescript
export class AppError extends Error {
  constructor(
    public message: string,
    public statusCode: number = 500,
    public isOperational: boolean = true,
  ) {
    super(message);
    Object.setPrototypeOf(this, AppError.prototype);
    Error.captureStackTrace(this, this.constructor);
  }
}

export class ValidationError extends AppError {
  constructor(message: string, public errors?: any[]) { super(message, 400); }
}
export class NotFoundError extends AppError {
  constructor(message = "Resource not found") { super(message, 404); }
}
export class UnauthorizedError extends AppError {
  constructor(message = "Unauthorized") { super(message, 401); }
}
export class ForbiddenError extends AppError {
  constructor(message = "Forbidden") { super(message, 403); }
}
```

### 全局错误处理程序

```typescript
import { Request, Response, NextFunction } from "express";
import { AppError, ValidationError } from "../utils/errors";

export const errorHandler = (
  err: Error, req: Request, res: Response, next: NextFunction,
) => {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      status: "error",
      message: err.message,
      ...(err instanceof ValidationError && { errors: err.errors }),
    });
  }

  // 不要在生产环境泄露详情
  const message = process.env.NODE_ENV === "production"
    ? "Internal server error"
    : err.message;

  res.status(500).json({ status: "error", message });
};

// 包装异步路由处理程序以转发错误
export const asyncHandler = (
  fn: (req: Request, res: Response, next: NextFunction) => Promise<any>,
) => (req: Request, res: Response, next: NextFunction) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};
```

## 验证中间件 (Zod)

```typescript
import { AnyZodObject, ZodError } from "zod";

export const validate = (schema: AnyZodObject) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      await schema.parseAsync({
        body: req.body,
        query: req.query,
        params: req.params,
      });
      next();
    } catch (error) {
      if (error instanceof ZodError) {
        const errors = error.errors.map((e) => ({
          field: e.path.join("."),
          message: e.message,
        }));
        next(new ValidationError("Validation failed", errors));
      } else {
        next(error);
      }
    }
  };
};

// 使用
import { z } from "zod";
const createUserSchema = z.object({
  body: z.object({
    name: z.string().min(1),
    email: z.string().email(),
    password: z.string().min(8),
  }),
});
router.post("/users", validate(createUserSchema), userController.createUser);
```

## 认证 —— JWT

### 认证中间件

```typescript
import jwt from "jsonwebtoken";

interface JWTPayload { userId: string; email: string; }

export const authenticate = async (
  req: Request, res: Response, next: NextFunction,
) => {
  try {
    const token = req.headers.authorization?.replace("Bearer ", "");
    if (!token) throw new UnauthorizedError("No token provided");

    req.user = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload;
    next();
  } catch {
    next(new UnauthorizedError("Invalid token"));
  }
};

export const authorize = (...roles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) return next(new UnauthorizedError("Not authenticated"));
    if (!roles.some((r) => req.user?.roles?.includes(r))) {
      return next(new ForbiddenError("Insufficient permissions"));
    }
    next();
  };
};
```

### 认证服务

```typescript
export class AuthService {
  constructor(private userRepository: UserRepository) {}

  async login(email: string, password: string) {
    const user = await this.userRepository.findByEmail(email);
    if (!user || !(await bcrypt.compare(password, user.password))) {
      throw new UnauthorizedError("Invalid credentials");
    }

    return {
      token: jwt.sign(
        { userId: user.id, email: user.email },
        process.env.JWT_SECRET!,
        { expiresIn: "15m" },
      ),
      refreshToken: jwt.sign(
        { userId: user.id },
        process.env.REFRESH_TOKEN_SECRET!,
        { expiresIn: "7d" },
      ),
      user: { id: user.id, name: user.name, email: user.email },
    };
  }
}
```

## 数据库模式

### PostgreSQL 连接池

```typescript
import { Pool, PoolConfig } from "pg";

const pool = new Pool({
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT || "5432"),
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

pool.on("error", (err) => {
  console.error("Unexpected database error", err);
  process.exit(-1);
});

export const closeDatabase = async () => { await pool.end(); };
```

### 事务模式

```typescript
async createOrder(userId: string, items: OrderItem[]) {
  const client = await this.db.connect();
  try {
    await client.query("BEGIN");

    const { rows } = await client.query(
      "INSERT INTO orders (user_id, total) VALUES ($1, $2) RETURNING id",
      [userId, calculateTotal(items)],
    );
    const orderId = rows[0].id;

    for (const item of items) {
      await client.query(
        "INSERT INTO order_items (order_id, product_id, quantity, price) VALUES ($1, $2, $3, $4)",
        [orderId, item.productId, item.quantity, item.price],
      );
      await client.query(
        "UPDATE products SET stock = stock - $1 WHERE id = $2",
        [item.quantity, item.productId],
      );
    }

    await client.query("COMMIT");
    return orderId;
  } catch (error) {
    await client.query("ROLLBACK");
    throw error;
  } finally {
    client.release();
  }
}
```

## 速率限制

```typescript
import rateLimit from "express-rate-limit";
import RedisStore from "rate-limit-redis";
import Redis from "ioredis";

const redis = new Redis({ host: process.env.REDIS_HOST });

export const apiLimiter = rateLimit({
  store: new RedisStore({ client: redis, prefix: "rl:" }),
  windowMs: 15 * 60 * 1000,
  max: 100,
  standardHeaders: true,
  legacyHeaders: false,
});

export const authLimiter = rateLimit({
  store: new RedisStore({ client: redis, prefix: "rl:auth:" }),
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
});
```

## 使用 Redis 缓存

```typescript
import Redis from "ioredis";

const redis = new Redis({
  host: process.env.REDIS_HOST,
  retryStrategy: (times) => Math.min(times * 50, 2000),
});

export class CacheService {
  async get<T>(key: string): Promise<T | null> {
    const data = await redis.get(key);
    return data ? JSON.parse(data) : null;
  }

  async set(key: string, value: any, ttl?: number): Promise<void> {
    const serialized = JSON.stringify(value);
    ttl ? await redis.setex(key, ttl, serialized) : await redis.set(key, serialized);
  }

  async delete(key: string): Promise<void> { await redis.del(key); }

  async invalidatePattern(pattern: string): Promise<void> {
    const keys = await redis.keys(pattern);
    if (keys.length) await redis.del(...keys);
  }
}
```

## API 响应辅助函数

```typescript
export class ApiResponse {
  static success<T>(res: Response, data: T, message?: string, statusCode = 200) {
    return res.status(statusCode).json({ status: "success", message, data });
  }

  static paginated<T>(res: Response, data: T[], page: number, limit: number, total: number) {
    return res.json({
      status: "success",
      data,
      pagination: { page, limit, total, pages: Math.ceil(total / limit) },
    });
  }
}
```

## 最佳实践

1. **使用 TypeScript** —— 类型安全可防止运行时错误
2. **验证所有输入** —— 在 middleware 层使用 Zod 或 Joi
3. **自定义错误类** —— 映射到 HTTP 状态码，使用全局处理程序
4. **切勿硬编码 secrets** —— 使用环境变量
5. **结构化日志** —— 使用 Pino 或 Winston，带请求上下文
6. **速率限制** —— 使用 Redis 支持分布式部署
7. **连接池** —— 数据库始终使用
8. **依赖注入** —— 构造函数注入以提高可测试性
9. **优雅关闭** —— 在 SIGTERM 时关闭 DB 连接池、排空连接
10. **健康检查** —— `/health` 端点用于存活/就绪探针
