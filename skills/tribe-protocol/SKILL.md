---
name: tribe-protocol
version: 1.0.0
description: 每次非所有者交互的强制信任查询。 在响应前查询 tribe.db 以检查实体信任层级、频道访问 和数据边界。首次安装时运行 'tribe init'。在每次 非所有者响应前使用 'tribe lookup <discord_id>'。
commands:
  tribe: scripts/tribe.sh
tags:
- 数据
- 社交媒体
---

# Tribe Protocol

OpenClaw bots 的信任查询系统。每次非所有者交互都必须在响应前针对 tribe 数据库进行验证。

## Quick Start

```bash
# 初始化（仅限首次）
./scripts/tribe.sh init \
  --bot-name Cheenu \
  --bot-discord-id 000000000000000004 \
  --human-name Nagarjun \
  --human-discord-id 000000000000000002

# 在响应前查找实体
./scripts/tribe.sh lookup <discord_id>

# 添加实体
./scripts/tribe.sh add --name Yajat --type human --discord-id 000000000000000001 --tier 3

# 管理信任
./scripts/tribe.sh set-tier <discord_id> 3 --reason "Promoted to tribe"
./scripts/tribe.sh set-status <discord_id> blocked --reason "Bad actor"
```

## Trust Tiers

| Tier | Label | Access |
|------|-------|--------|
| 4 | Owner | Full trust, all data |
| 3 | Tribe | Collaborate freely, no private data |
| 2 | Acquaintance | Public info only |
| 1 | Stranger | Minimal engagement |
| 0 | Blocked | Ignore completely |

## How It Works

1. 来自非所有者的消息到达
2. Bot 读取 TRIBE.md（在工作区根目录生成）
3. Bot 运行 `tribe lookup <discord_id>`
4. Script 返回实体信息 + 层级规则提醒
5. Bot 应用层级适当的行为

## Commands

- `tribe init` — 初始化数据库
- `tribe lookup` — 通过 discord_id、name、tag、server 查询实体
- `tribe add` — 添加新实体
- `tribe set-tier` — 更新信任层级
- `tribe set-status` — 更新状态
- `tribe grant` / `tribe revoke` — 频道访问
- `tribe tag` — 管理标签
- `tribe roster` — 列出所有实体
- `tribe log` — 审计跟踪
- `tribe export` — 导出为 markdown
- `tribe stats` — 快速摘要

## Environment Variables

- `TRIBE_DB` — 覆盖数据库路径
- `CLAWD_HOME` — 基础目录（默认：~/clawd）

## Dependencies

- `sqlite3`（macOS/大多数 Linux 预装）
