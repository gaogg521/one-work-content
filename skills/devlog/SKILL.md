---
name: devlog
tags:
- devlog
- 文档
- dev blog
- builder's log
- coding session blog
description: 从 AI 编程会话记录生成叙事式博客文章。读取会话文件，选择与主题相关的会话，并生成一篇由 agent 叙述的关于人机协作的博客文章。支持 builder's log、tutorial 和 technical deep-dive 风格。
version: 0.1.0
---

# DevLog Generator

从人机编程会话记录生成叙事式开发者博客文章。博客从 agent 的第一人称视角撰写——"我"是 agent，人类开发者被称为 "my human"。

## Workflow

### Phase 1: Understand the Request

从用户消息中提取：

- **Project** — 哪个代码库？（"eastore", "filecoin", "couponswap"）。如果未指定，使用当前工作目录。
- **Topic/feature** — 具体是什么？（"auth system", "dashboard" 或整个项目）。如果未指定，包含所有会话。
- **Style** — builder's log（默认）、tutorial 或 technical deep-dive。仅在用户明确要求时覆盖。
- **Time range** — "last week", "January sessions" 或 all（默认）。

### Phase 2: Discover Sessions

确定要扫描哪个平台。检查 `references/platforms/` 中支持的平台 — 每个子目录是一个平台。从当前环境或用户请求中自动检测。

仅加载相关的平台目录。每个目录包含一个参考文件（存储 schema、会话路径、发现指令）和脚本（`list-sessions.sh`, `read-session.sh`）。永远不要预先加载所有平台参考。

运行平台的 `list-sessions.sh <project>` 来扫描匹配的会话，或手动按照平台参考文件中的发现指令操作。

如果平台在 `references/platforms/` 中没有参考目录，则手动发现会话 — 检查平台的数据/配置目录（例如 `~/.local/share/`, `~/.config/`, `~/Library/`），查找会话存储文件（JSONL, JSON, SQLite），并检查 schema 以提取人机对话。遵循 Phase 3 中的相同过滤原则。

将会话索引呈现给用户确认。

### Phase 3: Select & Read

从会话索引中，确定哪些会话与用户的主题相关。读取所选会话的完整记录。

读取记录时，积极过滤：

**Keep:**
- User messages (text) — 人类的意图、方向、纠正
- Assistant messages (text) — agent 的推理、提案、解释
- Tool call names + file paths — 构建了什么
- Error messages — 挣扎和调试

**Strip:**
- `tool_result` content bodies (raw file contents, grep output — 80-90% 的 token 大小)
- System messages, usage metadata, compaction/summary entries
- Full tool input arguments (仅保留 name + file path, 不要整个 diff)

参考 Phase 2 中加载的平台参考文件，了解平台特定的字段名称和解析细节。

如果过滤后的记录仍超出上下文，则按会话处理：生成每会话摘要，然后跨会话综合。优先人机对话而非工具调用细节。

### Phase 4: Write the Blog

读取 `references/blog-writing-guide.md` 获取 agent 叙述的写作指南。其中包含 voice 定义、协作词汇、记录提取模式和博客结构。

从 `examples/` 中读取适合风格的示例：
- `examples/builders-log.md` 用于 builder's log 风格（默认）
- `examples/tutorial.md` 用于 tutorial 风格
- `examples/technical.md` 用于 technical deep-dive 风格

将 `assets/devlog-template.md` 加载为博客骨架。这是一个起始结构，不是 rigid 格式 — 根据会话记录实际包含的内容调整章节、重新排序、合并或删除标题。单会话博客可以完全跳过 phase 标题。迭代频繁的会话可以将 "The Hard Part" 扩展为多个章节。让故事决定形状。

按照写作指南生成博客。博客必须由 agent 以第一人称（"I"）叙述，将人类开发者称为 "my human"。当会话涉及架构、流程或多组件交互时，包含 Mermaid 图表（` ```mermaid ` code blocks）来可视化系统 — 参见写作指南中的图表章节了解何时以及如何使用。

### Phase 5: Output

将博客写入当前工作目录下的 `{project}-{topic}-devlog.md`，或用户指定的路径。

报告：title, word count, sessions included, time span covered, key files referenced.

### Phase 6: Publish

1. 询问用户是否想要在线发布博客。
2. 如果是，检查 `references/publishing/` 中支持的平台。每个子目录是一个发布平台。
3. 加载相关平台的参考文件以获取 API 细节和要求。
4. 检查所需的环境变量（例如 Hashnode 的 `HASHNODE_PAT`, `HASHNODE_PUBLICATION_ID`）。
5. 如果缺少任何变量，告诉用户需要设置什么以及如何设置 — 例如在 `~/.zshrc` 或 `~/.bashrc` 中 `export HASHNODE_PAT=...` 以供将来会话使用。请用户为当前会话提供值。
6. **Cover image (optional):** 如果你有图像生成能力（例如图像生成工具或 MCP server），生成一张视觉上代表博客主题的封面图片。将其上传到公开可访问的 URL，并通过 `--cover-image <url>` 标志传递给 `publish.sh`。图片应为横向（1200×630 或类似），视觉上与博客主题相关，且不包含重复标题的文字。如果你没有图像生成能力，跳过此步骤 — 博客没有封面图片也能正常发布。
7. 运行平台的 `publish.sh`，传入博客文件路径和 title（如果生成了封面图片，则加上 `--cover-image <url>`）。
8. 向用户报告已发布帖子的 URL。

## Edge Cases

| Scenario | Handling |
|---|---|
| No sessions found | 报告扫描了哪些路径。请用户检查项目名称或提供路径。 |
| Ambiguous project match | 列出匹配的项目，请用户选择。 |
| Single session | 更简单的结构 — 不需要多会话 phase 标题。 |
| Huge session (5000+ lines) | 按 turn group 分块，总结章节，然后综合。 |
| Mixed platforms | 按时间顺序合并来自多个平台的会话。 |
| Subagent transcripts | 默认跳过。主会话已经引用了它们的结果。 |
| Current session | 当用户说 "what we just did" — 直接使用当前会话上下文，不需要 JSONL。 |
| Compacted sessions | Compaction 不会删除数据。Raw messages 保留。读取所有内容，跳过 compaction/summary 行。 |
| User declines to publish | 完全跳过 Phase 6。博客文件已从 Phase 5 保存在本地。 |

## Resources

### Platform References (load only the relevant one)
- **`references/platforms/claude-code/`** — Claude Code reference + scripts
  - `claude-code.md` — Session paths, JSONL schema, discovery instructions
  - `list-sessions.sh` — Scan Claude Code projects for matching sessions
  - `read-session.sh` — Extract transcript from Claude Code JSONL
- **`references/platforms/opencode/`** — OpenCode reference + scripts
  - `opencode.md` — Storage layout, JSON hierarchy, discovery instructions
  - `list-sessions.sh` — Scan OpenCode projects for matching sessions
  - `read-session.sh` — Extract transcript from OpenCode's JSON hierarchy
- **`references/platforms/openclaw/`** — OpenClaw reference + scripts
  - `openclaw.md` — Session paths, JSONL schema, discovery instructions
  - `list-sessions.sh` — Scan OpenClaw agents for matching sessions
  - `read-session.sh` — Extract transcript from OpenClaw JSONL
- **`references/platforms/codex/`** — Codex reference + scripts
  - `codex.md` — Rollout file format, JSONL schema, discovery instructions
  - `list-sessions.sh` — Scan Codex rollout files for matching sessions
  - `read-session.sh` — Extract transcript from Codex rollout JSONL
- **`references/platforms/gemini-cli/`** — Gemini CLI reference + scripts
  - `gemini-cli.md` — JSON session format, SHA256 project hashing, discovery instructions
  - `list-sessions.sh` — Scan Gemini CLI session files for matching projects
  - `read-session.sh` — Extract transcript from Gemini CLI session JSON
- **`references/blog-writing-guide.md`** — Voice, collaboration vocabulary, transcript extraction patterns, blog structure

### Publishing Platforms (load only the relevant one)
- **`references/publishing/hashnode/`** — Hashnode publishing reference + script
  - `hashnode.md` — GraphQL API endpoint, authentication, `publishPost` mutation, required env vars
  - `publish.sh` — Publish a markdown file to Hashnode, outputs the post URL

### Examples
- **`examples/builders-log.md`** — Builder's log style output
- **`examples/tutorial.md`** — Tutorial style output
- **`examples/technical.md`** — Technical deep-dive output

### Assets
- **`assets/devlog-template.md`** — Blog skeleton template
