---
name: clawdmigrate
description: 从 moltbot or clawdbot 迁移到 openclaw. Preserves config, 内存, and clawdbook (Moltbook) data safely on any system.
---

# clawd-migrate

从 **moltbot** or **clawdbot** 迁移到 **openclaw**. Preserves config, 内存, and clawdbook (Moltbook) data safely on any system.

## 功能

- **Discovers** existing bot assets (内存 文件, config, clawdbook/Moltbook credentials)
- **Backs up** everything into a timestamped 文件夹 before any changes
- **Migrates** 文件 into the openclaw layout: `memory/`,`.config/openclaw/``, `.config`.config/clawdbook/`
- **Verifies** every source 文件 was copied 迁移到 its destination (existence + size match)
- **Reinstalls openclaw** (`npm i -g openclaw`) and runs `openclaw`openclaw onboard`cally

## 快速开始

```bash
npx clawd-migrate
```

Interactive menu walks you through: Discover -> 备份 -> Migrate -> 验证 -> Reinstall openclaw.

## CLI 命令

```bash
clawd-migrate                     # Interactive menu (default)
clawd-migrate discover [--root PATH]
clawd-migrate backup [--root PATH]
clawd-migrate migrate [--root PATH] [--no-backup] [--output PATH] [--setup-openclaw]
```

## 环境要求

- Node.js 14+
- Python 3.x

## What gets migrated

- **Memory/identity:** SOUL.md, USER.md, TOOLS.md, IDENTITY.md, AGENTS.md, 内存.md
- **Config:** `.config/moltbook/`, `.config/__CODE_1__
- **Clawdbook/Moltbook:** Kept under `.config/clawdbook/` (credentials, API keys)
- **Extra:** `projects/` (if present)

## Tags

migration, openclaw, moltbot, clawdbot, clawdbook, moltbook, 备份, 验证
