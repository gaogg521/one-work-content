---
name: amplitude-automation
description: 通过 Rube MCP (Composio) 自动化 Amplitude 任务：events、user activity、cohorts、user identification。始终先搜索工具以获取当前 schema。
requires: None
mcp:
- rube
tags:
- Schema
- 自动化
---

# Amplitude Automation via Rube MCP

通过 Composio 的 Amplitude toolkit 经由 Rube MCP 自动化 Amplitude 产品分析。

## 前置条件

- Rube MCP 必须已连接（RUBE_SEARCH_TOOLS 可用）
- 通过 `RUBE_MANAGE_CONNECTIONS` 使用 toolkit `amplitude` 建立活跃的 Amplitude 连接
- 始终先调用 `RUBE_SEARCH_TOOLS` 以获取当前 tool schema

## 设置

**获取 Rube MCP**：在你的 client 配置中将 `https://rube.app/mcp` 添加为 MCP server。无需 API key —— 只需添加 endpoint 即可工作。

1. 通过确认 `RUBE_SEARCH_TOOLS` 响应来验证 Rube MCP 是否可用
2. 使用 toolkit `amplitude` 调用 `RUBE_MANAGE_CONNECTIONS`
3. 如果连接不是 ACTIVE，按照返回的 auth link 完成 Amplitude 认证
4. 在运行任何工作流前确认连接状态显示为 ACTIVE

## 核心工作流

### 1. 发送 Events

**何时使用**：用户想要跟踪 events 或发送 event 数据到 Amplitude

**Tool sequence**：
1. `AMPLITUDE_SEND_EVENTS` - 发送一个或多个 events 到 Amplitude [Required]

**Key parameters**：
- `events`：event objects 数组，每个包含：
  - `event_type`：event 名称（例如 'page_view'、'purchase'）
  - `user_id`：唯一用户标识符（如果没有 `device_id` 则必填）
  - `device_id`：设备标识符（如果没有 `user_id` 则必填）
  - `event_properties`：自定义 event properties 的对象
  - `user_properties`：要设置的 user properties 对象
  - `time`：event 时间戳，以毫秒为单位的 epoch 时间

**Pitfalls**：
- 每个 event 必须至少提供 `user_id` 或 `device_id` 之一
- `event_type` 每个 event 必填；不能为空
- `time` 必须是毫秒（13 位 epoch），不是秒
- 存在 batch limit；检查 schema 了解每次请求的最大 events 数
- Events 是异步处理的；成功的 API 响应不代表数据立即可查询

### 2. 获取 User Activity

**何时使用**：用户想要查看特定用户的 event 历史

**Tool sequence**：
1. `AMPLITUDE_FIND_USER` - 通过 ID 或 property 查找用户 [Prerequisite]
2. `AMPLITUDE_GET_USER_ACTIVITY` - 检索用户的 event stream [Required]

**Key parameters**：
- `user`：Amplitude 内部 user ID（来自 FIND_USER）
- `offset`：event 列表的分页 offset
- `limit`：返回的最大 event 数

**Pitfalls**：
- `user` 参数需要 Amplitude 的内部 user ID，不是你的 application 的 user_id
- 必须先调用 FIND_USER 才能将你的 user_id 解析为 Amplitude 的内部 ID
- Activity 默认按时间倒序返回
- 大量 activity 历史需要通过 `offset` 分页

### 3. 查找并识别用户

**何时使用**：用户想要查找用户或设置 user properties

**Tool sequence**：
1. `AMPLITUDE_FIND_USER` - 通过各种标识符搜索用户 [Required]
2. `AMPLITUDE_IDENTIFY` - 设置或更新 user properties [Optional]

**Key parameters**：
- 对于 FIND_USER：
  - `user`：搜索词（user_id、email 或 Amplitude ID）
- 对于 IDENTIFY：
  - `user_id`：你的 application 的用户标识符
  - `device_id`：设备标识符（user_id 的替代）
  - `user_properties`：包含 `$set`、`$unset`、`$add`、`$append` 操作的对象

**Pitfalls**：
- FIND_USER 跨 user_id、device_id 和 Amplitude ID 搜索
- IDENTIFY 使用特殊的 property 操作（`$set`、`$unset`、`$add`、`$append`）
- `$set` 会覆盖已有值；`$setOnce` 仅在尚未设置时才设置
- IDENTIFY 必须至少提供 `user_id` 或 `device_id` 之一
- User property 变更最终一致；非即时生效

### 4. 管理 Cohorts

**何时使用**：用户想要列出 cohorts、查看 cohort 详情或更新 cohort 成员

**Tool sequence**：
1. `AMPLITUDE_LIST_COHORTS` - 列出所有已保存的 cohorts [Required]
2. `AMPLITUDE_GET_COHORT` - 获取详细的 cohort 信息 [Optional]
3. `AMPLITUDE_UPDATE_COHORT_MEMBERSHIP` - 向 cohort 添加/移除用户 [Optional]
4. `AMPLITUDE_CHECK_COHORT_STATUS` - 检查异步 cohort 操作状态 [Optional]

**Key parameters**：
- LIST_COHORTS：无必需参数
- GET_COHORT：`cohort_id`（来自列表结果）
- UPDATE_COHORT_MEMBERSHIP：
  - `cohort_id`：目标 cohort ID
  - `memberships`：包含 `add` 和/或 `remove` 用户 ID 数组的对象
- CHECK_COHORT_STATUS：来自 update 响应的 `request_id`

**Pitfalls**：
- 所有 cohort-specific 操作都需要 Cohort IDs
- UPDATE_COHORT_MEMBERSHIP 是异步的；使用 CHECK_COHORT_STATUS 验证
- 状态检查需要 update 响应中的 `request_id`
- 每次请求的最大成员变更数可能有限制；大块更新请分片
- 只有行为 cohorts 支持 API 成员更新

### 5. 浏览 Event Categories

**何时使用**：用户想要发现 Amplitude 中可用的 event types 和 categories

**Tool sequence**：
1. `AMPLITUDE_GET_EVENT_CATEGORIES` - 列出所有 event categories [Required]

**Key parameters**：
- 无必需参数；返回所有已配置的 event categories

**Pitfalls**：
- Categories 在 Amplitude UI 中配置；API 提供只读访问
- Category 内的 event names 区分大小写
- 使用这些 categories 在发送 events 前验证 event_type 值

## 常见模式

### ID Resolution

**Application user_id -> Amplitude internal ID**：
```
1. 使用 user=your_user_id 调用 AMPLITUDE_FIND_USER
2. 从响应中提取 Amplitude 的内部 user ID
3. 将内部 ID 用于 GET_USER_ACTIVITY
```

**Cohort name -> Cohort ID**：
```
1. 调用 AMPLITUDE_LIST_COHORTS
2. 在结果中按名称查找 cohort
3. 提取 id 用于 cohort 操作
```

### User Property Operations

Amplitude IDENTIFY 支持以下 property 操作：
- `$set`：设置 property 值（覆盖已有）
- `$setOnce`：仅在 property 尚未设置时才设置
- `$add`：递增数值 property
- `$append`：追加到列表 property
- `$unset`：完全移除 property

示例结构：
```json
{
  "user_properties": {
    "$set": {"plan": "premium", "company": "Acme"},
    "$add": {"login_count": 1}
  }
}
```

### 异步操作模式

对于 cohort 成员更新：
```
1. 调用 AMPLITUDE_UPDATE_COHORT_MEMBERSHIP -> 获取 request_id
2. 使用 request_id 调用 AMPLITUDE_CHECK_COHORT_STATUS
3. 重复步骤 2 直到状态为 'complete' 或 'error'
```

## 已知 Pitfalls

**User IDs**：
- Amplitude 有自己的内部 user IDs，与你的 application 的分开
- FIND_USER 将你的 ID 解析为 Amplitude 的内部 ID
- GET_USER_ACTIVITY 需要 Amplitude 的内部 ID，不是你的 user_id

**Event Timestamps**：
- 必须是自 epoch 起的毫秒（13 位）
- 秒（10 位）会被解释为非常旧的日期
- 省略时间戳则使用服务器接收时间

**Rate Limits**：
- Event ingestion 每个 project 有吞吐量限制
- 尽可能批量发送 events 以减少 API 调用
- Cohort 成员更新有异步处理限制

**Response Parsing**：
- 响应数据可能嵌套在 `data` key 下
- User activity 按时间倒序返回 events
- Cohort 列表可能包含已归档的 cohorts；检查 status 字段
- 对可选字段进行防御性解析并设置 fallback

## 快速参考

| Task | Tool Slug | Key Params |
|------|-----------|------------|
| Send events | AMPLITUDE_SEND_EVENTS | events (array) |
| Find user | AMPLITUDE_FIND_USER | user |
| Get user activity | AMPLITUDE_GET_USER_ACTIVITY | user, offset, limit |
| Identify user | AMPLITUDE_IDENTIFY | user_id, user_properties |
| List cohorts | AMPLITUDE_LIST_COHORTS | (none) |
| Get cohort | AMPLITUDE_GET_COHORT | cohort_id |
| Update cohort members | AMPLITUDE_UPDATE_COHORT_MEMBERSHIP | cohort_id, memberships |
| Check cohort status | AMPLITUDE_CHECK_COHORT_STATUS | request_id |
| List event categories | AMPLITUDE_GET_EVENT_CATEGORIES | (none) |
