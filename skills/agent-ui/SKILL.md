---
name: agent-ui
description: 来自 ui.inference.sh 的 React/Next.js batteries-included agent 组件，内置 runtime、tools、streaming、approvals 和 widgets。用于构建 AI 聊天界面、agentic UIs、SaaS copilots 和 assistants。
tags:
- AI
- Kafka
- SaaS
- 推理
---

# Agent Component

来自 [ui.inference.sh](https://ui.inference.sh) 的 batteries-included agent 组件。

## 快速开始

```bash
# 安装 agent 组件
npx shadcn@latest add https://ui.inference.sh/r/agent.json

# 为代理路由添加 SDK
npm install @inferencesh/sdk
```

## 设置

### 1. API 代理路由（Next.js）

```typescript
// app/api/inference/proxy/route.ts
import { route } from '@inferencesh/sdk/proxy/nextjs';
export const { GET, POST, PUT } = route;
```

### 2. 环境变量

```bash
# .env.local
INFERENCE_API_KEY=inf_...
```

### 3. 使用组件

```tsx
import { Agent } from "@/registry/blocks/agent/agent"

export default function Page() {
  return (
    <Agent
      proxyUrl="/api/inference/proxy"
      agentConfig={{
        core_app: { ref: 'openrouter/claude-haiku-45@0fkg6xwb' },
        description: 'a helpful ai assistant',
        system_prompt: 'you are helpful.',
      }}
    />
  )
}
```

## 功能

| 功能 | 描述 |
|---------|-------------|
| Runtime included | 不需要后端逻辑 |
| Tool lifecycle | Pending, progress, approval, results |
| Human-in-the-loop | 内置 approval flows |
| Widgets | 来自 agent 响应的声明式 JSON UI |
| Streaming | 实时 token streaming |
| Client-side tools | 在浏览器中运行的工具 |

## Client-Side Tools 示例

```tsx
import { Agent } from "@/registry/blocks/agent/agent"
import { createScopedTools } from "./blocks/agent/lib/client-tools"

const formRef = useRef<HTMLFormElement>(null)
const scopedTools = createScopedTools(formRef)

<Agent
  proxyUrl="/api/inference/proxy"
  config={{
    core_app: { ref: 'openrouter/claude-haiku-45@0fkg6xwb' },
    tools: scopedTools,
    system_prompt: 'You can fill forms using scan_ui and fill_field tools.',
  }}
/>
```

## Props

| Prop | Type | 描述 |
|------|------|-------------|
| `proxyUrl` | string | API 代理端点 |
| `name` | string | Agent 名称（可选） |
| `config` | AgentConfig | Agent 配置 |
| `allowFiles` | boolean | 启用文件上传 |
| `allowImages` | boolean | 启用图像上传 |

## 相关技能

```bash
# Chat UI 构建块
npx skills add inference-sh/agent-skills@chat-ui

# 来自 JSON 的声明式 widgets
npx skills add inference-sh/agent-skills@widgets-ui

# Tool lifecycle UI
npx skills add inference-sh/agent-skills@tools-ui
```

## 文档

- [Agents Overview](https://inference.sh/docs/agents/overview) - 构建 AI agent
- [Agent SDK](https://inference.sh/docs/api/agent/overview) - 编程式 agent 控制
- [Human-in-the-Loop](https://inference.sh/docs/runtime/human-in-the-loop) - Approval flows
- [Agents That Generate UI](https://inference.sh/blog/ux/generative-ui) - 构建生成式 UI
- [Agent UX Patterns](https://inference.sh/blog/ux/agent-ux-patterns) - 最佳实践

组件文档：[ui.inference.sh/blocks/agent](https://ui.inference.sh/blocks/agent)
