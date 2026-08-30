---
name: claude-agent-sdk
description: 使用 Claude Agent SDK 构建自主 AI agent，支持结构化输出(JSON Schema 验证)、插件系统和事件驱动 hooks。涵盖 14 个常见错误的预防与排查，适用于编码 agent、SRE 系统和安全审计等场景。
user-invocable: True
tags:
- AI
- SRE
- Schema
---

# Claude Agent SDK - 结构化输出与错误预防指南

**包**: @anthropic-ai/claude-agent-sdk@0.2.12
**破坏性变更**: v0.1.45 - 结构化输出（2025年11月），v0.1.0 - 无默认 system prompt，settingSources 必需

---

## v0.1.45+ 的新内容（2025年11月）

**主要特性：**

### 1. 结构化输出（v0.1.45，2025年11月14日）
- **JSON schema 验证** - 保证响应匹配精确的 schemas
- **`outputFormat` 参数** - 使用 JSON schema 或 Zod 定义输出结构
- **访问验证后的结果** - 通过 `message.structured_output`
- **需要 Beta header**: `structured-outputs-2025-11-13`
- **类型安全** - 使用 Zod schemas 实现完整的 TypeScript 推断

**示例：**
```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";
import { zodToJsonSchema } from "zod-to-json-schema";

const schema = z.object({
  summary: z.string(),
  sentiment: z.enum(['positive', 'neutral', 'negative']),
  confidence: z.number().min(0).max(1)
});

const response = query({
  prompt: "Analyze this code review feedback",
  options: {
    model: "claude-sonnet-4-5",
    outputFormat: {
      type: "json_schema",
      json_schema: {
        name: "AnalysisResult",
        strict: true,
        schema: zodToJsonSchema(schema)
      }
    }
  }
});

for await (const message of response) {
  if (message.type === 'result' && message.structured_output) {
    // 保证匹配 schema
    const validated = schema.parse(message.structured_output);
    console.log(`Sentiment: ${validated.sentiment}`);
  }
}
```

**Zod 兼容性（v0.1.71+）：** SDK 支持 Zod v3.24.1+ 和 Zod v4.0.0+ 作为 peer dependencies。导入保持为 `import { z } from "zod"`，适用于任一版本。

### 2. 插件系统（v0.1.27）
- **`plugins` 数组** - 加载本地插件路径
- **自定义插件支持** - 扩展 agent 能力

### 3. Hooks 系统（v0.1.0+）

**所有 12 个 Hook 事件：**

| Hook | 触发时机 | 使用场景 |
|------|------------|----------|
| `PreToolUse` | 工具执行前 | 验证、修改或阻止工具调用 |
| `PostToolUse` | 工具执行后 | 记录结果、触发副作用 |
| `Notification` | Agent 通知 | 显示状态更新 |
| `UserPromptSubmit` | 收到用户 prompt | 预处理或验证输入 |
| `SubagentStart` | Subagent 生成 | 跟踪委派、记录上下文 |
| `SubagentStop` | Subagent 完成 | 聚合结果、清理 |
| `PreCompact` | 上下文压缩前 | 在截断前保存状态 |
| `PermissionRequest` | 需要权限 | 自定义审批工作流 |
| `Stop` | Agent 停止 | 清理、最终记录 |
| `SessionStart` | 会话开始 | 初始化状态 |
| `SessionEnd` | 会话结束 | 持久化状态、清理 |
| `Error` | 发生错误 | 自定义错误处理 |

**Hook 配置：**
```typescript
const response = query({
  prompt: "...",
  options: {
    hooks: {
      PreToolUse: async (input) => {
        console.log(`Tool: ${input.toolName}`);
        return { allow: true };  // 或 { allow: false, message: "..." }
      },
      PostToolUse: async (input) => {
        await logToolUsage(input.toolName, input.result);
      }
    }
  }
});
```

### 4. 附加选项
- **`fallbackModel`** - 失败时自动模型回退
- **`maxThinkingTokens`** - 控制扩展思考预算
- **`strictMcpConfig`** - 严格的 MCP 配置验证
- **`continue`** - 使用新 prompt 恢复（与 `resume` 不同）
- **`permissionMode: 'plan'`** - 用于规划工作流的新权限模式

📚 **文档**: https://platform.claude.com/docs/en/agent-sdk/structured-outputs

---

## 完整的 Claude Agent SDK 参考

## 目录

1. [核心 Query API](#核心-query-api)
2. [工具集成](#工具集成-内置--自定义)
3. [MCP 服务器](#mcp-服务器-model-context-protocol)
4. [Subagent 编排](#subagent-编排)
5. [会话管理](#会话管理)
6. [权限控制](#权限控制)
7. [沙盒设置](#沙盒设置-安全关键)
8. [文件检查点](#文件检查点)
9. [文件系统设置](#文件系统设置)
10. [Query 对象方法](#query-对象方法)
11. [消息类型与流式传输](#消息类型--流式传输)
12. [错误处理](#错误处理)
13. [已知问题](#已知问题-预防)

---

## 核心 Query API

**关键签名：**
```typescript
query(prompt: string | AsyncIterable<SDKUserMessage>, options?: Options)
  -> AsyncGenerator<SDKMessage>
```

**关键选项：**
- `outputFormat` - 结构化 JSON schema 验证（v0.1.45+）
- `settingSources` - 文件系统设置加载（'user'|'project'|'local'）
- `canUseTool` - 自定义权限逻辑回调
- `agents` - 程序化 subagent 定义
- `mcpServers` - MCP 服务器配置
- `permissionMode` - 'default'|'acceptEdits'|'bypassPermissions'|'plan'
- `betas` - 启用 beta 特性（例如，1M 上下文窗口）
- `sandbox` - 安全执行的沙盒设置
- `enableFileCheckpointing` - 启用文件状态快照
- `systemPrompt` - 系统 prompt（字符串或预设对象）

### 扩展上下文（1M Tokens）

启用 100 万 token 上下文窗口：

```typescript
const response = query({
  prompt: "Analyze this large codebase",
  options: {
    betas: ['context-1m-2025-08-07'],  // 启用 1M 上下文
    model: "claude-sonnet-4-5"
  }
});
```

### 系统 Prompt 配置

systemPrompt 的两种形式：

```typescript
// 1. 简单字符串
systemPrompt: "You are a helpful coding assistant."

// 2. 带可选 append 的预设（保留 Claude Code 默认值）
systemPrompt: {
  type: 'preset',
  preset: 'claude_code',
  append: "\n\nAdditional context: Focus on security."
}
```

**使用预设形式** 当你想要 Claude Code 的默认行为加上自定义附加内容时。

---

## 工具集成（内置 + 自定义）

**工具控制：**
- `allowedTools` - 白名单（优先）
- `disallowedTools` - 黑名单
- `canUseTool` - 自定义权限回调（参见权限控制部分）

**内置工具：** Read、Write、Edit、Bash、Grep、Glob、WebSearch、WebFetch、Task、NotebookEdit、BashOutput、KillBash、ListMcpResources、ReadMcpResource、AskUserQuestion

### AskUserQuestion 工具（v0.1.71+）

在 agent 执行期间启用用户交互：

```typescript
const response = query({
  prompt: "Review and refactor the codebase",
  options: {
    allowedTools: ["Read", "Write", "Edit", "AskUserQuestion"]
  }
});

// Agent 现在可以提出澄清问题
// 问题以 tool_call 形式出现在消息流中，名称为 "AskUserQuestion"
```

**使用场景：**
- 在任务中途澄清模糊的需求
- 在执行破坏性操作前获得用户批准
- 展示选项并获取选择

### 工具配置（v0.1.57+）

**工具配置的三种形式：**

```typescript
// 1. 精确的 allowlist（字符串数组）
tools: ["Read", "Write", "Grep"]

// 2. 禁用所有工具（空数组）
tools: []

// 3. 带默认值的预设（对象形式）
tools: { type: 'preset', preset: 'claude_code' }
```

**注意：** `allowedTools` 和 `disallowedTools` 仍然有效，但 `tools` 提供了更大的灵活性。

---

## MCP 服务器（Model Context Protocol）

**服务器类型：**
- **进程内** - `createSdkMcpServer()` 配合 `tool()` 定义
- **外部** - stdio、HTTP、SSE 传输

**工具定义：**
```typescript
tool(name: string, description: string, zodSchema, handler)
```

**Handler 返回：**
```typescript
{ content: [{ type: "text", text: "..." }], isError?: boolean }
```

### 外部 MCP 服务器（stdio）

```typescript
const response = query({
  prompt: "List files and analyze Git history",
  options: {
    mcpServers: {
      // 文件系统服务器
      "filesystem": {
        command: "npx",
        args: ["@modelcontextprotocol/server-filesystem"],
        env: {
          ALLOWED_PATHS: "/Users/developer/projects:/tmp"
        }
      },
      // Git 操作服务器
      "git": {
        command: "npx",
        args: ["@modelcontextprotocol/server-git"],
        env: {
          GIT_REPO_PATH: "/Users/developer/projects/my-repo"
        }
      }
    },
    allowedTools: [
      "mcp__filesystem__list_files",
      "mcp__filesystem__read_file",
      "mcp__git__log",
      "mcp__git__diff"
    ]
  }
});
```

### 外部 MCP 服务器（HTTP/SSE）

```typescript
const response = query({
  prompt: "Analyze data from remote service",
  options: {
    mcpServers: {
      "remote-service": {
        url: "https://api.example.com/mcp",
        headers: {
          "Authorization": "Bearer your-token-here",
          "Content-Type": "application/json"
        }
      }
    },
    allowedTools: ["mcp__remote-service__analyze"]
  }
});
```

### MCP 工具命名约定

**格式**: `mcp__<server-name>__<tool-name>`

**关键：**
- 服务器名称和工具名称必须与配置匹配
- 使用双下划线（`__`）作为分隔符
- 包含在 `allowedTools` 数组中

**示例：** `mcp__weather-service__get_weather`、`mcp__filesystem__read_file`

---

## Subagent 编排

### AgentDefinition 类型

```typescript
type AgentDefinition = {
  description: string;        // 何时使用此 agent
  prompt: string;             // agent 的系统 prompt
  tools?: string[];           // 允许的工具（可选）
  model?: 'sonnet' | 'opus' | 'haiku' | 'inherit';  // 模型（可选）
  skills?: string[];          // 要加载的技能（v0.2.10+）
  maxTurns?: number;          // 停止前的最大轮数（v0.2.10+）
}
```

**字段详情：**

- **description**: 何时使用 agent（由主 agent 用于委派）
- **prompt**: 系统 prompt（定义角色，继承主上下文）
- **tools**: 允许的工具（如果省略，继承自主 agent）
- **model**: 模型覆盖（`haiku`/`sonnet`/`opus`/`inherit`）
- **skills**: 为 agent 加载的技能（v0.2.10+）
- **maxTurns**: 在返回控制前将 agent 限制为 N 轮（v0.2.10+）

**用法：**
```typescript
agents: {
  "security-checker": {
    description: "Security audits and vulnerability scanning",
    prompt: "You check security. Scan for secrets, verify OWASP compliance.",
    tools: ["Read", "Grep", "Bash"],
    model: "sonnet",
    skills: ["security-best-practices"],  // 加载特定技能
    maxTurns: 10  // 限制为 10 轮
  }
}
```

### ⚠️ Subagent 清理警告

**已知问题**：Subagent 在父 agent 停止时不会停止（[Issue #132](https://github.com/anthropics/claude-agent-sdk-typescript/issues/132)）

当父 agent 被停止（通过取消或错误）时，生成的 subagent 会继续作为孤儿进程运行。这可能导致：
- 资源泄漏
- 父 agent 停止后继续执行工具
- 递归场景中的 RAM 内存不足（[Claude Code Issue #4850](https://github.com/anthropics/claude-code/issues/4850)）

**变通方法**：在 Stop hooks 中实现清理：

```typescript
const response = query({
  prompt: "Deploy to production",
  options: {
    agents: {
      "deployer": {
        description: "Handle deployments",
        prompt: "Deploy the application",
        tools: ["Bash"]
      }
    },
    hooks: {
      Stop: async (input) => {
        // 手动清理生成的进程
        console.log("Parent stopped - cleaning up subagents");
        // 实现进程跟踪和终止
      }
    }
  }
});
```

**增强跟踪**：[Issue #142](https://github.com/anthropics/claude-agent-sdk-typescript/issues/142) 提议自动终止

---

## 会话管理

**选项：**
- `resume: sessionId` - 继续之前的会话
- `forkSession: true` - 从会话创建新分支
- `continue: prompt` - 使用新 prompt 恢复（与 `resume` 不同）

**会话 Forking 模式（独特能力）：**

```typescript
// 在不修改原始会话的情况下探索替代方案
const forked = query({
  prompt: "Try GraphQL instead of REST",
  options: {
    resume: sessionId,
    forkSession: true  // 创建新分支，原始会话不变
  }
});
```

**捕获会话 ID：**
```typescript
for await (const message of response) {
  if (message.type === 'system' && message.subtype === 'init') {
    sessionId = message.session_id;  // 保存以供后续 resume/fork
  }
}
```

### V2 会话 API（预览版 - v0.1.54+）

**更简单的多轮对话模式：**

```typescript
import {
  unstable_v2_createSession,
  unstable_v2_resumeSession,
  unstable_v2_prompt
} from "@anthropic-ai/claude-agent-sdk";

// 创建新会话
const session = await unstable_v2_createSession({
  model: "claude-sonnet-4-5",
  workingDirectory: process.cwd(),
  allowedTools: ["Read", "Grep", "Glob"]
});

// 发送 prompt 并流式传输响应
const stream = unstable_v2_prompt(session, "Analyze the codebase structure");
for await (const message of stream) {
  console.log(message);
}

// 在同一会话中继续对话
const stream2 = unstable_v2_prompt(session, "Now suggest improvements");
for await (const message of stream2) {
  console.log(message);
}

// 恢复之前的会话
const resumedSession = await unstable_v2_resumeSession(session.sessionId);
```

**注意：** V2 API 处于预览阶段（`unstable_` 前缀）。`.receive()` 方法在 v0.1.72 中重命名为 `.stream()`。

---

## 权限控制

**权限模式：**
```typescript
type PermissionMode = "default" | "acceptEdits" | "bypassPermissions" | "plan";
```

- `default` - 标准权限检查
- `acceptEdits` - 自动批准文件编辑
- `bypassPermissions` - 跳过所有检查（仅在 CI/CD 中使用）
- `plan` - 规划模式（v0.1.45+）

### 自定义权限逻辑

```typescript
const response = query({
  prompt: "Deploy application to production",
  options: {
    permissionMode: "default",
    canUseTool: async (toolName, input) => {
      // 允许只读操作
      if (['Read', 'Grep', 'Glob'].includes(toolName)) {
        return { behavior: "allow" };
      }

      // 拒绝破坏性 bash 命令
      if (toolName === 'Bash') {
        const dangerous = ['rm -rf', 'dd if=', 'mkfs', '> /dev/'];
        if (dangerous.some(pattern => input.command.includes(pattern))) {
          return {
            behavior: "deny",
            message: "Destructive command blocked for safety"
          };
        }
      }

      // 部署需要确认
      if (input.command?.includes('deploy') || input.command?.includes('kubectl apply')) {
        return {
          behavior: "ask",
          message: "Confirm deployment to production?"
        };
      }

      // 默认允许
      return { behavior: "allow" };
    }
  }
});
```

### canUseTool 回调

```typescript
type CanUseToolCallback = (
  toolName: string,
  input: any
) => Promise<PermissionDecision>;

type PermissionDecision =
  | { behavior: "allow" }
  | { behavior: "deny"; message?: string }
  | { behavior: "ask"; message?: string };
```

**示例：**

```typescript
// 阻止所有文件写入
canUseTool: async (toolName, input) => {
  if (toolName === 'Write' || toolName === 'Edit') {
    return { behavior: "deny", message: "No file modifications allowed" };
  }
  return { behavior: "allow" };
}

// 对特定文件需要确认
canUseTool: async (toolName, input) => {
  const sensitivePaths = ['/etc/', '/root/', '.env', 'credentials.json'];
  if ((toolName === 'Write' || toolName === 'Edit') &&
      sensitivePaths.some(path => input.file_path?.includes(path))) {
    return {
      behavior: "ask",
      message: `Modify sensitive file ${input.file_path}?`
    };
  }
  return { behavior: "allow" };
}

// 记录所有工具使用
canUseTool: async (toolName, input) => {
  console.log(`Tool requested: ${toolName}`, input);
  await logToDatabase(toolName, input);
  return { behavior: "allow" };
}
```

---

## 沙盒设置（安全关键）

**为 Bash 命令启用沙盒执行：**

```typescript
const response = query({
  prompt: "Run system diagnostics",
  options: {
    sandbox: {
      enabled: true,
      autoAllowBashIfSandboxed: true,  // 在沙盒中自动批准 bash
      excludedCommands: ["rm", "dd", "mkfs"],  // 永远不要自动批准这些
      allowUnsandboxedCommands: false  // 拒绝无法沙盒化的命令
    }
  }
});
```

### SandboxSettings 类型

```typescript
type SandboxSettings = {
  enabled: boolean;
  autoAllowBashIfSandboxed?: boolean;  // 默认: false
  excludedCommands?: string[];
  allowUnsandboxedCommands?: boolean;  // 默认: false
  network?: NetworkSandboxSettings;
  ignoreViolations?: SandboxIgnoreViolations;
};

type NetworkSandboxSettings = {
  enabled: boolean;
  proxyUrl?: string;  // 用于网络请求的 HTTP 代理
};
```

**关键选项：**
- `enabled` - 激活沙盒隔离
- `autoAllowBashIfSandboxed` - 跳过安全 bash 命令的权限提示
- `excludedCommands` - 始终需要权限的命令
- `allowUnsandboxedCommands` - 允许无法沙盒化的命令（有风险）
- `network.proxyUrl` - 通过代理路由网络以进行监控

**最佳实践：** 在生产 agent 处理不受信任的输入时始终使用沙盒。

---

## 文件检查点

**启用文件状态快照以实现回滚能力：**

```typescript
const response = query({
  prompt: "Refactor the authentication module",
  options: {
    enableFileCheckpointing: true  // 启用文件快照
  }
});

// 稍后：将文件更改回退到特定点
for await (const message of response) {
  if (message.type === 'user' && message.uuid) {
    // 稍后可以将文件回退到此点
    const userMessageUuid = message.uuid;

    // 回退（在 Query 对象上调用）
    await response.rewindFiles(userMessageUuid);
  }
}
```

**使用场景：**
- 撤销失败的重构尝试
- A/B 测试代码更改
- 安全地探索替代方案

---

## 文件系统设置

**设置来源：**
```typescript
type SettingSource = 'user' | 'project' | 'local';
```

- `user` - `~/.claude/settings.json`（全局）
- `project` - `.claude/settings.json`（团队共享）
- `local` - `.claude/settings.local.json`（gitignored 覆盖）

**默认：** 不加载设置（`settingSources: []`）

### 设置优先级

当加载多个来源时，设置按此顺序合并（优先级从高到低）：

1. **程序化选项**（传递给 `query()`）- 始终优先
2. **本地设置**（`.claude/settings.local.json`）
3. **项目设置**（`.claude/settings.json`）
4. **用户设置**（`~/.claude/settings.json`）

**示例：**

```typescript
// .claude/settings.json
{
  "allowedTools": ["Read", "Write", "Edit"]
}

// .claude/settings.local.json
{
  "allowedTools": ["Read"]  // 覆盖项目设置
}

// 程序化
const response = query({
  options: {
    settingSources: ["project", "local"],
    allowedTools: ["Read", "Grep"]  // ← 这个优先
  }
});

// 实际 allowedTools: ["Read", "Grep"]
```

**最佳实践：** 在 CI/CD 中使用 `settingSources: ["project"]` 以获得一致的行为。

---

## Query 对象方法

`query()` 函数返回一个带有这些方法的 `Query` 对象：

```typescript
const q = query({ prompt: "..." });

// 异步迭代（主要用法）
for await (const message of q) { ... }

// 运行时模型控制
await q.setModel("claude-opus-4-5");           // 在会话中更改模型
await q.setMaxThinkingTokens(4096);            // 设置思考预算

// 内省
const models = await q.supportedModels();     // 列出可用模型
const commands = await q.supportedCommands(); // 列出可用命令
const account = await q.accountInfo();        // 获取账户详情

// MCP 状态
const status = await q.mcpServerStatus();     // 检查 MCP 服务器状态
// 返回: { [serverName]: { status: 'connected' | 'failed', error?: string } }

// 文件操作（需要 enableFileCheckpointing）
await q.rewindFiles(userMessageUuid);         // 回退到检查点
```

**使用场景：**
- 根据任务复杂度动态切换模型
- 监控 MCP 服务器健康
- 为推理任务调整思考预算

---

## 消息类型与流式传输

**消息类型：**
- `system` - 会话初始化/完成（包括 `session_id`）
- `assistant` - Agent 响应
- `tool_call` - 工具执行请求
- `tool_result` - 工具执行结果
- `error` - 错误消息
- `result` - 最终结果（v0.1.45+ 包括 `structured_output`）

**流式模式：**
```typescript
for await (const message of response) {
  if (message.type === 'system' && message.subtype === 'init') {
    sessionId = message.session_id;  // 捕获以供 resume/fork
  }
  if (message.type === 'result' && message.structured_output) {
    // 结构化输出可用（v0.1.45+）
    const validated = schema.parse(message.structured_output);
  }
}
```

---

## 错误处理

**错误代码：**

| 错误代码 | 原因 | 解决方案 |
|------------|-------|----------|
| `CLI_NOT_FOUND` | Claude Code 未安装 | 安装：`npm install -g @anthropic-ai/claude-code` |
| `AUTHENTICATION_FAILED` | API key 无效 | 检查 ANTHROPIC_API_KEY 环境变量 |
| `RATE_LIMIT_EXCEEDED` | 请求过多 | 使用退避实现重试 |
| `CONTEXT_LENGTH_EXCEEDED` | Prompt 太长 | 使用会话压缩，减少上下文 |
| `PERMISSION_DENIED` | 工具被阻止 | 检查 permissionMode、canUseTool |
| `TOOL_EXECUTION_FAILED` | 工具错误 | 检查工具实现 |
| `SESSION_NOT_FOUND` | 会话 ID 无效 | 验证会话 ID |
| `MCP_SERVER_FAILED` | 服务器错误 | 检查服务器配置 |

---

## 已知问题预防

此技能预防 **14** 个已记录的问题：

### 问题 #1：CLI Not Found 错误
**错误**：`"Claude Code CLI not installed"`
**来源**：SDK 需要 Claude Code CLI
**原因**：CLI 未全局安装
**预防**：在使用 SDK 之前安装：`npm install -g @anthropic-ai/claude-code`

### 问题 #2：Authentication Failed
**错误**：`"Invalid API key"`
**来源**：缺少或错误的 ANTHROPIC_API_KEY
**原因**：未设置环境变量
**预防**：始终设置 `export ANTHROPIC_API_KEY="sk-ant-..."`

### 问题 #3：Permission Denied 错误
**错误**：工具执行被阻止
**来源**：`permissionMode` 限制
**原因**：权限不允许该工具
**预防**：使用 `allowedTools` 或自定义 `canUseTool` 回调

### 问题 #4：Context Length Exceeded（会话破坏性）
**错误**：`"Prompt too long"`
**来源**：输入超过模型上下文窗口（[Issue #138](https://github.com/anthropics/claude-agent-sdk-typescript/issues/138)）
**原因**：大型代码库、长对话

**⚠️ 关键行为**：一旦会话达到上下文限制：
1. 所有后续对该会话的请求返回 "Prompt too long"
2. `/compact` 命令因相同错误失败
3. **会话永久损坏，必须放弃**

**预防策略：**

```typescript
// 1. 主动会话 forking（在达到限制前创建检查点）
const checkpoint = query({
  prompt: "Checkpoint current state",
  options: {
    resume: sessionId,
    forkSession: true  // 在达到限制前创建分支
  }
});

// 2. 监控时间并主动轮换会话
const MAX_SESSION_TIME = 80 * 60 * 1000;  // 80 分钟（在 90 分钟崩溃前）
let sessionStartTime = Date.now();

function shouldRotateSession() {
  return Date.now() - sessionStartTime > MAX_SESSION_TIME;
}

// 3. 在达到上下文限制前开始新会话
if (shouldRotateSession()) {
  const summary = await getSummary(currentSession);
  const newSession = query({
    prompt: `Continue with context: ${summary}`
  });
  sessionStartTime = Date.now();
}
```

**注意**：SDK 自动压缩，但如果达到限制，会话将无法恢复

### 问题 #5：Tool Execution Timeout
**错误**：工具无响应
**来源**：长时间运行的工具执行
**原因**：工具耗时过长（默认 >5 分钟）
**预防**：在工具实现中实现超时处理

### 问题 #6：Session Not Found
**错误**：`"Invalid session ID"`
**来源**：会话过期或无效
**原因**：会话 ID 不正确或太旧
**预防**：从 `system` init 消息捕获 `session_id`

### 问题 #7：MCP Server Connection Failed
**错误**：服务器无响应
**来源**：服务器未运行或配置错误
**原因**：命令/URL 不正确，服务器崩溃
**预防**：独立测试 MCP 服务器，验证命令/URL

### 问题 #8：Subagent Definition 错误
**错误**：Invalid AgentDefinition
**来源**：缺少必需字段
**原因**：`description` 或 `prompt` 缺失
**预防**：始终包含 `description` 和 `prompt` 字段

### 问题 #9：Settings File Not Found
**错误**：`"Cannot read settings"`
**来源**：设置文件不存在
**原因**：`settingSources` 包含不存在的文件
**预防**：在包含到 sources 之前检查文件是否存在

### 问题 #10：Tool Name Collision
**错误**：Duplicate tool name
**来源**：多个同名工具
**原因**：两个 MCP 服务器定义了相同的工具名称
**预防**：使用唯一的工具名称，使用服务器名称作为前缀

### 问题 #11：Zod Schema Validation Error
**错误**：Invalid tool input
**来源**：输入与 Zod schema 不匹配
**原因**：Agent 提供了错误的数据类型
**预防**：使用描述性的 Zod schemas 配合 `.describe()`

### 问题 #12：Filesystem Permission Denied
**错误**：Cannot access path
**来源**：文件系统访问受限
**原因**：路径在 `workingDirectory` 之外或没有权限
**预防**：设置正确的 `workingDirectory`，检查文件权限

### 问题 #13：MCP Server Config 缺少 `type` 字段
**错误**：`"Claude Code process exited with code 1"`（隐晦，无上下文）
**来源**：[GitHub Issue #131](https://github.com/anthropics/claude-agent-sdk-typescript/issues/131)
**原因**：基于 URL 的 MCP 服务器需要显式的 `type: "http"` 或 `type: "sse"` 字段
**预防**：始终为基于 URL 的 MCP 服务器指定传输类型

```typescript
// ❌ 错误 - 缺少 type 字段（导致隐晦的 exit code 1）
mcpServers: {
  "my-server": {
    url: "https://api.example.com/mcp"
  }
}

// ✅ 正确 - URL 服务器需要 type 字段
mcpServers: {
  "my-server": {
    url: "https://api.example.com/mcp",
    type: "http"  // 或 "sse" 表示 Server-Sent Events
  }
}
```

**诊断线索**：如果你看到 "process exited with code 1" 且没有其他上下文，请检查你的 MCP 服务器配置是否缺少 `type` 字段。

### 问题 #14：MCP Tool Result 包含 Unicode 行分隔符
**错误**：JSON parse error，agent 挂起
**来源**：[GitHub Issue #137](https://github.com/anthropics/claude-agent-sdk-typescript/issues/137)
**原因**：Unicode U+2028（行分隔符）和 U+2029（段落分隔符）在 JSON 中有效，但会破坏 JavaScript 解析
**预防**：在 MCP 工具结果中转义这些字符

```typescript
// MCP 工具 handler - 清理外部数据
tool("fetch_content", "Fetch text content", {}, async (args) => {
  const content = await fetchExternalData();

  // ✅ 清理 Unicode 行/段落分隔符
  const sanitized = content
    .replace(/ /g, '\\u2028')
    .replace(/ /g, '\\u2029');

  return {
    content: [{ type: "text", text: sanitized }]
  };
});
```

**何时重要**：外部数据源（API、网页抓取、用户输入）可能包含这些字符

**相关**：[MCP Python SDK Issue #1356](https://github.com/modelcontextprotocol/python-sdk/issues/1356)

---

## 官方文档

- **Agent SDK 概述**: https://platform.claude.com/docs/en/api/agent-sdk/overview
- **TypeScript API**: https://platform.claude.com/docs/en/api/agent-sdk/typescript
- **结构化输出**: https://platform.claude.com/docs/en/agent-sdk/structured-outputs
- **GitHub (TypeScript)**: https://github.com/anthropics/claude-agent-sdk-typescript
- **CHANGELOG**: https://github.com/anthropics/claude-agent-sdk-typescript/blob/main/CHANGELOG.md

---

**Token 效率**：
- **不使用 skill**：~15,000 tokens（MCP 设置、权限模式、会话 API、沙盒配置、hooks、结构化输出、错误处理）
- **使用 skill**：~4,500 tokens（全面的 v0.2.12 覆盖 + 错误预防 + 高级模式）
- **节省**：~70%（~10,500 tokens）

**预防的错误**：14 个已记录的问题，带有精确的解决方案（包括 2 个社区来源的陷阱）
**关键价值**：V2 会话 API、沙盒设置、文件检查点、Query 方法、AskUserQuestion 工具、结构化输出（v0.1.45+）、会话 forking、canUseTool 模式、完整的 hooks 系统（12 个事件）、Zod v4 支持、subagent 清理模式

---

**最后验证**：2026-01-20 | **Skill 版本**：3.1.0 | **变更**：添加了 Issue #13（MCP type 字段）、Issue #14（Unicode U+2028/U+2029）、扩展了 Issue #4（会话破坏性）、添加了带有 Stop hook 模式的 subagent 清理警告
