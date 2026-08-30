---
name: bot-status-api
description: 部署轻量级状态 API，暴露 OpenClaw 机器人的运行时健康、服务连通性、cron 任务、技能和系统指标。适用于搭建监控仪表板、健康检查端点或状态页面，支持通过配置自定义 HTTP 检查、CLI 命令和文件检查，零依赖仅需 Node.js。
tags:
- API
---

# Bot Status API

一个可配置的 HTTP 服务，将你的 OpenClaw 机器人的运行状态以 JSON 形式暴露。为仪表板集成、监控和透明性而设计。

## 它提供什么

- **Bot Core:** 在线状态、模型、上下文使用、运行时间、心跳计时
- **Services:** 任何 HTTP 端点、CLI 工具或文件路径的健康检查
- **Email:** 来自任何邮箱提供商的未读计数（himalaya、gog 等）
- **Cron Jobs:** 直接从 OpenClaw 的 `cron/jobs.json` 读取
- **Docker:** 通过 Portainer API 的容器健康
- **Dev Servers:** 通过进程 grep 自动检测运行的开发服务器
- **Skills:** 列出已安装和可用的 OpenClaw 技能
- **System:** 来自 `/proc` 的 CPU、RAM、磁盘指标

## 设置

### 1. 复制服务文件

将 `server.js`、`collectors/` 和 `package.json` 复制到你想要的位置。

### 2. 创建 config.json

将 `config.example.json` 复制为 `config.json` 并自定义：

```json
{
  "port": 3200,
  "name": "MyBot",
  "workspace": "/path/to/.openclaw/workspace",
  "openclawHome": "/path/to/.openclaw",
  "cache": { "ttlMs": 10000 },
  "model": "claude-sonnet-4-20250514",
  "skillDirs": ["/path/to/openclaw/skills"],
  "services": [
    { "name": "myservice", "type": "http", "url": "http://...", "healthPath": "/health" }
  ]
}
```

### 服务检查类型

| 类型 | 说明 | 配置 |
|------|-------------|--------|
| `http` | 获取 URL，检查 HTTP 200 | `url`、`healthPath`、`method`、`headers`、`body` |
| `command` | 运行 shell 命令，检查退出 0 | `command`、`timeout` |
| `file-exists` | 检查路径是否存在 | `path` |

### 3. 运行

```bash
node server.js
```

### 4. 持久化（systemd 用户服务）

```ini
# ~/.config/systemd/user/bot-status.service
[Unit]
Description=Bot Status API
After=network.target

[Service]
Type=simple
WorkingDirectory=/path/to/bot-status
ExecStart=/usr/bin/node server.js
Restart=always
RestartSec=5
Environment=PORT=3200
Environment=HOME=/home/youruser
Environment=PATH=/usr/local/bin:/usr/bin:/bin

[Install]
WantedBy=default.target
```

```bash
systemctl --user daemon-reload
systemctl --user enable --now bot-status
loginctl enable-linger $USER  # 在注销后存活
```

### 5. 来自 OpenClaw 的上下文/生命体征

机器人应该定期将生命体征写入其工作区中的 `heartbeat-state.json`：

```json
{
  "vitals": {
    "contextPercent": 62,
    "contextUsed": 124000,
    "contextMax": 200000,
    "model": "claude-opus-4-5",
    "updatedAt": 1770304500000
  }
}
```

将此添加到你的 HEARTBEAT.md，以便机器人在每个心跳周期更新它。

## 端点

| Endpoint | 说明 |
|----------|-------------|
| `GET /status` | 完整状态 JSON（缓存） |
| `GET /health` | 简单的 `{"status":"ok"}` |

## 架构

- **零依赖** —— 仅 Node.js 内置模块（`http`、`fs`、`child_process`）
- **非阻塞** —— 所有 shell 命令使用异步 `exec`，从不使用 `execSync`
- **后台刷新** —— 缓存按间隔刷新，请求始终从缓存即时服务（~10ms）
- **配置驱动** —— `config.json` 中的所有内容，无硬编码值
