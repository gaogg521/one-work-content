---
name: config-guardian
description: 验证并保护 OpenClaw 配置更新（openclaw.json 或 openclaw config set/apply）。每当更改网关配置、模型、通道、代理、工具、会话或路由时使用此技能。强制要求备份、模式验证和安全回滚，然后才能重新启动。
tags:
- 配置
---

# Config Guardian

## 概述
每当编辑 `~/.openclaw/openclaw.json` 或运行 `openclaw config set/apply` 时使用此工作流。它可以防止无效配置、创建备份、针对模式进行验证并启用回滚。

## 工作流（每次使用）

1. **飞行前检查**
   - 确认请求的更改和范围。
   - 检查敏感密钥（令牌、凭证）。

2. **备份**
   - 运行 `scripts/backup_config.sh` 以创建带时间戳的快照。

3. **验证（更改前）**
   - 运行 `scripts/validate_config.sh`。
   - 如果验证失败，停止并报告。

4. **应用更改**
   - 对于小更改，优先使用 `openclaw config set <path> <value>`。
   - 对于复杂编辑，直接编辑文件并保持差异最小。

5. **验证（更改后）**
   - 再次运行 `scripts/validate_config.sh`。
   - 如果失败，使用 `scripts/restore_config.sh` 从备份恢复。

6. **重新启动（仅在获得明确批准时）**
   - 如果更改需要重新启动，请先征求批准。
   - 使用 `openclaw gateway restart`。

## 护栏
- **绝不**在没有明确用户批准的情况下重新启动或应用配置。
- **绝不**除非有要求，否则移除密钥或重新排序块。
- **始终**在编辑前保留备份。
- 如果不确定模式：运行 `openclaw doctor --non-interactive` 并在出错时停止。

## 脚本
- `scripts/backup_config.sh` — 创建带时间戳的备份
- `scripts/validate_config.sh` — 通过 OpenClaw doctor 验证配置
- `scripts/diff_config.sh` — 比较当前配置与备份
- `scripts/restore_config.sh` — 恢复备份

## 验证
- 使用 `openclaw doctor --non-interactive` 进行模式验证
- 这会针对网关使用的实际模式进行检查
- 警告未知密钥、无效类型和安全问题
