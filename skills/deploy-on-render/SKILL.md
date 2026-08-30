---
name: deploy-on-render
description: 在 Render 上部署和运行应用（Blueprint + 一键 Dashboard deeplink，与 Codex render-deploy 流程相同）。当用户想要部署、托管或发布应用；创建或编辑 render.yaml；添加 web、static、workers、cron、Postgres 或 Key Value；获取 Blueprint deeplink 进行部署；在设置了 RENDER_API_KEY 时通过 API 触发或验证部署；通过 mcporter 连接 Render MCP 进行直接服务创建；或配置 env vars、health checks、scaling、previews 和 projects 时使用。
---

# Render 技能

使用 Blueprints (`render.yaml`)、Dashboard 或 API 在 Render 上部署和管理应用。此技能镜像 **Codex render-deploy** 流程：分析代码库 → 生成/验证 Blueprint → 提交并推送 → 一键 Dashboard deeplink → 可选的 API/mcporter 验证或重新部署。

## 何时使用此技能

当用户想要以下操作时激活此技能：
- 在 Render 上部署、托管或发布应用
- 创建或编辑 `render.yaml` Blueprint（新仓库或现有仓库）
- 添加 web、static site、private、worker 或 cron 服务；Postgres；或 Key Value
- 配置 env vars、health checks、scaling、disks 或 regions
- 设置 preview environments 或 projects
- 验证 Blueprint 或获取 Dashboard/API 链接

## 部署方式选择

1. **如果 `RENDER_API_KEY` 已设置** → 优先使用 REST API 或 MCP（最快；无需用户点击）。使用 `references/rest-api-deployment.md` 获取请求体，或如果已配置则使用 mcporter（参见 `references/mcp-integration.md`）。
2. **如果没有 API key** → 使用 Blueprint + deeplink（用户提交、推送，然后点击 deeplink 进行部署）。

检查 API key：

```bash
[ -n "$RENDER_API_KEY" ] && echo "RENDER_API_KEY is set" || echo "RENDER_API_KEY is not set"
```

## 快捷路径（新用户）

在进行深入分析之前，使用以下简短序列来减少摩擦：
1. 询问用户是否想要从 **Git repo**（Blueprint 和 deeplink 所需）进行部署，或仅获取指导。如果没有 Git remote，他们必须先创建并推送一个。
2. 询问应用是否需要 database、workers、cron 或其他服务，以便你可以选择正确的 Blueprint 结构。

然后按照下面的 **部署到 Render** 进行操作（Blueprint → push → deeplink → verify）。

## 前置条件检查

1. **Git remote** – Blueprint 部署所需。运行 `git remote -v`；如果没有，请要求用户在 GitHub/GitLab/Bitbucket 上创建仓库，添加 `origin`，并推送。
2. **Render CLI (optional)** – 用于本地验证：`render blueprints validate render.yaml`。安装：`brew install render` 或 [Render CLI](https://github.com/render-oss/cli)。
3. **API key (optional)** – 用于验证部署或触发重新部署：[Dashboard → API Keys](https://dashboard.render.com/u/*/settings#api-keys)。在环境中设置 `RENDER_API_KEY`。

## 安全注意事项

- **永远不要将 secrets 提交到 render.yaml** —— 始终对 API keys、passwords 和 tokens 使用 `sync: false`；用户在 Dashboard 中填写它们。
- **在建议部署之前验证** —— 运行 `render blueprints validate render.yaml` 或使用 Validate Blueprint API，以确保不会推送无效的 YAML。
- **验证用户提供的值** —— 当将用户输入的 env vars 或 service names 写入 YAML 时，根据需要清理或引用以避免注入。

## 参考文档

- `references/codebase-analysis.md` (检测 runtime、build/start 命令、env vars)
- `references/blueprint-spec.md` (root keys、service types、env vars、validation)
- `references/rest-api-deployment.md` (直接 API 创建服务：ownerId、request bodies、type mapping)
- `references/mcp-integration.md` (Render MCP tools、mcporter 用法、支持的 runtimes/plans/regions)
- `references/post-deploy-checks.md` (通过 API 验证部署状态和 health)
- `references/troubleshooting-basics.md` (build/startup/runtime 故障)
- `assets/` (示例 Blueprints：node-express.yaml、python-web.yaml、static-site.yaml、web-with-postgres.yaml)

## Blueprint 基础

- **文件：** `render.yaml` 位于 Git 仓库的 **根目录**（必需）。
- **Root-level keys (official spec)：** `services`、`databases`、`envVarGroups`、`projects`、`ungrouped`、`previews.generation`、`previews.expireAfterDays`。
- **Spec：** [Blueprint YAML Reference](https://render.com/docs/blueprint-spec)。用于 IDE 验证的 JSON Schema：https://render.com/schema/render.yaml.json（例如 VS Code/Cursor 中的 Red Hat YAML 扩展）。

**验证：** `render blueprints validate render.yaml` (Render CLI v2.7.0+)，或 [Validate Blueprint API](https://api-docs.render.com/reference/validate-blueprint) 端点。

## 服务类型

| type       | Purpose |
|------------|--------|
| `web`      | Public HTTP app 或 static site (static 使用 `runtime: static`) |
| `pserv`    | Private service (仅内部 hostname，无 public URL) |
| `worker`   | Background worker (持续运行，例如 job queues) |
| `cron`     | Scheduled job (cron expression；运行后退出) |
| `keyvalue` | Render Key Value instance (Redis/Valkey-compatible；**在 `services` 中定义**) |

**注意：** Private services 使用 `pserv`，不是 `private`。Key Value 是 `type: keyvalue` 的服务；不要在新 Blueprints 中使用单独的 root key（一些旧 blueprints 使用 `keyValueStores` 和 `fromKeyValueStore`——优先使用官方格式）。

## Runtimes

使用 **`runtime`** (优先；`env` 已弃用)：`node`、`python`、`elixir`、`go`、`ruby`、`rust`、`docker`、`image`、`static`。对于 static sites：`type: web`、`runtime: static`，并且需要 **`staticPublishPath`**（例如 `./build` 或 `./dist`）。

## 最小 Web 服务

```yaml
services:
  - type: web
    name: my-app
    runtime: node
    buildCommand: npm install
    startCommand: npm start
    envVars:
      - key: NODE_ENV
        value: production
```

Python 示例：`runtime: python`、`buildCommand: pip install -r requirements.txt`、`startCommand: uvicorn app.main:app --host 0.0.0.0 --port $PORT`（或 gunicorn）。在需要时通过 envVars 设置 `PYTHON_VERSION` / `NODE_VERSION`。

## 静态站点

```yaml
- type: web
  name: my-blog
  runtime: static
  buildCommand: yarn build
  staticPublishPath: ./build
```

可选：`headers`、`routes` (redirects/rewrites)。参见 [Static Sites](https://render.com/docs/static-sites)。

## 环境变量

- **Literal：** `key` + `value` (永远不要硬编码 secrets)。
- **From Postgres：** `fromDatabase.name` + `fromDatabase.property` (例如 `connectionString`)。
- **From Key Value 或其他 service：** `fromService.type` + `fromService.name` + `fromService.property` (例如 `connectionString`、`host`、`port`、`hostport`) 或 `fromService.envVarKey` 用于另一个 service 的 env var。
- **Secret / user-set：** `sync: false` (用户在首次创建时在 Dashboard 中提示；之后手动添加新 secrets)。**不能在 env var groups 中使用。**
- **Generated：** `generateValue: true` (base64 256-bit value)。
- **Shared：** `fromGroup: <envVarGroups[].name>` 用于附加 env var group。

Env groups **不能**引用其他 services（groups 中无 `fromDatabase`/`fromService`）并且 **不能**使用 `sync: false`。将 secrets 和 DB/KV 引用放在 **service-level** 的 `envVars` 中，或引用一个 group 并在旁边添加 service-specific 的 vars。

## 数据库 (Render Postgres)

```yaml
databases:
  - name: my-db
    plan: basic-256mb
    databaseName: my_app
    user: my_user
    region: oregon
    postgresMajorVersion: "18"
```

**Plans (current)：** `free`、`basic-256mb`、`basic-1gb`、`basic-4gb`、`pro-*`、`accelerated-*`。Legacy：`starter`、`standard`、`pro`、`pro plus` (legacy 上不再新建 DB)。可选：`diskSizeGB`、`ipAllowList`、`readReplicas`、`highAvailability.enabled`。在 services 中引用：`fromDatabase.name`、`property: connectionString`。

## Key Value (Redis/Valkey)

Key Value instances 是 `type: keyvalue` 的 **services**（或已弃用的 `redis`）。**`ipAllowList` 是必需的：** 使用 `[]` 表示仅内部访问，或 `- source: 0.0.0.0/0` 允许外部访问。

```yaml
services:
  - type: keyvalue
    name: my-cache
    ipAllowList:
      - source: 0.0.0.0/0
        description: everywhere
    plan: free
    maxmemoryPolicy: allkeys-lru
```

在另一个 service 中引用：`fromService.type: keyvalue`、`fromService.name: my-cache`、`property: connectionString`。Policies：`allkeys-lru` (caching)、`noeviction` (job queues) 等。参见 [Key Value](https://render.com/docs/key-value)。

**注意：** 一些仓库使用 root-level 的 `keyValueStores` 和 `fromKeyValueStore`；官方 spec 使用 `services` + `fromService`。为新 Blueprints 优先使用官方形式。

## Cron Jobs

```yaml
- type: cron
  name: my-cron
  runtime: python
  schedule: "0 * * * *"
  buildCommand: "true"
  startCommand: python scripts/daily.py
  envVars: []
```

`schedule` 是 cron expression (minute hour day month weekday)。`buildCommand` 是必需的（如果没有 build 则使用 `"true"`）。Free plan 不适用于 cron/worker/pserv。

## Env Var Groups

跨 services 共享 vars。Groups 内部不能使用 `fromDatabase`/`fromService`/`sync: false`——只允许 literal values 或 `generateValue: true`。

```yaml
envVarGroups:
  - name: app-env
    envVars:
      - key: CONCURRENCY
        value: "2"
      - key: APP_SECRET
        generateValue: true

services:
  - type: web
    name: api
    envVars:
      - fromGroup: app-env
      - key: DATABASE_URL
        fromDatabase:
          name: my-db
          property: connectionString
```

## Health Check、Region、Pre-deploy

- **Web only：** `healthCheckPath: /health` 用于 zero-downtime deploys。
- **Region：** `region: oregon` (default)、`ohio`、`virginia`、`frankfurt`、`singapore` (在创建时设置；之后不能更改)。
- **Pre-deploy：** `preDeployCommand` 在 build 之后、start 之前运行（例如 migrations）。

## 扩缩容

- **Manual：** `numInstances: 2`。
- **Autoscaling** (Professional workspace)：`scaling.minInstances`、`scaling.maxInstances`、`scaling.targetCPUPercent` 或 `scaling.targetMemoryPercent`。不能与 persistent disks 一起使用。

## Disks、Monorepos、Docker

- **Persistent disk：** `disk.name`、`disk.mountPath`、`disk.sizeGB` (web、pserv、worker)。
- **Monorepo：** `rootDir`、`buildFilter.paths` / `buildFilter.ignoredPaths`、`dockerfilePath` / `dockerContext`。
- **Docker：** `runtime: docker` (从 Dockerfile build) 或 `runtime: image` (从 registry pull)。在需要时使用 `dockerCommand` 代替 `startCommand`。

## Preview Environments & Projects

- **Preview environments：** Root-level `previews.generation: off | manual | automatic`，可选 `previews.expireAfterDays`。Per-service `previews.generation`、`previews.numInstances`、`previews.plan`。
- **Projects/environments：** Root-level `projects` 包含 `environments`（每个列出 `services`、`databases`、`envVarGroups`）。用于 staging/production。可选 `ungrouped` 用于不在任何 environment 中的资源。

## 常见部署模式

### 全栈 (web + Postgres + Key Value)

Web service 使用 `fromDatabase` 连接 Postgres 和 `fromService` 连接 Key Value。添加一个 `databases` 条目和一个 `type: keyvalue` 的 service；从 web service 的 `envVars` 中引用两者。参见 `assets/web-with-postgres.yaml` 了解 Postgres；添加 keyvalue service 和 `fromService` 用于 Redis URL。

### 微服务 (API + worker + cron)

一个 Blueprint 中的多个 services：`type: web` 用于 API，`type: worker` 用于 background processor，`type: cron` 用于 scheduled jobs。共享 `envVarGroups` 或重复 env vars；使用 `fromDatabase`/`fromService` 用于共享 DB/Redis。所有 services 使用相同的 `branch`，并根据 runtime 使用适当的 `buildCommand`/`startCommand`。

### PR 的 Preview environments

设置 root-level `previews.generation: automatic` (或 `manual`)。可选 `previews.expireAfterDays: 7`。每个 PR 获得一个 preview URL；在需要时使用 per-service overrides 设置 `previews.generation`、`previews.numInstances` 或 `previews.plan`。

## Plans (Services)

`plan: free | starter | standard | pro | pro plus` (以及用于 web/pserv/worker 的：`pro max`、`pro ultra`)。省略以保持现有设置或新服务默认使用 `starter`。Free 不适用于 pserv、worker、cron。

## Dashboard & API

- **Dashboard：** https://dashboard.render.com — New → Blueprint，连接 repo，选择 `render.yaml`。
- **Key Value：** https://dashboard.render.com/new/redis

## API 访问

要从 agent 使用 Render API（验证部署、触发部署、列出 services/logs）：

1. **获取 API key：** Dashboard → Account Settings → [API Keys](https://dashboard.render.com/u/*/settings#api-keys)。
2. **存储为 env var：** 在环境中设置 `RENDER_API_KEY`（例如 `skills.entries.render.env` 或 process env）。
3. **Authentication：** 使用 Bearer token：`Authorization: Bearer $RENDER_API_KEY` 用于所有请求。
4. **API docs：** https://api-docs.render.com — services、deploys、logs、validate Blueprint 等。

---

# 部署到 Render (与 Codex render-deploy 技能相同流程)

目标：通过生成 Blueprint 来部署应用，然后 **通过 Dashboard deeplink 一键部署**；当用户拥有 `RENDER_API_KEY` 时，可选地 **通过 API 触发或验证**。

## Step 1: 分析代码库并创建 render.yaml

- 使用 `references/codebase-analysis.md` 确定 runtime、build/start 命令、env vars 和 datastores。
- 在 repo root 添加或更新 `render.yaml`（参见上面的 Blueprint 部分和 `references/blueprint-spec.md`）。对 secrets 使用 `sync: false`。参见 `assets/` 获取示例。
- 在要求用户推送之前 **验证**：
  - CLI：`render blueprints validate render.yaml`（安装：`brew install render` 或 [Render CLI install](https://github.com/render-oss/cli)）。
  - 或 API：POST 到 [Validate Blueprint](https://api-docs.render.com/reference/validate-blueprint) 并携带 YAML body。
- 在继续之前修复任何验证错误。

## Step 2: 提交并推送 (required)

Render 从 **Git remote** 读取 Blueprint。文件必须被提交并推送。

```bash
git add render.yaml
git commit -m "Add Render deployment configuration"
git push origin main
```

如果没有 Git remote，停止并要求用户在 GitHub/GitLab/Bitbucket 上创建仓库，将其添加为 `origin`，并推送。没有推送的仓库，Dashboard deeplink 将无法工作。

## Step 3: Dashboard deeplink (one-click deploy)

获取 repo URL 并构建 Blueprint deeplink：

```bash
git remote get-url origin
```

如果 URL 是 **SSH**，转换为 **HTTPS**（Render 需要 HTTPS 用于 deeplink）：

| SSH | HTTPS |
|-----|--------|
| `git@github.com:user/repo.git` | `https://github.com/user/repo` |
| `git@gitlab.com:user/repo.git` | `https://gitlab.com/user/repo` |
| `git@bitbucket.org:user/repo.git` | `https://bitbucket.org/user/repo` |

模式：将 `git@<host>:` 替换为 `https://<host>/`，移除 `.git` 后缀。

**Deeplink format：**
```
https://dashboard.render.com/blueprint/new?repo=<REPO_HTTPS_URL>
```

示例：`https://dashboard.render.com/blueprint/new?repo=https://github.com/username/my-app`

给用户以下 checklist：

1. 确认 `render.yaml` 在 repo root 中（他们刚刚推送了它）。
2. **点击 deeplink** 打开 Render Dashboard。
3. 如果被提示，完成 Git provider OAuth。
4. 命名 Blueprint（或接受默认名称）。
5. **填写 secret env vars**（那些带有 `sync: false` 的）。
6. 查看 services/databases，然后点击 **Apply** 进行部署。

部署自动开始。用户可以在 Dashboard 中监控。

## Step 4: 验证部署 (optional, needs API key)

如果用户已设置 `RENDER_API_KEY`（例如在 `skills.entries.render.env` 或 process env 中），agent 可以在用户应用 Blueprint 后进行验证：

- **列出 services：** `GET https://api.render.com/v1/services` — Header: `Authorization: Bearer $RENDER_API_KEY`。按名称查找 service。
- **列出 deploys：** `GET https://api.render.com/v1/services/{serviceId}/deploys?limit=1` — 检查 `status: "live"` 以确认成功。
- **Logs (如果需要)：** Render API 或 Dashboard → service → Logs。

示例 (exec tool 或 curl)：
```bash
curl -s -H "Authorization: Bearer $RENDER_API_KEY" "https://api.render.com/v1/services" | head -100
curl -s -H "Authorization: Bearer $RENDER_API_KEY" "https://api.render.com/v1/services/{serviceId}/deploys?limit=1"
```

对于简短的 checklist 和常见修复，使用 `references/post-deploy-checks.md` 和 `references/troubleshooting-basics.md`。

## 触发部署 (re-deploy without push)

- **Repo 连接后：** 推送到 linked branch 会在 auto-deploy 开启时触发自动部署。
- **通过 API 触发：** 使用 `RENDER_API_KEY`，触发新的 deploy：
  - **POST** `https://api.render.com/v1/services/{serviceId}/deploys`
  - Header: `Authorization: Bearer $RENDER_API_KEY`
  - 可选 body：`{ "clearCache": "do_not_clear" }` 或 `"clear"`
- **Deploy hook (no API key)：** Dashboard → service → Settings → Deploy Hook。用户可以将该 URL 设置为 env var（例如 `RENDER_DEPLOY_HOOK_URL`）；然后 agent 可以运行 `curl -X POST "$RENDER_DEPLOY_HOOK_URL"` 来触发部署。

因此：**OpenClaw 可以通过** (1) 创建 `render.yaml`，(2) 让用户推送并点击 Blueprint deeplink（一键），以及可选地 (3) 在凭证可用时通过 API 或 deploy hook 触发或验证部署来部署。

## 从 OpenClaw 使用 Render (无原生 MCP)

OpenClaw 不会从配置加载 MCP servers。使用以下方式之一：

### Option A: REST API (推荐在已设置 API key 时使用)

使用 `RENDER_API_KEY` 和 Render REST API (curl/exec)：创建 services、列出 services、触发 deploys、列出 deploys、列出 logs。**请求体和端点：** `references/rest-api-deployment.md`。

### Option B: 通过 mcporter 使用 MCP (如果已安装)

如果用户已安装 **mcporter** 并配置了 Render（URL `https://mcp.render.com/mcp`，Bearer `$RENDER_API_KEY`），agent 可以直接调用 Render MCP tools。**工具列表和示例命令：** `references/mcp-integration.md`。

示例：

```bash
mcporter call render.list_services
mcporter call render.create_web_service name=my-api runtime=node buildCommand="npm ci" startCommand="npm start" repo=https://github.com/user/repo branch=main plan=free
```

必须首先设置 Workspace（例如用户：“Set my Render workspace to MyTeam”）。使用 `mcporter list render --schema` 查看当前工具和参数。

---

## 新部署检查清单

1. 使用 `services`（以及可选的 `databases`、`envVarGroups`、`projects`）添加或更新 `render.yaml`。使用 `runtime` 和官方 Key Value 形式（services 中的 `type: keyvalue`，引用使用 `fromService`）。
2. 仅在 **service** envVars 中对 secrets 使用 `sync: false`；告诉用户在 Dashboard 中设置它们。永远不要将 secrets 放在 env groups 中。
3. 对于 Key Value，设置 `ipAllowList`（必需）。
4. 验证：`render blueprints validate render.yaml` 或 API。
5. 用户必须 commit 并 push，然后使用 **Blueprint deeplink** (`https://dashboard.render.com/blueprint/new?repo=<HTTPS_REPO_URL>`) 进行部署。如果设置了 `RENDER_API_KEY`，可选地通过 API 验证或重新部署。

## 规则

- 优先使用 Blueprint 进行完整的应用定义；仅在 Blueprint 无法表达某些内容时建议使用 Dashboard/API。
- 永远不要提交真实的 API keys 或 secrets；使用 `sync: false` 并记录用户必须设置哪些 env vars。
- 使用 `runtime`（不是已弃用的 `env`）。对于 Python/Node，在需要时通过 envVars 设置 `PYTHON_VERSION`/`NODE_VERSION`。
- 引用 Key Value 或其他 services 时，使用 `fromService` 并指定正确的 `type`（例如 `keyvalue`、`pserv`）。
