---
name: n8n-builder
description: 专业的 n8n 工作流(workflow)构建器，通过 n8n REST API 以编程方式创建、部署和管理 workflow。支持 webhook 流(flow)、定时任务(scheduled task)、AI 代理(agent)、数据库同步(database sync)和错误处理(error handling)。触发词：n8n 工作流(workflow)、自动化(automation)、webhook、AI 代理(agent)、数据库同步(database sync)
tags:
- AI
- API
- 数据库
- 自动化
---

# n8n Workflow Builder

## Setup

需要两个环境变量：
- `N8N_URL` — n8n 实例 URL (例如 `https://your-n8n.example.com`)
- `N8N_API_KEY` — n8n API key (Settings → API → Create API Key)

## Workflow

1. **理解自动化** — 明确触发器 (webhook/schedule/manual)、数据源、处理逻辑、输出和错误处理需求。

2. **设计 workflow JSON** — 构建有效的 n8n workflow JSON，遵循 `references/workflow-schema.md` 中的 schema。使用 `references/workflow-patterns.md` 中的模式作为模板。

3. **通过 API 部署** — 使用 `scripts/n8n-api.sh create <file>` 或将 JSON 通过管道传递给 `scripts/n8n-api.sh create-stdin`。

4. **激活** — 对于基于触发器的 workflow，使用 `scripts/n8n-api.sh activate <workflow_id>`。

5. **验证** — 列出 workflow 以确认部署：`scripts/n8n-api.sh list`。

## API Script Reference

```bash
# List all workflows
scripts/n8n-api.sh list

# Create workflow from JSON file
scripts/n8n-api.sh create /tmp/workflow.json

# Create from stdin
echo '{"name":"Test",...}' | scripts/n8n-api.sh create-stdin

# Get, activate, deactivate, delete, execute
scripts/n8n-api.sh get <id>
scripts/n8n-api.sh activate <id>
scripts/n8n-api.sh deactivate <id>
scripts/n8n-api.sh delete <id>
scripts/n8n-api.sh execute <id>

# List credentials and tags
scripts/n8n-api.sh credentials
scripts/n8n-api.sh tags
```

## Building Workflow JSON

每个 workflow 需要：`name`、`nodes[]`、`connections{}`、`settings{}`。

每个 node 需要：`id`、`name`、`type`、`typeVersion`、`position`、`parameters`。

Connections 使用 **source node display name** 作为 key，将输出映射到目标 node。

完整的 schema、node types 和 expression syntax → 阅读 `references/workflow-schema.md`
完整的 workflow 示例 (webhook、schedule、AI agent、DB sync、error handling) → 阅读 `references/workflow-patterns.md`

## Key Rules

- **始终设置 `"executionOrder": "v1"`** 在 settings 中
- **Node names 必须在 workflow 中唯一**
- **Node IDs 必须唯一** — 使用描述性 slugs 如 `webhook1`、`code1`
- **定位 nodes** 从 `[250, 300]` 开始，水平间距约 200px
- **IF nodes** 有两个输出：index 0 = true，index 1 = false
- **Webhook workflows** 如果 `responseMode` 是 `responseNode`，需要 `respondToWebhook` node
- **Credentials** 必须在激活前存在于 n8n 中 — 使用 `scripts/n8n-api.sh credentials` 检查
- **激活前测试** — 使用 `scripts/n8n-api.sh execute <id>` 进行 manual trigger workflows 测试
- **在危险的 HTTP/API nodes 上使用 `continueOnFail: true`**，然后在下游检查错误

## Common Real Estate Workflows

- **Lead intake**: Webhook → validate → dedupe → insert DB → notify Slack/SMS
- **Call follow-up**: Schedule → query DB for completed calls → send SMS/email based on outcome
- **Drip campaign**: Schedule → query leads by stage → send stage-appropriate email/SMS
- **CRM sync**: Webhook → transform → update HubSpot/Salesforce + internal DB
- **Property alerts**: Schedule → scrape/API listings → filter new → notify leads
- **AI qualification**: Webhook → AI Agent (classify lead intent) → route to appropriate pipeline
