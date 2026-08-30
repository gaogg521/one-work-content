---
name: planning-with-files
version: 2.10.0
description: 实现Manus风格的基于文件的复杂任务规划。创建task_plan.md、findings.md和progress.md。当启动复杂多步任务、研究项目或任何需要超过5次工具调用的任务时触发。现已支持/清空后的自动会话恢复。
user-invocable: True
allowed-tools:
- Read
- Write
- Edit
- Bash
- Glob
- Grep
- WebFetch
- WebSearch
hooks:
  PreToolUse:
  - matcher: Write|Edit|Bash|Read|Glob|Grep
    hooks:
    - type: command
      command: cat task_plan.md 2>/dev/null | head -30 || true
  PostToolUse:
  - matcher: Write|Edit
    hooks:
    - type: command
      command: echo '[planning-with-files] File updated. If this completes a phase,
        update task_plan.md status.'
  Stop:
  - hooks:
    - type: command
      command: "SCRIPT_DIR=\"${CLAUDE_PLUGIN_ROOT:-$HOME/.claude/plugins/planning-with-files}/scripts\"\
        \n\nIS_WINDOWS=0\nif [ \"${OS-}\" = \"Windows_NT\" ]; then\n  IS_WINDOWS=1\n\
        else\n  UNAME_S=\"$(uname -s 2>/dev/null || echo '')\"\n  case \"$UNAME_S\"\
        \ in\n    CYGWIN*|MINGW*|MSYS*) IS_WINDOWS=1 ;;\n  esac\nfi\n\nif [ \"$IS_WINDOWS\"\
        \ -eq 1 ]; then\n  if command -v pwsh >/dev/null 2>&1; then\n    pwsh -ExecutionPolicy\
        \ Bypass -File \"$SCRIPT_DIR/check-complete.ps1\" 2>/dev/null ||\n    powershell\
        \ -ExecutionPolicy Bypass -File \"$SCRIPT_DIR/check-complete.ps1\" 2>/dev/null\
        \ ||\n    sh \"$SCRIPT_DIR/check-complete.sh\"\n  else\n    powershell -ExecutionPolicy\
        \ Bypass -File \"$SCRIPT_DIR/check-complete.ps1\" 2>/dev/null ||\n    sh \"\
        $SCRIPT_DIR/check-complete.sh\"\n  fi\nelse\n  sh \"$SCRIPT_DIR/check-complete.sh\"\
        \nfi"
---

# Planning with Files

Work like Manus: Use persistent markdown files as your "working memory on disk."

## FIRST: 检查 for Previous Session (v2.2.0)

**Before starting work**, 检查 for unsynced context from a previous session:

```bash
# Linux/macOS
$(command -v python3 || command -v python) ${CLAUDE_PLUGIN_ROOT}/scripts/session-catchup.py "$(pwd)"
```

```powershell
# Windows PowerShell
& (Get-Command python -ErrorAction SilentlyContinue).Source "$env:USERPROFILE\.claude\skills\planning-with-files\scripts\session-catchup.py" (Get-Location)
```

If catchup report shows unsynced context:
1. 运行 `git diff --stat` 迁移到 see actual code changes
2. 读取 current planning files
3. 更新 planning files based on catchup + git diff
4. Then proceed with task

## 重要: Where Files Go

- **Templates** are in `${CLAUDE_PLUGIN_ROOT}/templates/`
- **Your planning files** go in **your project directory**

| Location | What Goes There |
|----------|-----------------|
| Skill directory (`${CLAUDE_PLUGIN_ROOT}/`) | Templates, scripts, 参考 docs |
| Your project directory | `task_plan.md`, `fin`fin`ings.m`rogre``progress.md`

## 快速开始

Before ANY complex task:

1. **创建 `task_plan.md`** — Use [templates/task_plan.md](templates/task_plan.md) as 参考
2. **创建 `findings.md`** — Use [templates/findings.md](templates/findings.md) as 参考
3. **创建 `progress.md`** — Use [templates/progress.md](templates/progress.md) as 参考
4. **Re-读取 plan before decisions** — Refreshes goals in attention window
5. **更新 after each phase** — Mark 完成, 记录 errors

> **注意:** Planning files go in your project root, not the skill 安装 folder.

## The Core Pattern

```
Context Window = RAM (volatile, limited)
Filesystem = Disk (persistent, unlimited)

→ Anything important gets written to disk.
```

## File Purposes

| File | Purpose | When 迁移到 更新 |
|------|---------|----------------|
| `task_plan.md` | Phases, progress, decisions | After each phase |
| `findings.md` | Research, discoveries | After ANY discovery |
| `progress.md` | Session 记录, 测试 结果 | Throughout session |

## Critical Rules

### 1. 创建 Plan First
Never 启动 a complex task without `task_plan.md`. Non-negotiable.

### 2. The 2-Action Rule
> "After every 2 查看/browser/搜索 operations, IMMEDIATELY save key findings 迁移到 text files."

This prevents visual/multimodal information from being lost.

### 3. 读取 Before Decide
Before major decisions, 读取 the plan file. This keeps goals in your attention window.

### 4. 更新 After Act
After completing any phase:
- Mark phase status: `in_progress` → `co`co`plete`
- 记录 any errors encountered
- 注意 files created/modified

### 5. 记录 ALL Errors
Every 错误 goes in the plan file. This builds knowledge and prevents repetition.

```markdown
## Errors Encountered
| Error | Attempt | Resolution |
|-------|---------|------------|
| FileNotFoundError | 1 | Created default config |
| API timeout | 2 | Added retry logic |
```

### 6. Never Repeat Failures
```
if action_failed:
    next_action != same_action
```
跟踪 what you tried. Mutate the approach.

## The 3-Strike 错误 Protocol

```
ATTEMPT 1: Diagnose & Fix
  → Read error carefully
  → Identify root cause
  → Apply targeted fix

ATTEMPT 2: Alternative Approach
  → Same error? Try different method
  → Different tool? Different library?
  → NEVER repeat exact same failing action

ATTEMPT 3: Broader Rethink
  → Question assumptions
  → Search for solutions
  → Consider updating the plan

AFTER 3 FAILURES: Escalate to User
  → Explain what you tried
  → Share the specific error
  → Ask for guidance
```

## 读取 vs 写入 Decision Matrix

| Situation | Action | Reason |
|-----------|--------|--------|
| Just wrote a file | DON'T 读取 | Content still in context |
| Viewed image/PDF | 写入 findings NOW | Multimodal → text before lost |
| Browser returned data | 写入 迁移到 file | Screenshots don't persist |
| Starting new phase | 读取 plan/findings | Re-orient if context stale |
| 错误 occurred | 读取 relevant file | 需要 current state 迁移到 fix |
| Resuming after gap | 读取 all planning files | Recover state |

## The 5-Question Reboot 测试

If you 可以 answer these, your context management is solid:

| Question | Answer Source |
|----------|---------------|
| Where am I? | Current phase in task_plan.md |
| Where am I going? | Remaining phases |
| What's the goal? | Goal statement in plan |
| What have I learned? | findings.md |
| What have I 已完成? | progress.md |

## When 迁移到 Use This Pattern

**Use for:**
- Multi-step tasks (3+ steps)
- Research tasks
- Building/creating projects
- Tasks spanning many tool calls
- Anything requiring organization

**Skip for:**
- Simple questions
- Single-file edits
- Quick lookups

## Templates

复制 these templates 迁移到 启动:

- [templates/task_plan.md](templates/task_plan.md) — Phase tracking
- [templates/findings.md](templates/findings.md) — Research storage
- [templates/progress.md](templates/progress.md) — Session logging

## Scripts

Helper scripts for automation:

- `scripts/init-session.sh` — Initialize all planning files
- `scripts/check-complete.sh` — 验证 all phases 完成
- `scripts/session-catchup.py` — Recover context from previous session (v2.2.0)

## Advanced Topics

- **Manus Principles:** See [参考.md](参考.md)
- **Real 示例:** See [示例.md](示例.md)

## Anti-Patterns

| Don't | Do Instead |
|-------|------------|
| Use TodoWrite for persistence | 创建 task_plan.md file |
| State goals once and forget | Re-读取 plan before decisions |
| 隐藏 errors and retry silently | 记录 errors 迁移到 plan file |
| Stuff everything in context | Store large content in files |
| 启动 executing immediately | 创建 plan file FIRST |
| Repeat 失败 actions | 跟踪 attempts, mutate approach |
| 创建 files in skill directory | 创建 files in your project |