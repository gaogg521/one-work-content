---
name: elizacloud
description: 管理 elizaOS Cloud - 部署 AI agents、chat completions、image/video generation、voice cloning、knowledge base、containers 和 marketplace。当与 elizaOS Cloud、elizacloud.ai 交互、部署 eliza agents 或管理 cloud-hosted AI agents 时使用。需要 ELIZACLOUD_API_KEY 环境变量。
metadata: None
openclaw: None
requires: None
env:
- ELIZACLOUD_API_KEY
tags:
- AI
- Docker
- API
- Key
- 云服务
---

# elizaOS Cloud

elizaOS Cloud 是一个用于构建、部署和扩展智能 AI agents 的平台。此技能提供对完整 elizaOS Cloud API 的访问，用于管理 agents、生成内容和构建 AI 驱动的应用程序。

## 快速开始

将你的 API key 设置为环境变量：

```bash
export ELIZACLOUD_API_KEY="your_api_key_here"
```

使用包含的 bash 客户端执行常见操作：

```bash
./scripts/elizacloud-client.sh status
./scripts/elizacloud-client.sh agents list
./scripts/elizacloud-client.sh chat agent-id "Hello!"
```

## API 配置

- **Base URL**: `https://elizacloud.ai/api/v1`
- **认证**: 
  - `Authorization: Bearer $ELIZACLOUD_API_KEY`
  - `X-API-Key: $ELIZACLOUD_API_KEY`
- **Content-Type**: `application/json`

## 核心端点

### Chat Completions (兼容 OpenAI)

```bash
curl https://elizacloud.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $ELIZACLOUD_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "your-agent-id",
    "messages": [{"role": "user", "content": "Hello!"}],
    "stream": true
  }'
```

**特性**: 流式传输、function calling、结构化输出

### Agent 管理

**列出 Agents**
```bash
GET /api/my-agents/characters
```

**创建 Agent**
```bash
POST /api/v1/app/agents
{
  "name": "My Assistant",
  "bio": "A helpful AI assistant"
}
```

**获取 Agent**
```bash
GET /api/my-agents/characters/{id}
```

**删除 Agent**
```bash
DELETE /api/my-agents/characters/{id}
```

### 图像生成

```bash
POST /api/v1/images/generate
{
  "prompt": "A futuristic city at sunset",
  "model": "flux-pro",
  "width": 1024,
  "height": 1024
}
```

**模型**: FLUX Pro, FLUX Dev, Stable Diffusion

### 视频生成

```bash
POST /api/v1/video/generate
{
  "prompt": "A peaceful lake with mountains in the background",
  "duration": 5,
  "model": "minimax-01"
}
```

**模型**: MiniMax, Runway

### 语音克隆 (ElevenLabs)

```bash
POST /api/v1/voice/clone
{
  "text": "Hello, this is a test of voice cloning",
  "voice_id": "21m00Tcm4TlvDq8ikWAM",
  "model": "eleven_turbo_v2"
}
```

### 知识库

**上传文档**
```bash
POST /api/v1/knowledge/upload
```

**查询知识**
```bash
POST /api/v1/knowledge/query
{
  "query": "How do I deploy an agent?",
  "limit": 5
}
```

### 容器

**部署容器**
```bash
POST /api/v1/containers
{
  "name": "my-app",
  "image": "nginx:latest",
  "ports": [{"containerPort": 80}]
}
```

### A2A 协议 (Agent-to-Agent)

**发现 Agents**
```bash
GET /api/v1/discovery
```

**发送任务**
```bash
POST /api/a2a
{
  "jsonrpc": "2.0",
  "method": "tasks/send",
  "params": {
    "id": "task_123",
    "message": {
      "role": "user",
      "parts": [{"type": "text", "text": "Analyze this data"}]
    }
  },
  "id": 1
}
```

## API Keys

**创建 API Key**
```bash
POST /api/v1/api-keys
{
  "name": "Production Key",
  "permissions": ["chat", "agents", "images"]
}
```

**可用权限**: `chat`, `embeddings`, `images`, `video`, `voice`, `knowledge`, `agents`, `apps`

## 错误处理

所有错误遵循以下格式：

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "The request body is invalid",
    "details": "Field 'model' is required"
  }
}
```

**常见错误代码**:
- `UNAUTHORIZED` (401): 认证无效/缺失
- `FORBIDDEN` (403): 权限不足
- `NOT_FOUND` (404): 资源未找到
- `RATE_LIMITED` (429): 请求过多
- `INSUFFICIENT_CREDITS` (402): 积分不足

## 速率限制

| 端点         | 速率限制  |
|------------------|-------------|
| Chat completions | 60 req/min  |
| Embeddings       | 100 req/min |
| Image generation | 20 req/min  |
| Video generation | 5 req/min   |

## 示例工作流

### 部署客服 Agent

```bash
# 1. 创建 agent
curl -X POST https://elizacloud.ai/api/v1/app/agents \
  -H "Authorization: Bearer $ELIZACLOUD_API_KEY" \
  -d '{"name": "Support Bot", "bio": "Customer support specialist"}'

# 2. 与 agent 聊天
curl https://elizacloud.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $ELIZACLOUD_API_KEY" \
  -d '{"model": "agent-id", "messages": [{"role": "user", "content": "Help me"}]}'
```

### 生成营销素材

```bash
# 1. 生成图像
curl -X POST https://elizacloud.ai/api/v1/images/generate \
  -H "Authorization: Bearer $ELIZACLOUD_API_KEY" \
  -d '{"prompt": "Modern tech startup logo", "model": "flux-pro"}'

# 2. 生成视频
curl -X POST https://elizacloud.ai/api/v1/video/generate \
  -H "Authorization: Bearer $ELIZACLOUD_API_KEY" \
  -d '{"prompt": "Product demo animation", "duration": 10}'
```

### 使用 A2A 构建 Agent 网络

```bash
# 1. 发现可用 agents
curl https://elizacloud.ai/api/v1/discovery \
  -H "Authorization: Bearer $ELIZACLOUD_API_KEY"

# 2. 委派任务给专业 agent
curl -X POST https://elizacloud.ai/api/a2a \
  -H "Authorization: Bearer $ELIZACLOUD_API_KEY" \
  -d '{"jsonrpc": "2.0", "method": "tasks/send", "params": {"message": {"role": "user", "parts": [{"type": "text", "text": "Analyze financial data"}]}}}'
```

## 入门指南

### 注册
在 [elizacloud.ai/login](https://elizacloud.ai/login) 注册（Privy 认证 —— 需要浏览器）。
新账户获得 **1,000 免费积分** —— 足够测试聊天、图像生成等功能。

### 获取 API Key
```bash
# 注册后，在 Dashboard → API Keys 创建 key
# 或通过 API（认证后）：
POST /api/v1/api-keys
{
  "name": "My OpenClaw Agent",
  "permissions": ["chat", "agents", "images", "video", "voice", "knowledge"]
}
```

### 安装 CLI（可选）
```bash
bun add -g @elizaos/cli
elizaos login
```

## 支付与积分

### 查询余额
```bash
GET /api/v1/credits/balance
```

### 购买积分 (Stripe)
```bash
POST /api/v1/credits/checkout
{ "amount": 5000 }
# 返回 Stripe 结账 URL —— 重定向以完成支付
```

### x402 加密支付 (USDC)
按请求使用加密货币支付 —— 无需预购积分：
```bash
# 在任何 API 请求中包含 x402 支付头
curl -X POST "https://elizacloud.ai/api/v1/chat/completions" \
  -H "X-PAYMENT: <x402-payment-header>" \
  -H "Content-Type: application/json" \
  -d '{"model": "agent-id", "messages": [{"role": "user", "content": "Hello"}]}'
```

### 自动充值
```bash
PUT /api/v1/billing/settings
{
  "autoTopUp": true,
  "threshold": 100,
  "amount": 1000
}
```

### 积分交易记录
```bash
GET /api/credits/transactions?limit=50
```

### 使用摘要
```bash
GET /api/v1/credits/summary
# 返回：组织余额、agent 预算、应用收益、可兑换收益
```

## 钱包与加密 RPC

### 创建加密支付
```bash
POST /api/crypto/payments
```

### 检查支付状态
```bash
GET /api/crypto/status
```

### 认证方法
| 方法 | Header | 使用场景 |
|--------|--------|----------|
| API Key | `Authorization: Bearer ek_xxx` | 服务器到服务器 |
| X-API-Key | `X-API-Key: ek_xxx` | 替代 header |
| x402 | `X-PAYMENT: <header>` | 使用 USDC 按请求付费 |
| Session | Cookie-based | 浏览器应用 |

## 额外资源

- **完整 API 文档**: 查看 `references/api-reference.md` 获取完整端点详情
- **Dashboard**: https://elizacloud.ai/dashboard 用于可视化管理
- **OpenAPI 规范**: https://elizacloud.ai/api/openapi.json
- **SDK**: 提供 TypeScript、Python 客户端
- **社区**: Discord 地址 https://discord.gg/elizaos

## 环境变量

- `ELIZACLOUD_API_KEY`: 你的 elizaOS Cloud API key（必需）
- `ELIZACLOUD_BASE_URL`: API base URL（默认: https://elizacloud.ai/api/v1）

## 安全注意事项

- 切勿将 API key 提交到版本控制
- 为开发/生产环境使用独立的 key
- 定期轮换 key
- 将权限限制在最小必需范围
- 在 dashboard 中监控异常使用
