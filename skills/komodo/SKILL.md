---
name: komodo
description: 管理 Komodo 基础设施(infrastructure)，包括服务器(server)、Docker 部署(deployment)、stacks、构建(build)和流程(procedure)。触发词：Komodo、基础设施管理(infrastructure management)、Docker 部署(deployment)、容器管理(container management)
tags:
- Docker
- 基础设施
- 部署
---

# Komodo Skill

通过 Komodo Core API 管理服务器、Docker 容器、stacks、builds 和 procedures。

## 前置条件

设置环境变量:
- `KOMODO_ADDRESS` - Komodo Core URL (例如, `https://komodo.example.com`)
- `KOMODO_API_KEY` - API key (以 `K-` 开头)
- `KOMODO_API_SECRET` - API secret (以 `S-` 开头)

## 快速参考

```bash
# 设置环境变量 (或从 credentials 文件 source)
export KOMODO_ADDRESS="https://komodo.weird.cyou"
export KOMODO_API_KEY="K-..."
export KOMODO_API_SECRET="S-..."

# 列出资源
python scripts/komodo.py servers
python scripts/komodo.py deployments
python scripts/komodo.py stacks
python scripts/komodo.py builds
python scripts/komodo.py procedures
python scripts/komodo.py repos

# Server 操作
python scripts/komodo.py server <name>
python scripts/komodo.py server-stats <name>

# Deployment 操作
python scripts/komodo.py deployment <name>
python scripts/komodo.py deploy <name>
python scripts/komodo.py start <name>
python scripts/komodo.py stop <name>
python scripts/komodo.py restart <name>
python scripts/komodo.py logs <name> [lines]

# Stack 操作
python scripts/komodo.py stack <name>
python scripts/komodo.py deploy-stack <name>
python scripts/komodo.py start-stack <name>
python scripts/komodo.py stop-stack <name>
python scripts/komodo.py restart-stack <name>
python scripts/komodo.py create-stack <name> <server> <compose.yml> [env_file]
python scripts/komodo.py delete-stack <name>
python scripts/komodo.py stack-logs <name> [service]

# Build 操作
python scripts/komodo.py build <name>
python scripts/komodo.py run-build <name>

# Procedure 操作
python scripts/komodo.py procedure <name>
python scripts/komodo.py run-procedure <name>
```

## 状态指示器

- 🟢 Running/Ok
- 🔴 Stopped
- ⚪ NotDeployed
- 🟡 Unhealthy
- 🔄 Restarting
- 🔨 Building
- ⏳ Pending

## 直接 API 调用

对于 CLI 未覆盖的操作, 使用 curl:

```bash
# Read 操作
curl -X POST "$KOMODO_ADDRESS/read/ListServers" \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: $KOMODO_API_KEY" \
  -H "X-Api-Secret: $KOMODO_API_SECRET" \
  -d '{}'

# Execute 操作
curl -X POST "$KOMODO_ADDRESS/execute/Deploy" \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: $KOMODO_API_KEY" \
  -H "X-Api-Secret: $KOMODO_API_SECRET" \
  -d '{"deployment": "my-deployment"}'
```

## API 参考

Read endpoints: `ListServers`, `ListDeployments`, `ListStacks`, `ListBuilds`, `ListProcedures`, `ListRepos`, `GetSystemStats`, `GetLog`

Execute endpoints: `Deploy`, `StartDeployment`, `StopDeployment`, `RestartDeployment`, `DeployStack`, `StartStack`, `StopStack`, `RestartStack`, `RunBuild`, `RunProcedure`

完整 API 文档: https://komo.do/docs
