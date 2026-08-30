---
name: azure-ai-agents-py
description: 基于Azure AI Agents Python SDK（azure-ai-agents）构建AI代理。适用于在Azure AI Foundry上创建托管代理，集成文件搜索、代码解释器、Bing Grounding、Azure AI搜索、函数调用、OpenAPI、MCP等工具，管理线程与消息，实现流式响应及向量存储操作。
package: azure-ai-agents
---

# Azure AI Agents Python SDK

构建 agents hosted on Azure AI Foundry using the `azure-ai-agents` SDK.

## 安装

```bash
pip install azure-ai-agents azure-identity
# Or with azure-ai-projects for additional features
pip install azure-ai-projects azure-identity
```

## 环境 Variables

```bash
PROJECT_ENDPOINT="https://<resource>.services.ai.azure.com/api/projects/<project>"
MODEL_DEPLOYMENT_NAME="gpt-4o-mini"
```

## 认证

```python
from azure.identity import DefaultAzureCredential
from azure.ai.agents import AgentsClient

credential = DefaultAzureCredential()
client = AgentsClient(
    endpoint=os.environ["PROJECT_ENDPOINT"],
    credential=credential,
)
```

## Core 工作流

The basic 代理 lifecycle: **创建 代理 → 创建 线程 → 创建 message → 创建 运行 → 获取 响应**

### Minimal 示例

```python
import os
from azure.identity import DefaultAzureCredential
from azure.ai.agents import AgentsClient

client = AgentsClient(
    endpoint=os.environ["PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
)

# 1. Create agent
agent = client.create_agent(
    model=os.environ["MODEL_DEPLOYMENT_NAME"],
    name="my-agent",
    instructions="You are a helpful assistant.",
)

# 2. Create thread
thread = client.threads.create()

# 3. Add message
client.messages.create(
    thread_id=thread.id,
    role="user",
    content="Hello!",
)

# 4. Create and process run
run = client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)

# 5. Get response
if run.status == "completed":
    messages = client.messages.list(thread_id=thread.id)
    for msg in messages:
        if msg.role == "assistant":
            print(msg.content[0].text.value)

# Cleanup
client.delete_agent(agent.id)
```

## Tools 概述

| 工具 | 类 | Use Case |
|------|-------|----------|
| Code Interpreter | `CodeInterpreterTool` | 执行 Python, 生成 文件 |
| 文件 搜索 | `FileSearchTool` | RAG over uploaded documents |
| Bing Grounding | `BingGroundingTool` | Web 搜索 |
| Azure AI 搜索 | `AzureAISearchTool` | 搜索 your indexes |
| 函数 Calling | `FunctionTool` | Call your Python functions |
| OpenAPI | `OpenApiTool` | Call REST APIs |
| MCP | `McpTool` | 模型 Context 协议 servers |

See [参考/tools.md](参考/tools.md) for detailed patterns.

## Adding Tools

```python
from azure.ai.agents import CodeInterpreterTool, FileSearchTool

agent = client.create_agent(
    model=os.environ["MODEL_DEPLOYMENT_NAME"],
    name="tool-agent",
    instructions="You can execute code and search files.",
    tools=[CodeInterpreterTool()],
    tool_resources={"code_interpreter": {"file_ids": [file.id]}},
)
```

## 函数 Calling

```python
from azure.ai.agents import FunctionTool, ToolSet

def get_weather(location: str) -> str:
    """Get weather for a location."""
    return f"Weather in {location}: 72F, sunny"

functions = FunctionTool(functions=[get_weather])
toolset = ToolSet()
toolset.add(functions)

agent = client.create_agent(
    model=os.environ["MODEL_DEPLOYMENT_NAME"],
    name="function-agent",
    instructions="Help with weather queries.",
    toolset=toolset,
)

# Process run - toolset auto-executes functions
run = client.runs.create_and_process(
    thread_id=thread.id,
    agent_id=agent.id,
    toolset=toolset,  # Pass toolset for auto-execution
)
```

## 流式

```python
from azure.ai.agents import AgentEventHandler

class MyHandler(AgentEventHandler):
    def on_message_delta(self, delta):
        if delta.text:
            print(delta.text.value, end="", flush=True)

    def on_error(self, data):
        print(f"Error: {data}")

with client.runs.stream(
    thread_id=thread.id,
    agent_id=agent.id,
    event_handler=MyHandler(),
) as stream:
    stream.until_done()
```

See [参考/流式.md](参考/流式.md) for advanced patterns.

## 文件 Operations

### 上传 文件

```python
file = client.files.upload_and_poll(
    file_path="data.csv",
    purpose="assistants",
)
```

### 创建 向量 Store

```python
vector_store = client.vector_stores.create_and_poll(
    file_ids=[file.id],
    name="my-store",
)

agent = client.create_agent(
    model=os.environ["MODEL_DEPLOYMENT_NAME"],
    tools=[FileSearchTool()],
    tool_resources={"file_search": {"vector_store_ids": [vector_store.id]}},
)
```

## 异步 客户端

```python
from azure.ai.agents.aio import AgentsClient

async with AgentsClient(
    endpoint=os.environ["PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
) as client:
    agent = await client.create_agent(...)
    # ... async operations
```

See [参考/async-patterns.md](参考/asynasynasync-patterns 异步 patterns.

## 响应 格式

### JSON Mode

```python
agent = client.create_agent(
    model=os.environ["MODEL_DEPLOYMENT_NAME"],
    response_format={"type": "json_object"},
)
```

### JSON 模式

```python
agent = client.create_agent(
    model=os.environ["MODEL_DEPLOYMENT_NAME"],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "weather_response",
            "schema": {
                "type": "object",
                "properties": {
                    "temperature": {"type": "number"},
                    "conditions": {"type": "string"},
                },
                "required": ["temperature", "conditions"],
            },
        },
    },
)
```

## 线程 Management

### Continue Conversation

```python
# Save thread_id for later
thread_id = thread.id

# Resume later
client.messages.create(
    thread_id=thread_id,
    role="user",
    content="Follow-up question",
)
run = client.runs.create_and_process(thread_id=thread_id, agent_id=agent.id)
```

### 列表 Messages

```python
messages = client.messages.list(thread_id=thread.id, order="asc")
for msg in messages:
    role = msg.role
    content = msg.content[0].text.value
    print(f"{role}: {content}")
```

## Best Practices

1. **Use context managers** for 异步 客户端
2. **清理 up agents** when 已完成: `client.delete_agent(agent.id)`
3. **Use `create_and_process`** for simple cases, **流式** for real-time UX
4. **Pass toolset 迁移到 运行** for 自动 函数 execution
5. **Poll operations** use `*_and_poll` methods for long operations

## 参考 文件

- [参考/tools.md](参考/tools.md): All 工具 types with detailed 示例
- [参考/流式.md](参考/流式.md): 事件 handlers and 流式 patterns
- [参考/async-patterns.md](参考/asynasynasync-patterns 客户端 用法