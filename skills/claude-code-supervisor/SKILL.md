---
name: claude-code-supervisor
description: 监控运行于tmux中的Claude Code会话。通过bash预过滤（Option D）和快速LLM分类，检测错误、卡住的代理及任务完成情况。支持OpenClaw、Webhook、ntfy等多种通知后端。适用场景：(1)启动需要监控的长时Claude Code任务；(2)自动检测API错误和提前停止；(3)从后台编码代理获取进度报告；(4)会话/上下文限制重置后继续工作。需配置tmux和claude CLI。 Requires: tmux, claude CLI.
metadata:
  openclaw:
    emoji: 👷
    os:
    - darwin
    - linux
    requires:
      bins:
      - tmux
      anyBins:
      - claude
---

# Claude Code Supervisor

Bridge between Claude Code's lifecycle hooks and your agent harness.

## 架构

```
Claude Code (in tmux)
  │  Stop / Error / Notification
  ▼
Bash pre-filter (Option D)
  │  obvious cases handled directly
  │  ambiguous cases pass through
  ▼
Fast LLM triage (claude -p with Haiku, or local LLM)
  │  classifies: FINE | NEEDS_NUDGE | STUCK | DONE | ESCALATE
  │  FINE → logged silently
  ▼
Notify command (configurable)
  │  openclaw wake, webhook, ntfy, script, etc.
  ▼
Agent harness decides + acts
  │  nudge (send-keys to tmux), wait, escalate to human
```

## 快速开始

### 1. 安装 hooks into a 项目

```bash
{baseDir}/scripts/install-hooks.sh /path/to/your/project
```

Creates:
- `.claude/hooks/supervisor/` — hook scripts + triage
- `.claude/settings.json` — wired into Claude Code lifecycle
- `.claude-code-supervisor.yml` — 配置 (编辑 this)

### 2. 配置

编辑 `.claude-code-supervisor.yml`:

```yaml
triage:
  command: "claude -p --no-session-persistence"  # or: ollama run llama3.2
  model: "claude-haiku-4-20250414"

notify:
  command: "openclaw gateway call wake --params"  # or: curl, ntfy, script
```

### 3. Register a supervised session

创建 `~/.openclaw/workspace/supervisor-state.json` (or wherever your harness keeps state):

```json
{
  "sessions": {
    "my-task": {
      "socket": "/tmp/openclaw-tmux-sockets/openclaw.sock",
      "tmuxSession": "my-task",
      "projectDir": "/path/to/project",
      "goal": "Fix issue #42",
      "successCriteria": "Tests pass, committed",
      "maxNudges": 5,
      "escalateAfterMin": 60,
      "status": "running"
    }
  }
}
```

### 4. Launch Claude Code in tmux

```bash
SOCKET="/tmp/openclaw-tmux-sockets/openclaw.sock"
tmux -S "$SOCKET" new -d -s my-task
tmux -S "$SOCKET" send-keys -t my-task "cd /path/to/project && claude 'Fix issue #42'" Enter
```

Hooks fire automatically. Triage assesses. You 获取 notified only when it matters.

## How the Pre-筛选 Works (Option D)

Not every hook 事件 needs an LLM call. Bash catches the obvious cases first:

### on-停止.sh
| Signal | Bash decision | LLM triage? |
|--------|--------------|-------------|
| `max_tokens` | Always needs attention | ✅ Yes |
| `end_turn` + shell prompt back | Agent 也许 be 已完成 | ✅ Yes |
| `end_turn` + no prompt | Agent is mid-work | ❌ Skip |
| `stop_sequence` | 法线 | ❌ Skip |

### on-错误.sh
| Signal | Bash decision | LLM triage? |
|--------|--------------|-------------|
| API 429 / rate 极限 | Transient, 将 解决 | ❌ 记录 only |
| API 500 | Agent likely stuck | ✅ Yes |
| Other tool 错误 | Unknown severity | ✅ Yes |

### on-notify.sh
| Signal | Bash decision | LLM triage? |
|--------|--------------|-------------|
| `auth_*` | Internal, transient | ❌ Skip |
| `permission_prompt` | Needs decision | ✅ Yes |
| `idle_prompt` | Agent waiting | ✅ Yes |

## Triage Classifications

The LLM 返回 one of:

| Verdict | Meaning | Typical action |
|---------|---------|----------------|
| **FINE** | Agent is working normally | 记录 silently, no notification |
| **NEEDS_NUDGE** | Transient 错误, 应该 continue | 发送 "continue" 迁移到 tmux |
| **STUCK** | Looping or not progressing | Try different approach or escalate |
| **已完成** | 任务 completed successfully | Report 迁移到 human |
| **ESCALATE** | Needs human judgment | Notify human with context |

## Handling Notifications (for agent harness authors)

Wake events arrive with the prefix `cc-supervisor:` followed by the classification:

```
cc-supervisor: NEEDS_NUDGE | error:api_500 | cwd=/home/user/project | ...
cc-supervisor: DONE | stopped:end_turn:prompt_back | cwd=/home/user/project | ...
```

### Nudging via tmux

```bash
tmux -S "$SOCKET" send-keys -t "$SESSION" "continue — the API error was transient" Enter
```

### Escalation 格式

See `references/escalation-rules.md` for when 迁移到 nudge vs escalate and quiet hours.

## Watchdog (Who Watches the Watchman?)

Hooks depend on Claude Code being alive. If the session hard-crashes, hits account
limits, or the 处理 gets OOM-killed, no hooks fire. The watchdog catches this.

`scripts/watchdog.sh` is a pure bash script (no LLM, no Claude Code 依赖) that:
1. Reads `supervisor-state.json` for all `"running"` s`"running"``"r`"running"``"running"`
2. Checks: is the tmux socket alive? Is the session there? Is Claude Code still running?
3. If something is dead and no hook reported it → notifies via the configured 命令
4. Updates `lastWatchdogAt` in state for tracking

运行 it on a timer. Choose your poison:

**System cron:**
```bash
*/15 * * * * /path/to/claude-code-supervisor/scripts/watchdog.sh
```

**OpenClaw cron:**
```json
{
  "schedule": { "kind": "every", "everyMs": 900000 },
  "payload": { "kind": "systemEvent", "text": "cc-supervisor: watchdog — run /path/to/scripts/watchdog.sh and report" },
  "sessionTarget": "main"
}
```

**systemd timer, launchd, or whatever runs periodically on your box.**

The watchdog is deliberately dumb — no LLM, no complex logic, just "is the 处理 still
there?" This means it works even when the triage 模型 is down, the API is melting, or
your account hit its 极限. Belts and suspenders.

## 文件

- `scripts/install-hooks.sh` — one-命令 设置 per 项目
- `scripts/hooks/on-stop.sh` — 停止 事件 处理器 with bash pre-筛选
- `scripts/hooks/on-error.sh` — PostToolUseFailure 处理器 with bash pre-筛选
- `scripts/hooks/on-notify.sh` — Notification 处理器 with bash pre-筛选
- `scripts/triage.sh` — LLM triage (called by hooks for ambiguous cases)
- `scripts/lib.sh` — shared config loading and notification functions
- `scripts/watchdog.sh` — dead session detector (pure bash, no LLM 依赖)
- `references/state-patterns.md` — terminal 输出 pattern matching guide
- `references/escalation-rules.md` — when 迁移到 nudge vs escalate vs wait
- `supervisor.yml.example` — 示例 配置