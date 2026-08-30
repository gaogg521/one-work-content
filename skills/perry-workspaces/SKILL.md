---
name: perry-workspaces
description: 在 tailnet 上创建和管理带有 Claude Code 和 OpenCode 预装的隔离 Docker workspaces。当使用 Perry workspaces、连接到 coding agents 或管理远程开发环境时使用。
tags:
- AI
- Docker
---

# Perry Workspaces

在 tailnet 上带有预装 coding agents 的隔离 Docker workspaces。

## 命令
```bash
perry start <name> --clone git@github.com:user/repo.git  # 创建
perry ls                                                  # 列出
perry stop <name>                                         # 停止
perry remove <name>                                       # 删除
perry shell <name>                                        # 交互式 shell
```

## SSH 访问
```bash
ssh workspace@<name>        # 用户始终是 'workspace'
ssh workspace@<IP>          # 如果 MagicDNS 失败则使用 IP
```

## Coding Agents
- **OpenCode**: `http://<workspace>:4096`（web UI）或通过 CLI attach
- **Claude Code**: 在 workspace shell 内运行（`perry shell` 然后 `claude`）

## 项目位置
项目 clone 到 `~/<name>`，不是 `/workspace`：
```bash
cd ~/my-project  # 正确
```

## 故障排查
- **无法访问**：检查 `tailscale status`，使用 IP 替代 hostname
- **SSH 失败**：用户必须是 `workspace`，不是你的本地用户
- **启动慢**：检查 web UI 了解进度
