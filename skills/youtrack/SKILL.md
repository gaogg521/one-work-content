---
name: youtrack
description: 通过 CLI 管理 YouTrack 事务、项目和工作流。用于创建、更新、搜索或评论 YouTrack 事务，列出项目，检查事务状态，或自动化事务工作流。
metadata:
  clawdbot:
    emoji: 🎫
    requires:
      bins:
      - jq
      - curl
tags:
- CLI
- 管理
---

# YouTrack CLI

使用 `ytctl`（位于 `scripts/` 中）进行 YouTrack 事务跟踪。

## 设置

凭据存储在 `~/.config/youtrack/config.json` 中：
```json
{
  "url": "https://your-instance.youtrack.cloud",
  "token": "perm:xxx"
}
```

或设置环境变量：`YOUTRACK_URL`、`YOUTRACK_TOKEN`

生成令牌：YouTrack → 个人资料 → 账户安全 → 新建令牌

## 命令

```bash
# 列出项目
ytctl projects

# 列出事务（带可选过滤器）
ytctl issues                           # 所有事务
ytctl issues SP                        # 项目 SP 中的事务
ytctl issues SP --query "state: Open"  # 带过滤条件
ytctl issues --max 50                  # 限制结果数量

# 获取事务详情
ytctl issue SP-123

# 创建事务
ytctl create SP "Bug: Login fails"
ytctl create SP "Feature request" "Detailed description here"

# 更新事务
ytctl update SP-123 state "In Progress"
ytctl update SP-123 assignee john.doe
ytctl update SP-123 priority Critical

# 添加评论
ytctl comment SP-123 "Investigating this now"

# 使用 YouTrack 查询语法搜索
ytctl search "project: SP state: Open assignee: me"
ytctl search "created: today"
ytctl search "#unresolved sort by: priority"

# 列出项目的工作流状态
ytctl states SP

# 列出用户
ytctl users
ytctl users --query "john"
```

## 查询语法

YouTrack 查询示例：
- `state: Open` — 按状态
- `assignee: me` — 分配给当前用户
- `created: today` — 今天创建的
- `updated: {last week}` — 上周更新的
- `#unresolved` — 所有未解决的
- `has: attachments` — 带附件的
- `sort by: priority desc` — 按优先级排序

组合：`project: SP state: Open assignee: me sort by: updated`

## 输出

默认：表格格式。添加 `--json` 获取原始 JSON 输出：
```bash
ytctl issues SP --json
ytctl issue SP-123  # 单个事务始终为 JSON
```

## 批量操作

```bash
# 更新所有匹配的事务（带试运行预览）
ytctl bulk-update "project: SP state: Open" state "In Progress" --dry-run
ytctl bulk-update "project: SP state: Open" state "In Progress"

# 对所有匹配的事务添加评论
ytctl bulk-comment "project: SP state: Open" "Batch update notice"

# 分配所有匹配的事务
ytctl bulk-assign "project: SP #unresolved" john.doe --dry-run
```

## 报告

```bash
# 项目摘要（默认 7 天）
ytctl report SP
ytctl report SP --days 14

# 用户活动报告
ytctl report-user zain
ytctl report-user zain --days 30

# 带条形图的状态分布
ytctl report-states SP
```

## 注意事项

- 项目可以是短名称（SP）或全名
- 字段：state、summary、description、assignee、priority
- 使用 `ytctl states PROJECT` 查看有效的状态名称
- 批量操作支持 `--dry-run` 以在执行前预览
