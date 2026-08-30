---
name: byterover
description: 使用 ByteRover 上下文树管理项目知识。提供两种操作：query（检索知识）和 curate（存储知识）。当用户请求信息查找、模式发现或知识持久化时调用。由 ByteRover Inc. 开发。
metadata: None
author: ByteRover Inc. (https://byterover.dev/)
version: 1.2.1
tags:
- SQL查询
---

# ByteRover 上下文树

一个跨会话持久化的项目级知识库。用它来避免重新发现模式、约定和决策。

## 为什么使用 ByteRover

- **工作前查询**：在实施前获取关于模式、约定和过去决策的现有知识
- **学习后整理**：捕获洞察、决策和错误修复，以便未来会话从知情状态开始

## 快速参考

| 命令 | 何时使用 | 示例 |
|------|----------|------|
| `brv query "question"` | 开始工作前 | `brv query "How is auth implemented?"` |
| `brv curate "context" -f file` | 完成工作后 | `brv curate "JWT 24h expiry" -f auth.ts` |
| `brv status` | 检查前置条件 | `brv status` |

## 何时使用

**Query** 当你需要理解某些内容时：
- "这个代码库中 X 是如何工作的？"
- "Y 有哪些模式？"
- "Z 有什么约定？"

**Curate** 当你学到或创造了有价值的东西时：
- 使用特定模式实现了一个功能
- 修复了一个错误并找到了根本原因
- 做了一个架构决策

## Curate 质量

上下文必须是 **具体的** 且 **可操作的**：

```bash
# 好 — 具体，解释在哪里以及为什么
brv curate "Auth uses JWT 24h expiry, tokens in httpOnly cookies" -f src/auth.ts

# 坏 — 太模糊
brv curate "Fixed auth"
```

**注意：** 上下文参数必须放在 `-f` 标志之前。最多 5 个文件。

## 最佳实践

1. **拆分大上下文** — 对于复杂主题，运行多个 `brv curate` 命令，而不是一个巨大的上下文。较小的块更容易检索和更新。

2. **让 ByteRover 读取文件** — 在整理前不要自己读取文件。使用 `-f` 标志让 ByteRover 直接读取它们：
   ```bash
   # 好 — ByteRover 读取文件
   brv curate "Auth implementation details" -f src/auth.ts -f src/middleware/jwt.ts

   # 浪费 — 重复读取文件
   # [agent reads files] then brv curate "..." -f same-files
   ```

3. **查询要具体** — 查询会阻塞你的工作流。使用精确的问题以获得更快、更相关的结果：
   ```bash
   # 好 — 具体
   brv query "What validation library is used for API request schemas?"

   # 坏 — 模糊，慢
   brv query "How is validation done?"
   ```

4. **标记过时上下文** — 当整理替换现有知识的更新时，明确告诉 ByteRover 进行清理：
   ```bash
   brv curate "OUTDATED: Previous auth used sessions. NEW: Now uses JWT with refresh tokens. Clean up old session-based auth context." -f src/auth.ts
   ```

5. **指定结构期望** — 指导 ByteRover 如何组织知识：
   ```bash
   # 指定主题/领域
   brv curate "Create separate topics for: 1) JWT validation, 2) refresh token flow, 3) logout handling" -f src/auth.ts

   # 指定详细程度
   brv curate "Document the error handling patterns in detail (at least 30 lines covering all error types)" -f src/errors/
   ```

## 前置条件

先运行 `brv status`。如果出现错误，智能体无法修复它们 — 指示用户在他们的 brv 终端中采取行动。详情请参见 [TROUBLESHOOTING.md](TROUBLESHOOTING.md)。

---

**另请参阅：** [WORKFLOWS.md](WORKFLOWS.md) 了解详细模式和示例，[TROUBLESHOOTING.md](TROUBLESHOOTING.md) 了解错误处理
