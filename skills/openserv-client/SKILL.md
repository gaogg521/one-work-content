---
name: openserv-client
description: 使用 @openserv-labs/client 管理 OpenServ 平台上的代理、工作流、触发器和任务的完整指南。涵盖配置、认证、x402 支付、ERC-8004 链上身份和完整的 Platform API。重要 —— 始终同时阅读配套技能 openserv-agent-sdk，因为构建任何代理都需要这两个包。在 reference.md 中阅读完整的 API 参考。
tags:
- AI
- API
---

# OpenServ Client

`@openserv-labs/client` 包是 OpenServ Platform API 的 TypeScript 客户端。当你的代码需要与平台通信时使用它 —— 注册代理、创建工作流、设置触发器或运行任务。

## 为什么你需要这个包

你的代理（使用 `@openserv-labs/sdk` 构建）在你的机器或服务器上运行。平台不知道它的存在，直到你告诉它：代理是什么、在哪里可以访问它以及如何触发它。客户端就是你做到这一点的方式。它让你创建平台账户（或重用现有账户）、注册你的代理、定义工作流和触发器（webhook、cron、手动或 x402 付费），并绑定凭证以便你的代理可以接受任务。没有它，你的代理将无法进入平台或接收工作。

## 你可以用它做什么

- **Provision** —— 一键设置：创建或重用账户（通过钱包）、注册代理、创建带有触发器和任务的工作流，并获取 API 密钥和认证令牌。通常你每个应用启动时调用一次 `provision()`；它是幂等的。
- **Platform API** —— 通过 `PlatformClient` 进行完全控制：创建和列出代理、工作流、触发器和任务；触发触发器；运行工作流；管理凭证。当你需要超出默认配置流程的功能时使用此选项。
- **Model Parameters** —— 配置平台用于你的代理任务的 LLM 模型和参数。在代理创建/更新或通过 `provision()` 时设置 `model_parameters`。
- **Models API** —— 通过 `client.models.list()` 发现可用的 LLM 模型及其参数模式。
- **x402 payments** —— 将你的代理置于付费墙后面；调用者在任务运行之前按请求付费（例如 USDC）。配置可以设置 x402 触发器并返回付费墙 URL。
- **ERC-8004 on-chain identity** —— 在链上（Base）注册你的代理，铸造身份 NFT，并将你的代理服务端点发布到 IPFS，以便其他人可以以标准方式发现和支付你的代理。

**参考：** `reference.md`（完整 API）· `troubleshooting.md`（常见问题）· `examples/`（可运行代码）

## 安装

```bash
npm install @openserv-labs/client
```

---

## 快速开始：只需 `provision()` + `run()`

**最简单的部署只需两个调用：`provision()` 和 `run()`。** 就是这样。

你需要一个平台账户来注册代理和工作流。最简单的方法是让 `provision()` 为你创建一个：它创建一个钱包并用它注册（无需电子邮件）。该账户在每次运行时都会重用。

有关完整的可运行示例，请参见 `examples/agent.ts`。

> **关键点：** `provision()` 是**幂等的**。每次应用启动时都调用它 —— 无需先检查 `isProvisioned()`。

### `provision()` 做什么

1. 创建或重用以太坊钱包（如果是新的，则创建平台账户）
2. 使用 OpenServ 平台认证
3. 创建或更新代理（幂等）
4. 生成 API 密钥和认证令牌
5. **将凭证绑定到代理实例**（如果提供了 `agent.instance`）
6. 创建或更新带有触发器和任务的工作流
7. 创建工作流图（将触发器链接到任务的边）
8. 激活触发器并将工作流设置为运行状态
9. 将状态持久化到 `.openserv.json`

### 工作流名称和目标

`workflow` 配置需要两个重要属性：

- **`name`** (string) - 这成为 **ERC-8004 中的代理名称**。使其精致、简洁且令人难忘 —— 这是用户看到的面向公众的品牌名称。像产品发布一样思考，而不是变量名。示例：`'Viral Content Engine'`、`'Crypto Alpha Scanner'`、`'Life Catalyst Pro'`。
- **`goal`** (string, required) - 工作流完成内容的详细描述。必须具有描述性和彻底性 —— 简短或模糊的目标将导致 API 调用失败。至少写一个完整的句子来解释工作流的目的。

```typescript
workflow: {
  name: 'Deep Research Pro',
  goal: 'Research any topic in depth, synthesize findings from multiple sources, and produce a comprehensive report with citations',
  trigger: triggers.webhook({ waitForCompletion: true, timeout: 600 }),
  task: { description: 'Research the given topic' }
}
```

### 代理实例绑定 (v1.1+)

将你的代理实例传递给 `provision()` 以进行自动凭证绑定：

```typescript
const agent = new Agent({ systemPrompt: '...' })

await provision({
  agent: {
    instance: agent, // 自动调用 agent.setCredentials()
    name: 'my-agent',
    description: '...',
    model_parameters: { model: 'gpt-5', verbosity: 'medium', reasoning_effort: 'high' } // 可选
  },
  workflow: { ... }
})

// 代理现在已设置 apiKey 和 authToken - 准备好运行 run()
await run(agent)
```

这消除了手动设置 `OPENSERV_API_KEY` 环境变量的需要。

### Model Parameters

可选的 `model_parameters` 字段控制平台在执行你的代理任务时使用的 LLM 模型和参数（包括无运行能力功能和 `generate()` 调用）。如果未提供，则使用平台默认值。

```typescript
await provision({
  agent: {
    instance: agent,
    name: 'my-agent',
    description: '...',
    model_parameters: {
      model: 'gpt-4o',
      temperature: 0.5,
      parallel_tool_calls: false
    }
  },
  workflow: { ... }
})
```

发现可用的模型及其参数：

```typescript
const { models, default: defaultModel } = await client.models.list()
// models: [{ model: 'gpt-5', provider: 'openai', parameters: { ... } }, ...]
// default: 'gpt-5-mini'
```

### Provision 结果

```typescript
interface ProvisionResult {
  agentId: number
  apiKey: string
  authToken?: string
  workflowId: number
  triggerId: string
  triggerToken: string
  paywallUrl?: string // 用于 x402 触发器
  apiEndpoint?: string // 用于 webhook 触发器
}
```

---

## PlatformClient: 完整 API 访问

对于高级用例，直接使用 `PlatformClient`：

```typescript
import { PlatformClient } from '@openserv-labs/client'

// 使用 API 密钥
const client = new PlatformClient({
  apiKey: process.env.OPENSERV_USER_API_KEY
})

// 或使用钱包认证
const client = new PlatformClient()
await client.authenticate(process.env.WALLET_PRIVATE_KEY)
```

有关完整 API 文档，请参见 `reference.md`：

- `client.agents.*` - 代理管理
- `client.workflows.*` - 工作流管理
- `client.triggers.*` - 触发器管理
- `client.tasks.*` - 任务管理
- `client.models.*` - 可用的 LLM 模型和参数
- `client.integrations.*` - 集成连接
- `client.payments.*` - x402 支付
- `client.web3.*` - 积分充值

---

## Triggers Factory

使用 `triggers` 工厂进行类型安全的触发器配置：

```typescript
import { triggers } from '@openserv-labs/client'

// Webhook（免费，公共端点）
triggers.webhook({
  input: { query: { type: 'string', description: 'Search query' } },
  waitForCompletion: true,
  timeout: 600
})

// x402（带有付费墙的付费 API）
triggers.x402({
  name: 'AI Research Assistant',
  description: 'Get comprehensive research reports on any topic',
  price: '0.01',
  timeout: 600,
  input: {
    prompt: {
      type: 'string',
      title: 'Your Request',
      description: 'Describe what you would like the agent to do'
    }
  }
})

// Cron（计划）
triggers.cron({
  schedule: '0 9 * * *', // 每天早上 9 点
  timezone: 'America/New_York'
})

// Manual（仅限平台 UI）
triggers.manual()
```

### Timeout

> **重要：** 始终将 webhook 和 x402 触发器的 `timeout` 设置为至少 **600 秒**（10 分钟）。代理通常需要大量时间来处理请求 —— 特别是在多代理工作流中或执行研究、内容生成或其他复杂任务时。低超时（例如 180 秒）将导致过早失败。如有疑问，请选择更长的超时。对于具有许多顺序步骤的多代理管道，请考虑 900 秒或更长时间。

### Input Schema

为 webhook/x402 付费墙 UI 定义字段：

```typescript
triggers.x402({
  name: 'Content Writer',
  description: 'Generate polished content on any topic',
  price: '0.01',
  input: {
    topic: {
      type: 'string',
      title: 'Content Topic',
      description: 'Enter the subject you want covered'
    },
    style: {
      type: 'string',
      title: 'Writing Style',
      enum: ['formal', 'casual', 'humorous'],
      default: 'casual'
    }
  }
})
```

### Cron 表达式

```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-6, Sunday=0)
* * * * *
```

常见：`0 9 * * *`（每天早上 9 点），`*/5 * * * *`（每 5 分钟），`0 9 * * 1-5`（工作日早上 9 点）

---

## 状态管理

```typescript
import { getProvisionedInfo, clearProvisionedState } from '@openserv-labs/client'

// 获取存储的 ID 和令牌
const info = getProvisionedInfo('my-agent', 'My Awesome Workflow')

// 清除状态（强制全新创建）
clearProvisionedState()
```

---

## 发现和触发 x402 服务

### 发现可用服务（无需认证）

`discoverServices()` 列出所有公共 x402 启用的工作流。**无需认证** —— 你可以在裸 `PlatformClient` 上调用它：

```typescript
import { PlatformClient } from '@openserv-labs/client'

const client = new PlatformClient() // 无需 API 密钥或钱包
const services = await client.payments.discoverServices()

for (const service of services) {
  console.log(`${service.name}: $${service.x402Pricing}`)
  console.log(`URL: ${service.webhookUrl}`)
}
```

### 触发触发器

#### Webhook

```typescript
// 按工作流 ID（推荐）
const result = await client.triggers.fireWebhook({
  workflowId: 123,
  input: { query: 'hello world' }
})

// 或按直接 URL
const result = await client.triggers.fireWebhook({
  triggerUrl: 'https://api.openserv.ai/webhooks/trigger/TOKEN',
  input: { query: 'hello world' }
})
```

#### x402 (程序化)

```typescript
// 按工作流 ID（推荐）
const result = await client.payments.payWorkflow({
  workflowId: 123,
  input: { prompt: 'Hello world' }
})

// 或按直接 URL
const result = await client.payments.payWorkflow({
  triggerUrl: 'https://api.openserv.ai/webhooks/x402/trigger/TOKEN',
  input: { prompt: 'Hello world' }
})
```

---

## 环境变量

| Variable                | Description                  | Required |
| ----------------------- | ---------------------------- | -------- |
| `OPENSERV_USER_API_KEY` | 用户 API 密钥（来自平台） | Yes*    |
| `WALLET_PRIVATE_KEY`    | 用于 SIWE 认证的钱包         | Yes*    |
| `OPENSERV_API_URL`      | 自定义 API URL               | No       |

*需要 API 密钥或钱包密钥之一

---

## ERC-8004: 链上代理身份

在配置后在链上注册你的代理。这在 Identity Registry 上铸造一个 NFT，并将你的代理服务端点发布到 IPFS。

> **需要 Base 上的 ETH。** `provision()` 创建的钱包开始时余额为零。在注册之前，为其提供少量 Base mainnet 上的 ETH。始终使用 try/catch 包装，以便失败不会阻止 `run(agent)` 启动。

> **在 `provision()` 后重新加载 `.env`。** `provision()` 在运行时将 `WALLET_PRIVATE_KEY` 写入 `.env`，但 `process.env` 在启动时已加载空值。在 `provision()` 后使用 `dotenv.config({ override: true })` 来获取新写入的密钥。有关完整的 dotenv 模式，请参见 **openserv-agent-sdk** 技能。

```typescript
import { PlatformClient } from '@openserv-labs/client'

// 重新加载 .env 以获取由 provision() 写入的 WALLET_PRIVATE_KEY
dotenv.config({ override: true })

try {
  const client = new PlatformClient()
  await client.authenticate(process.env.WALLET_PRIVATE_KEY)

  const erc8004 = await client.erc8004.registerOnChain({
    workflowId: result.workflowId,
    privateKey: process.env.WALLET_PRIVATE_KEY!,
    name: 'My Agent',
    description: 'What this agent does',
  })

  console.log(`Agent ID: ${erc8004.agentId}`)         // "8453:42"
  console.log(`Explorer: ${erc8004.blockExplorerUrl}`)
  console.log(`Scan: ${erc8004.scanUrl}`)              // "https://www.8004scan.io/agents/base/42"
} catch (error) {
  console.warn('ERC-8004 registration skipped:', error instanceof Error ? error.message : error)
}
```

- **首次运行** 铸造新 NFT。**重新运行更新 URI** —— 代理 ID 保持不变。
- 默认链是 Base mainnet (8453)。为其他链传递 `chainId` 和 `rpcUrl`。
- **除非你想有意获得新的代理 ID，否则永远不要清除钱包状态。**

有关完整 ERC-8004 API（钱包管理、链助手、故障排除），请参见 `reference.md`。

---

## 清理

```typescript
await client.workflows.delete({ id: workflowId })
await client.agents.delete({ id: agentId })
clearProvisionedState()
```

---

## 更新此技能

此技能可能已过时。OpenServ 团队定期发布更新 —— 在你更新之前，新功能、API 更改和错误修复不会反映在这里。**如果某些功能未按预期工作，或者你想确保拥有最新信息，请在继续之前更新技能。**

```bash
# 检查是否有可用更新
npx skills check

# 将所有已安装技能更新到最新版本
npx skills update
```

或直接重新安装 OpenServ 技能：

```bash
npx skills add openserv-labs/skills
```

---

## 相关技能

- **openserv-agent-sdk** - 使用功能构建代理
- **openserv-multi-agent-workflows** - 多代理协作模式
- **openserv-launch** - 在 Base 区块链上启动令牌
- **openserv-ideaboard-api** - 在 Ideaboard 上查找想法并发布代理服务
