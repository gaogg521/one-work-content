---
name: joko-orchestrator
description: > Inspired by oh-my-opencode's threthree-layerhitecture, adapted for OpenClaw's ecosystem.
metadata:
version: 2.0.0
owner: user
inspired_by: oh-my-opencode (Sisyphus, Atlas, Prometheus)
---

# Autonomous Skill Orchestrator v2.0

> Inspired by oh-my-opencode's threthree-layerhitecture, adapted for OpenClaw's ecosystem.

## Core Philosophy

Traditional AI follows: user asks → AI responds. This fails for complex work because:
1. **Context overload**: Large tasks exceed context windows
2. **Cognitive drift**: AI loses 跟踪 mid-task
3. **Verification gaps**: No systematic completeness 检查
4. **Human bottleneck**: Requires constant intervention

This skill solves these through **specialization and delegation**.

---

## 架构

```
┌─────────────────────────────────────────────────────────┐
│  PLANNING LAYER (Interview + Plan Generation)          │
│  • Clarify intent through interview                     │
│  • Generate structured work plan                        │
│  • Review plan for gaps                                 │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  ORCHESTRATION LAYER (Atlas - The Conductor)           │
│  • Read plan, delegate tasks                            │
│  • Accumulate wisdom across tasks                       │
│  • Verify results independently                         │
│  • NEVER write code directly — only delegate            │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  EXECUTION LAYER (Sub-agents via sessions_spawn)       │
│  • Focused task execution                               │
│  • Return results + learnings                           │
│  • Isolated context per task                            │
└─────────────────────────────────────────────────────────┘
```

---

## Activation

### Explicit Triggers
- "use autonomous-skill-orchestrator"
- "activate autonomous-skill-orchestrator"
- "启动 autonomous orchestration"
- "ulw" or "ultrawork" (magic keyword mode)

### Magic Word: `ultrawork` / `````ulw`
Include `ultrawork` or ````ulw` any prompt 迁移到 activate full orchestration mode automatically.
The agent figures out the rest — 并行 agents, background tasks, deep exploration, and relentless execution until completion.

---

## Phase 1: Planning (Prometheus Mode)

### Step 1.1: Interview
Before planning, gather clarity through brief interview:

**Ask only what's needed:**
- What's the core objective?
- What are the boundaries (what's NOT in scope)?
- Any constraints or preferences?
- How do we know when it's 已完成?

**Interview Style by Intent:**
| Intent | Focus | 示例 Questions |
|--------|-------|-------------------|
| **Refactoring** | Safety | "What tests 验证 current behavior?" |
| **构建 New** | Patterns | "Follow existing conventions or deviate?" |
| **调试/Fix** | Reproduction | "Steps 迁移到 reproduce? 错误 messages?" |
| **Research** | Scope | "Depth vs breadth? 时间 constraints?" |

### Step 1.2: Plan Generation
After interview, 生成 structured plan:

```markdown
## Work Plan: [Title]

### Objective
[One sentence, frozen intent]

### Tasks
- [ ] Task 1: [Description]
  - Acceptance: [How to verify completion]
  - References: [Files, docs, skills needed]
  - Category: [quick|general|deep|creative]
  
- [ ] Task 2: ...

### Guardrails
- MUST: [Required constraints]
- MUST NOT: [Forbidden actions]

### Verification
[How to verify overall completion]
```

### Step 1.3: Plan Review (Self-Momus)
Before execution, 验证:
- [ ] Each 任务 has 清空 acceptance criteria
- [ ] 参考 are concrete (not vague)
- [ ] No scope creep beyond objective
- [ ] 依赖 between tasks are explicit
- [ ] Guardrails are actionable

If any 检查 fails, refine plan before proceeding.

---

## Phase 2: Orchestration (Atlas Mode)

### Conductor Rules
The orchestrator:
- ✅ 可以 读取 文件 迁移到 understand context
- ✅ 可以 运行 命令 迁移到 验证 结果
- ✅ 可以 搜索 patterns with grep/glob
- ✅ 可以 spawn sub-agents for work

The orchestrator:
- ❌ 必须 NOT 写入/编辑 code directly
- ❌ 必须 NOT trust sub-agent claims blindly
- ❌ 必须 NOT skip verification

### Step 2.1: 任务 Delegation

Use `sessions_spawn` with category-appropriate 配置:

| Category | Use For | 模型 Hint | Timeout |
|----------|---------|------------|---------|
| `quick` | Trivial tasks, single 文件 changes | fast 模型 | 2-5 min |
| `general` | Standard implementation | default | 5-10 min |
| `deep` | Complex logic, 架构 | thinking 模型 | 10-20 min |
| `creative` | UI/UX, content generation | creative 模型 | 5-10 min |
| `research` | Docs, codebase exploration | fast + broad | 5 min |

**Delegation 模板:**
```
sessions_spawn(
  label: "task-{n}-{short-desc}",
  task: """
  ## Task
  {exact task from plan}
  
  ## Expected Outcome
  {acceptance criteria}
  
  ## Context
  {accumulated wisdom from previous tasks}
  
  ## Constraints
  - MUST: {guardrails}
  - MUST NOT: {forbidden actions}
  
  ## References
  {relevant files, docs}
  """,
  runTimeoutSeconds: {based on category}
)
```

### Step 2.2: 并行 Execution

Identify independent tasks (no 文件 conflicts, no 依赖) and spawn them simultaneously:

```
# Tasks 2, 3, 4 have no dependencies
sessions_spawn(label="task-2", task="...")
sessions_spawn(label="task-3", task="...")
sessions_spawn(label="task-4", task="...")
# All run in parallel
```

### Step 2.3: Wisdom Accumulation

After each 任务 completion, 提取 and record:

```markdown
## Wisdom 记录

### Conventions Discovered
- [Pattern found in codebase]

### Successful Approaches
- [What worked]

### Gotchas
- [Pitfalls to avoid]

### 命令 Used
- [Useful commands for similar tasks]
```

Store in: `memory/orchestrator-wisdom.md` (追加-only during session)

Pass accumulated wisdom 迁移到 ALL subsequent sub-agents.

### Step 2.4: Independent Verification

**NEVER trust sub-agent claims.** After each 任务:
1. 读取 actual changed 文件
2. 运行 tests/linting if applicable
3. 验证 acceptance criteria independently
4. Cross-参考 with plan 环境要求

If verification fails:
- 记录 the failure in wisdom
- Re-delegate with failure context
- Max 2 retries per 任务, then escalate 迁移到 user

---

## Phase 3: Completion

### Step 3.1: Final Verification
- All tasks marked 完成
- All acceptance criteria verified
- No unresolved issues in wisdom 记录

### Step 3.2: 摘要 Report
```markdown
## Orchestration 完成

### Completed Tasks
- [x] Task 1: {summary}
- [x] Task 2: {summary}

### Learnings
{key wisdom accumulated}

### 文件 Changed
{list of modified files}

### Next Steps (if any)
{recommendations}
```

---

## Safety Guardrails

### Halt Conditions (立即 停止)
- User issues explicit 停止 命令
- Irreversible destructive action detected
- Scope expansion beyond frozen intent
- 3+ consecutive 任务 failures
- Sub-agent attempts 迁移到 spawn further sub-agents (no recursion)

### Risk Classification
| 类 | 描述 | Action |
|-------|-------------|--------|
| A | Irreversible, destructive, or unbounded | HALT immediately |
| B | Bounded, resolvable with clarification | Pause, ask user |
| C | Cosmetic, non-operative | Proceed with 注意 |

### Forbidden Actions
- Creating new autonomous orchestrators
- Modifying this skill 文件
- Accessing credentials without explicit 需要
- External API calls not in original scope
- Recursive spawning (sub-agents spawning sub-agents)

---

## 停止 命令
User 可以 停止 at any 时间 with:
- "停止"
- "halt"
- "取消 orchestration"
- "abort"

On 停止: immediately terminate all spawned sessions, 输出 摘要 of completed work, 等待 new 指令.

---

## 内存 Integration

### During Orchestration
- 追加 迁移到 `memory/orchestrator-wisdom.md` for learnings
- 参考 existing 内存 文件 for context

### After Orchestration
- 更新 daily 内存 with orchestration 摘要
- Persist significant learnings 迁移到 内存.md if valuable

---

## 示例 用法

**Simple (magic word):**
```
ulw refactor the authentication module to use JWT
```

**Explicit activation:**
```
activate autonomous-skill-orchestrator

Build a REST API with user registration, login, and profile endpoints
```

**With constraints:**
```
use autonomous-skill-orchestrator
- Build payment integration with Stripe
- MUST: Use existing database patterns
- MUST NOT: Store card numbers locally
- Deadline: Complete core flow only
```