---
name: agent-context
description: AI coding agent 的持久本地内存。两个文件，一个理念 — AGENTS.md（已提交，共享）+ .agents.local.md（gitignored，个人）。Agent 在 session 开始时读取两者，在 session 结束时更新 scratchpad，并随着时间的推移提升稳定模式。适用于 Claude Code、Cursor、Copilot、Windsurf。Subagent-ready。无插件，无基础设施，无后台进程。
---

# Agent Context System

## 问题

Agent 每次 session 都从零开始。你花一个小时让你的 coding agent 熟悉一个项目，关闭 session，第二天从零开始。Agent 忘记了一切。每次 session 都是冷启动。

这不是模型的限制。这是一个 context delivery 问题。Agent 有记忆的能力——他们只是没有在正确的时间以正确的格式获得正确的输入。

## 解决方案

两个 markdown 文件。一个已提交，一个 gitignored。Agent 在每次 session 开始时读取两者，在结束时更新本地文件。

- **`AGENTS.md`** — 你的项目 source of truth。已提交并共享。始终在 agent 的 prompt 中。120 行以内。包含压缩的项目知识：patterns, boundaries, gotchas, commands, architecture。

- **`.agents.local.md`** — 你的个人 scratchpad。Gitignored。随着时间的推移，随着 agent 记录每次 session 学到的内容而增长。Session notes, dead ends, preferences, 尚未提升的 patterns。

就是这样。没有插件，没有基础设施，没有后台进程。约定存在于文件本身内部，agent 遵循它。

## 工作原理

### 1. 设置

运行 init 脚本。它从模板创建 `.agents.local.md`，确保它是 gitignored，并连接你的 agent 工具配置（Claude Code 用 CLAUDE.md symlink，Cursor 用 .cursorrules，Windsurf 用 .windsurfrules，Copilot 用 copilot-instructions.md）。

```bash
# 如果你克隆了模板 repo：
./scripts/init-agent-context.sh

# 如果你作为 skill 安装（npx skills add）：
bash .agents/skills/agent-context-system/scripts/init-agent-context.sh
```

然后编辑 `AGENTS.md`，添加你的项目 specifics：name, stack, commands, 来自你的代码库的实际 patterns 和 gotchas。这是你最高杠杆的编辑。

### 2. Session 期间

Agent 在 session 开始时读取两个文件。`AGENTS.md` 给它压缩的项目知识。`.agents.local.md` 给它来自过去 session 的累积学习。Agent 现在拥有跨 session 持久的 context。

在 session 结束时，agent 将 session log 条目附加到 scratchpad：what changed, what worked, what didn't, decisions made, patterns learned。大多数 agent（Copilot Chat, Cursor, Windsurf）没有 session-end hooks，所以这取决于 `AGENTS.md` 中的 Rule 7 被看到并执行，或者用户说 "log this session"。Claude Code 通过 auto memory 自动处理这个。

### 3. 随着时间的推移

Scratchpad 增长。当它达到 300 行时，agent 压缩：deduplicate, merge related entries, keep it tight。在压缩期间，如果一个 pattern 在 3+ session 中出现，agent 使用 `AGENTS.md` 期望的相同 pipe-delimited 格式在 scratchpad 的 "Ready to Promote" 部分标记它。

你决定何时将标记的项目从 scratchpad 移动到 `AGENTS.md`。Scratchpad 是实验性的地方。`AGENTS.md` 是 proven knowledge 所在的地方。

```
Session notes → .agents.local.md → agent flags stable patterns → you promote to AGENTS.md
                    (personal)                                        (shared)
```

## 脚本

模板包含五个脚本：

### `init-agent-context.sh`

设置本地 agent scratchpad 和 agent 工具集成。每个 clone 运行一次。安全地重新运行。

- 从模板创建 `.agents.local.md`
- 确保 `.agents.local.md` 是 gitignored
- 询问你使用哪些 agent（Claude, Cursor, Windsurf, Copilot）并连接正确的配置
- 为 Claude Code 创建 CLAUDE.md symlink（因为它还不读取 AGENTS.md）
- 添加 agent context directive 到 .cursorrules, .windsurfrules, 或 copilot-instructions.md

### `compress.sh`

当 scratchpad 超过 300 行时压缩它。Deduplicates, merges related entries, flags stable patterns for promotion。尚未实现——agent 的说明在 AGENTS.md 的 Local Context 部分。

### `promote.sh`

将标记的项目从 `.agents.local.md` 的 "Ready to Promote" 部分移动到 `AGENTS.md`。尚未实现——目前是手动步骤。

### `validate.sh`

验证 `AGENTS.md` 保持在 120 行以内并检查格式一致性。尚未实现。

### `publish-template.sh`

推送到 GitHub 并标记为 template repo。运行一次。创建一个 private GitHub repo 并将其标记为 template，以便你可以使用 `gh repo create my-project --template YOUR_USERNAME/agent-context-system --private` 创建新项目。

## 知识流

知识不仅仅停留在一个地方。它会流动。

学习从 `.agents.local.md` 中的 session notes 开始。Agent 在每个 session 结束时写入它们。在压缩期间，如果一个 pattern 在 3+ session 中出现，agent 使用 `AGENTS.md` 期望的相同 pipe-delimited 格式在 scratchpad 的 "Ready to Promote" 部分标记它。然后你决定何时将其移动到 `AGENTS.md` 中。

```
Session notes → .agents.local.md → agent flags stable patterns → you promote to AGENTS.md
                    (personal)                                        (shared)
```

Scratchpad 是仍然实验性的东西所在的地方。`AGENTS.md` 是 proven knowledge 所在的地方。Agent 标记候选者。你做决定。

## AGENTS.md 模板结构

模板 `AGENTS.md` 包括：

### Project

基本元数据：name, stack, package manager。保持单行。

### Commands

Build, test, lint, dev server 的可执行命令。这些放在前面，因为 agent 需要立即使用它们。

### Architecture

每行一个目录。Agent 在每次 turn 时获得高层结构，无需查找任何东西。

### Project Knowledge (Compressed)

这是最重要的部分。三个子部分：

- **Patterns** — `pattern | where-to-see-it` 格式。Named exports only, server components default, Zustand for client state, Result types not try/catch。
- **Boundaries** — `rule | reason` 格式。Never modify src/generated, env vars through src/config, no default exports, no inline styles。
- **Gotchas** — `trap | fix` 格式。pnpm build hides type errors, dev sessions expire after 24h, integration tests need DB up。

Vercel 的 evals 显示 passive context（始终在 prompt 中）达到 100% pass rate，而 agent 必须决定查找时只有 53%。这个部分是 passive context。Agent 在每次 turn 时自动获得它。

### Rules

编号列表。Read AGENTS.md and .agents.local.md first. Plan before code. Locate files before changing. Only touch what the task requires. Run tests after every change. Summarize changes。

### Deep References

指向 `agent_docs/`，当任务需要比压缩版本更多的深度时。Conventions, architecture, gotchas — 完整细节，按需加载。

### Local Context

读取和更新 `.agents.local.md` 的说明。解释 session-to-session learning, compression protocol, promotion pathway。告诉 subagents 明确读取 scratchpad（它们不继承主对话历史）。

## .agents.local.md 模板结构

模板 `.agents.local.md` 包括：

### Preferences

你的风格、代码偏好、规划偏好。Friendly vs technical tone。Minimal changes vs comprehensive refactors。Always state the plan before writing code。

### Patterns

关于这个项目的既定事实。在这里提升重复的 session learnings。

### Gotchas

看起来正确但实际上不正确的东西。包含 WHY。

### Dead Ends

尝试过但失败的方法。包含 WHY 它们失败了。防止 agent 重复错误。

### Ready to Promote

Agent 在压缩期间标记项目，当一个 pattern 在 3+ session 中重复出现时。使用 `AGENTS.md` 期望的相同 pipe-delimited 格式。人类决定何时将标记的项目移动到 `AGENTS.md`。

### Session Log

在末尾附加新条目。每个 session 一个条目。每个保持在 5-10 行。模板：what changed, what worked, what didn't, decisions, learnings。

### Compression Log

当文件超过 300 行时，压缩。在这里记录。

## Agent 兼容性

模板适用于所有主要 agent 工具：

| Agent | Config File | What It Does |
|---|---|---|
| Claude Code | CLAUDE.md | Symlink → AGENTS.md (Claude doesn't read AGENTS.md yet) |
| Cursor | .cursorrules | Directive pointing to AGENTS.md |
| Windsurf | .windsurfrules | Directive pointing to AGENTS.md |
| GitHub Copilot | .github/copilot-instructions.md | Directive pointing to AGENTS.md |

### Claude Code Auto Memory

Claude Code 在 2025 年末推出了 auto memory。它创建一个 `~/.claude/projects/<project>/memory/` 目录，Claude 在其中写入自己的笔记并在 session 开始时加载它们。这基本上是内置于工具中的 `.agents.local.md` 概念。

如果你只使用 Claude Code，auto memory 处理 session-to-session learning，scratchpad 是可选的。模板对你的价值在于 `AGENTS.md` 文件本身和 promotion pathway，它为你提供了一种结构化的方式，将 auto memory 学到的内容中的稳定部分移动到你的根文件中。

如果你的团队使用多个 agent（GitHub 刚刚推出了 Agent HQ，将 Copilot, Claude, 和 Codex 并排使用），scratchpad 很重要，因为 auto memory 只在 Claude Code 中工作。Scratchpad 在任何地方都工作。

## Subagent Context

Claude Code 现在提供 subagents。Copilot CLI 刚刚在实验模式下推出了 /fleet。两个工具都朝着同一个模式发展：一个 lead agent 协调一个专家团队。

Subagents 不继承主对话的历史。每个都从一个干净的 context window 开始。它们唯一共享的知识是你的根指令文件。

所以当你从一个 agent 变为五个并行 agent 时，`AGENTS.md` 从 "helpful project context" 变为 "防止五个 agent 独立做出关于你的代码库的冲突决策的唯一东西"。

压缩的项目知识、boundaries 部分、gotchas — 那就是共享的大脑。没有它，每个 subagent 都会从头开始重新发现你的项目。

这就是为什么模板明确告诉 subagents 也读取 `.agents.local.md`。它们默认不会获得它。它们需要这个指令。

这也是为什么指令预算纪律更加重要。如果你的 `AGENTS.md` 是 500 行，你将为每个你生成的 subagent 支付该 token 成本。120 行以内是一个 feature，而不是限制。

## 研究基础

本模板建立在以下研究之上：

- **LangChain** — Context Engineering for Agents: Write/Select/Compress/Isolate 框架
- **HumanLayer** — Writing a Good CLAUDE.md: instruction budgets, root file discipline
- **GitHub** — Lessons from 2,500+ Repositories: what makes agents.md files actually work
- **Vercel** — AGENTS.md Outperforms Skills: passive context vs skill retrieval eval data
- **Anthropic** — Equipping Agents with Skills: three-tier progressive disclosure
- **AI Hero** — A Complete Guide to AGENTS.md: cross-platform standard adoption
- **Anthropic** — Claude Code Subagents: context isolation, custom agents
- **GitHub** — Copilot CLI /fleet: parallel fleets with dependency-aware tasks

关键发现：frontier LLMs 可靠地遵循约 150-200 条指令。Claude Code 的 system prompt 已经使用了约 50 条。所以你根文件中任何不普遍适用的东西都有被悄悄降低优先级的风险。

这就是为什么 `AGENTS.md` 保持在 120 行以内并使用压缩格式（pipe-delimited patterns, boundaries, gotchas）。Dense beats verbose。

另一个关键发现：Vercel 的 evals 显示 passive context（始终在 prompt 中）达到 100% pass rate，而 agent 必须决定查找时只有 53%。可用的文档不等于使用的文档。将关键知识放在 agent  literally cannot miss 的地方。

## 开始使用

### 从模板创建新 repo

```bash
gh repo create my-project --template YOUR_USERNAME/agent-context-system --private
cd my-project
chmod +x scripts/init-agent-context.sh
./scripts/init-agent-context.sh
```

### 作为 skill 安装

如果你通过 `npx skills add` 安装，脚本位于 skill 目录内，而不是项目根目录。从那里运行：

```bash
bash .agents/skills/agent-context-system/scripts/init-agent-context.sh
```

### 现有 repo

将 `AGENTS.md`、`agent_docs/` 和 `scripts/` 复制到你的项目根目录，然后运行 init 脚本。

### 将此发布为你的模板

```bash
chmod +x scripts/publish-template.sh
./scripts/publish-template.sh
```

## 设置之后

1. **编辑 `AGENTS.md`。** 填写你的项目名称、技术栈、命令。用代码库中的真实 patterns 和 gotchas 替换占位符。这是你最高杠杆的编辑。
2. **填写 `agent_docs/`。** 添加更深入的参考。删除不适用的内容。
3. **自定义 `.agents.local.md`。** 添加你的偏好。
4. **工作。** Agent 读取所有内容，完成任务，更新 scratchpad。
5. **提升有效的内容。** 在压缩期间，agent 在 scratchpad 的 "Ready to Promote" 部分标记在 3+ session 中重复出现的 patterns。你决定何时将它们移动到 `AGENTS.md`。

## 文件结构

```
your-repo/
├── AGENTS.md                    # Committed. Always loaded. Under 120 lines.
├── .agents.local.md             # Gitignored. Personal scratchpad.
├── agent_docs/                  # Deeper docs. Read only when needed.
│   ├── conventions.md           # Full code patterns, naming, file structure
│   ├── architecture.md          # System design, data flow, key decisions
│   └── gotchas.md               # Extended known traps with full explanations
├── scripts/
│   ├── init-agent-context.sh    # Setup script (run once per clone)
│   ├── publish-template.sh      # Publish as GitHub template repo
│   └── agents-local-template.md # Template for .agents.local.md
└── CLAUDE.md                    # Symlink → AGENTS.md (created by init)
```

## License

MIT
