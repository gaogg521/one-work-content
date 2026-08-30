---
name: casual-cron
description: 从自然语言创建 Clawdbot cron 作业，带有严格的运行守护规则。在以下情况使用：用户要求安排提醒或消息（一次性或重复），特别是通过 Telegram，或者当他们使用 /at 或 /every 时。示例：'Create a daily reminder at 8am'、'Remind me in 20 minutes'、'Send me a Telegram message at 3pm'、'/every 2h'。
metadata:
  openclaw:
    emoji: ⏰
    requires:
      bins:
      - python3
      - openclaw
      env:
      - CRON_DEFAULT_CHANNEL
tags:
- 即时通讯
- 自动化
---

# Casual Cron

从自然语言创建 Clawdbot cron 作业。支持一次性及重复计划，带有安全的运行守护规则。

## Cron 运行守护（硬性规则）

- 在 cron 作业内部运行时：不要进行故障排除，不要重启 gateway，也不要检查时间。
- 不要发送确认或解释。
- 仅输出精确的消息负载，然后停止。

---

## 工作原理

1. Agent 从用户消息中检测调度意图（或 `/at` / `/every` 命令）
2. 解析：时间、频率、频道、目的地、消息
3. 使用正确的 flags 构建 `openclaw cron add` 命令
4. 在执行之前与用户确认解析后的时间、作业名称和作业 id

---

## 调度规则

当消息以 `/at` 或 `/every` 开头时，通过 CLI 进行调度（而不是 cron 工具 API）。

使用：`openclaw cron add`

### /at（一次性）

- 如果用户给出时钟时间（例如 "3pm"），转换为 ISO 格式，偏移量根据 America/New_York 在该日期计算（DST 安全）。
- 对于近期提醒，优先使用相对时间（例如 `--at "20m"`）。
- 使用 `--session isolated --message "Output exactly: <task>"`。
- 始终包含 `--delete-after-run`。
- 始终包含 `--deliver --channel <channel> --to <destination>`。

### /every（重复）

- 如果是间隔：使用 `--every "<duration>"`（无需时区）。
- 如果是时钟时间：使用 `--cron "<expr>" --tz "America/New_York"`。
- 使用 `--session isolated --message "Output exactly: <task>"`。
- 始终包含 `--deliver --channel <channel> --to <destination>`。

### 确认

- 在最终确定之前，始终与用户确认解析后的时间、作业名称和作业 id。

---

## 命令参考

一次性（时钟时间，DST 感知）：
```
openclaw cron add \
  --name "Reminder example" \
  --at "2026-01-28T15:00:00-05:00" \
  --session isolated \
  --message "Output exactly: <TASK>" \
  --deliver --channel telegram --to <TELEGRAM_CHAT_ID> \
  --delete-after-run
```

一次性（相对时间）：
```
openclaw cron add \
  --name "Reminder in 20m" \
  --at "20m" \
  --session isolated \
  --message "Output exactly: <TASK>" \
  --deliver --channel telegram --to <TELEGRAM_CHAT_ID> \
  --delete-after-run
```

重复（时钟时间，DST 感知）：
```
openclaw cron add \
  --name "Daily 3pm reminder" \
  --cron "0 15 * * *" --tz "America/New_York" \
  --session isolated \
  --message "Output exactly: <TASK>" \
  --deliver --channel telegram --to <TELEGRAM_CHAT_ID>
```

重复（间隔）：
```
openclaw cron add \
  --name "Every 2 hours" \
  --every "2h" \
  --session isolated \
  --message "Output exactly: <TASK>" \
  --deliver --channel telegram --to <TELEGRAM_CHAT_ID>
```

---

## 配置

| 设置 | 值 |
|---------|-------|
| 默认时区 | `America/New_York`（DST 感知） |
| 默认频道 | `telegram`（通过 `CRON_DEFAULT_CHANNEL` 环境变量覆盖） |
| 支持的频道 | telegram、whatsapp、slack、discord、signal |

---

## 支持的模式

### 时间格式

| 输入 | Cron |
|-------|------|
| `8am` | `0 8 * * *` |
| `8:45pm` | `45 20 * * *` |
| `noon` | `0 12 * * *` |
| `midnight` | `0 0 * * *` |
| `14:30` | `30 14 * * *` |

### 频率

| 输入 | 行为 |
|-------|----------|
| `daily` / `every day` | 在指定时间每日执行 |
| `weekdays` / `mon-fri` | 周一至周五在指定时间执行 |
| `mondays` / `every monday` | 每周一执行 |
| `hourly` / `every hour` | 每小时在 :00 执行 |
| `every 2 hours` | `0 */2 * * *` |
| `weekly` | 每周执行（默认为周一） |
| `monthly` | 每月执行（每月 1 日） |
