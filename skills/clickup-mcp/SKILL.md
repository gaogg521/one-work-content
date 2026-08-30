---
name: clickup-mcp
description: 通过官方 MCP 管理 ClickUp 任务、文档、时间跟踪、评论、聊天和搜索。需要 OAuth 认证。
homepage: https://clickup.com
metadata: None
clawdbot: None
emoji: ✅
requires: None
bins:
- mcporter
env:
- CLICKUP_TOKEN
tags:
- ClickUp
---

# ClickUp MCP (官方)

通过官方 MCP 服务器访问 ClickUp。完整的工作区搜索、任务管理、时间跟踪、评论、聊天和文档。

## 设置

### 选项 1：直接 OAuth（仅支持的客户端）

ClickUp MCP 仅允许来自**白名单客户端**的 OAuth：
- Claude Desktop、Claude Code、Cursor、VS Code、Windsurf、ChatGPT

```bash
# Claude Code
claude mcp add clickup --transport http https://mcp.clickup.com/mcp
# 然后在会话中运行 /mcp 以授权
```

### 选项 2：Claude Code → mcporter（推荐）

使用 Claude Code 进行 OAuth，然后提取令牌供 mcporter 使用：

**步骤 1：通过 Claude Code 授权**
```bash
claude mcp add clickup --transport http https://mcp.clickup.com/mcp
claude
# 在 Claude Code 中，运行：/mcp
# 在浏览器中完成 OAuth
```

**步骤 2：提取令牌**
```bash
jq -r '.mcpOAuth | to_entries | .[] | select(.key | startswith("clickup")) | .value.accessToken' ~/.claude/.credentials.json
```

**步骤 3：添加到环境**
```bash
# 添加到 ~/.clawdbot/.env
CLICKUP_TOKEN=eyJhbGciOiJkaXIi...
```

**步骤 4：配置 mcporter**

添加到 `config/mcporter.json`：
```json
{
  "mcpServers": {
    "clickup": {
      "baseUrl": "https://mcp.clickup.com/mcp",
      "description": "Official ClickUp MCP",
      "headers": {
        "Authorization": "Bearer ${CLICKUP_TOKEN}"
      }
    }
  }
}
```

**步骤 5：测试**
```bash
mcporter list clickup
mcporter call 'clickup.clickup_search(keywords: "test", count: 3)'
```

### 令牌刷新

令牌是长期有效的（约 10 年）。如果过期：
1. 在 Claude Code 中重新运行 `/mcp`
2. 从 `~/.claude/.credentials.json` 重新提取令牌
3. 在 `.env` 中更新 `CLICKUP_TOKEN`

## 可用工具 (32)

### 搜索

| 工具 | 描述 |
|------|-------------|
| `clickup_search` | 跨任务、文档、仪表板、聊天、文件的通用搜索 |

### 任务

| 工具 | 描述 |
|------|-------------|
| `clickup_create_task` | 创建任务，包括名称、描述、状态、分配人、截止日期、优先级 |
| `clickup_get_task` | 获取任务详情（可选子任务） |
| `clickup_update_task` | 更新任何任务字段 |
| `clickup_attach_task_file` | 将文件附加到任务（URL 或 base64） |
| `clickup_add_tag_to_task` | 向任务添加标签 |
| `clickup_remove_tag_from_task` | 从任务中移除标签 |

### 评论

| 工具 | 描述 |
|------|-------------|
| `clickup_get_task_comments` | 获取任务上的所有评论 |
| `clickup_create_task_comment` | 添加评论（支持 @提及） |

### 时间跟踪

| 工具 | 描述 |
|------|-------------|
| `clickup_start_time_tracking` | 在任务上启动计时器 |
| `clickup_stop_time_tracking` | 停止活动计时器 |
| `clickup_add_time_entry` | 手动记录时间 |
| `clickup_get_task_time_entries` | 获取任务的时间条目 |
| `clickup_get_current_time_entry` | 检查活动计时器 |

### 工作区与层级

| 工具 | 描述 |
|------|-------------|
| `clickup_get_workspace_hierarchy` | 获取完整结构（Spaces、Folders、Lists） |
| `clickup_create_list` | 在 Space 中创建列表 |
| `clickup_create_list_in_folder` | 在 Folder 中创建列表 |
| `clickup_get_list` | 获取列表详情 |
| `clickup_update_list` | 更新列表设置 |
| `clickup_create_folder` | 在 Space 中创建文件夹 |
| `clickup_get_folder` | 获取文件夹详情 |
| `clickup_update_folder` | 更新文件夹设置 |

### 成员

| 工具 | 描述 |
|------|-------------|
| `clickup_get_workspace_members` | 列出所有工作区成员 |
| `clickup_find_member_by_name` | 按名称/电子邮件查找成员 |
| `clickup_resolve_assignees` | 从名称获取用户 ID |

### 聊天

| 工具 | 描述 |
|------|-------------|
| `clickup_get_chat_channels` | 列出所有聊天频道 |
| `clickup_send_chat_message` | 向频道发送消息 |

### 文档

| 工具 | 描述 |
|------|-------------|
| `clickup_create_document` | 创建新文档 |
| `clickup_list_document_pages` | 获取文档结构 |
| `clickup_get_document_pages` | 获取页面内容 |
| `clickup_create_document_page` | 向文档添加页面 |
| `clickup_update_document_page` | 编辑页面内容 |

## 使用示例

### 搜索工作区

```bash
mcporter call 'clickup.clickup_search(
  keywords: "Q4 marketing",
  count: 10
)'
```

### 创建任务

```bash
mcporter call 'clickup.clickup_create_task(
  name: "Review PR #42",
  list_id: "901506994423",
  description: "Check the new feature",
  status: "to do"
)'
```

### 更新任务

```bash
mcporter call 'clickup.clickup_update_task(
  task_id: "abc123",
  status: "in progress"
)'
```

### 添加评论

```bash
mcporter call 'clickup.clickup_create_task_comment(
  task_id: "abc123",
  comment_text: "@Mark can you review this?"
)'
```

### 时间跟踪

```bash
# 启动计时器
mcporter call 'clickup.clickup_start_time_tracking(
  task_id: "abc123",
  description: "Working on feature"
)'

# 停止计时器
mcporter call 'clickup.clickup_stop_time_tracking()'

# 手动记录时间（持续时间以毫秒为单位，例如，2h = 7200000）
mcporter call 'clickup.clickup_add_time_entry(
  task_id: "abc123",
  start: "2026-01-06 10:00",
  duration: "2h",
  description: "Code review"
)'
```

### 获取工作区结构

```bash
mcporter call 'clickup.clickup_get_workspace_hierarchy(limit: 10)'
```

### 聊天

```bash
# 列出频道
mcporter call 'clickup.clickup_get_chat_channels()'

# 发送消息
mcporter call 'clickup.clickup_send_chat_message(
  channel_id: "channel-123",
  content: "Team standup in 5 minutes!"
)'
```

## 限制

- **无删除操作** — 安全措施；使用 ClickUp UI
- **无自定义字段** — 未在官方 MCP 中公开
- **无视图管理** — 不可用
- **需要 OAuth** — 必须使用白名单客户端（提供 Claude Code 变通方法）
- **速率限制** — 与 ClickUp API 相同（约 100 请求/分钟）

## 资源

- [ClickUp MCP 文档](https://developer.clickup.com/docs/connect-an-ai-assistant-to-clickups-mcp-server)
- [支持的工具](https://developer.clickup.com/docs/mcp-tools)
- [ClickUp API 参考](https://clickup.com/api)
- [反馈 / 白名单请求](https://feedback.clickup.com)
