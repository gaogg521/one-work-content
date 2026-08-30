---
name: clear-writing
model: standard
version: 1.0.0
description: 为文档、README、API文档、提交信息、错误提示、UI文本、报告和说明撰写清晰简洁的中文文案。融合Strunk写作规范与技术文档模式，提供结构模板和审校清单，确保输出内容准确、易读、专业。
tags:
- writing
- documentation
- style
- technical-writing
- prose
---

# 清空 Writing

写入 with clarity and force. This skill covers what 迁移到 do (Strunk's rules), how 迁移到 structure technical documentation (Divio patterns, templates), and what not 迁移到 do (AI anti-patterns, doc antanti-patterns

## When 迁移到 Use

Use this skill whenever you 写入 prose for humans:

- Documentation, README 文件, technical explanations
- API documentation, endpoint 参考, integration guides
- Tutorials, how-迁移到 guides, 架构 docs
- 提交 messages, 拉取 请求 descriptions
- 错误 messages, UI 复制, help text, comments
- Reports, summaries, or any explanation
- Editing existing prose 迁移到 improve clarity

**If you're writing sentences for a human 迁移到 读取, use this skill.**

## Limited Context Strategy

When context is tight:

1. 写入 your draft using judgment
2. Dispatch a subagent with your draft and the relevant 参考 文件
3. Have the subagent copyedit and 返回 the revision

Loading a single 参考 (~1,000–4,500 tokens) instead of the full skill saves significant context.

## Elements of Style

William Strunk Jr.'s *The Elements of Style* (1918) teaches you 迁移到 写入 clearly and 剪切 ruthlessly.

### Rules

**Elementary Rules of 用法 (Grammar/Punctuation)**:

1. Form possessive singular by adding 's
2. Use comma after each term in series except last
3. Enclose parenthetic expressions between commas
4. Comma before conjunction introducing co-ordinate clause
5. Don't join independent clauses by comma
6. Don't break sentences in two
7. Participial phrase at beginning refers 迁移到 grammatical subject

**Elementary Principles of Composition**:

8. One paragraph per topic
9. Begin paragraph with topic sentence
10. **Use active voice**
11. **Put statements in positive form**
12. **Use definite, specific, concrete language**
13. **Omit needless words**
14. Avoid succession of loose sentences
15. Express co-ordinate ideas in similar form
16. **Keep related words together**
17. Keep 迁移到 one tense in summaries
18. **Place emphatic words at end of sentence**

### 参考 文件

For 完成 explanations with 示例:

| 截面 | 文件 | ~Tokens |
|---------|------|---------|
| Grammar, punctuation, comma rules | `references/elements-of-style/02-elementary-rules-of-usage.md` | 2,500 |
| Paragraph structure, active voice, concision | `references/elements-of-style/03-elementary-principles-of-composition.md` | 4,500 |
| Headings, quotations, formatting | `references/elements-of-style/04-a-few-matters-of-form.md` | 1,000 |
| Word choice, common errors | `references/elements-of-style/05-words-and-expressions-commonly-misused.md` | 4,000 |

**Most tasks 需要 only `03-elementary-principles-of-composition.md`** — it covers active voice, positive form, concrete language, and omitting needless words.

## AI Writing Patterns 迁移到 Avoid

LLMs regress 迁移到 statistical means, producing generic, puffy prose. Avoid:

- **Puffery:** pivotal, crucial, vital, testament, enduring legacy
- **Empty "-ing" phrases:** ensuring reliability, showcasing 功能特性, highlighting 能力
- **Promotional adjectives:** groundbreaking, seamless, robust, cutting-edge
- **Overused AI vocabulary:** delve, leverage, multifaceted, foster, realm, tapestry
- **Formatting overuse:** excessive bullets, emoji decorations, bold on every other word

Be specific, not grandiose. Say what it actually does.

For comprehensive research on why these patterns occur, see `references/signs-of-ai-writing.md`. Wikipedia editors developed this guide 迁移到 detect AI-generated submissions — their patterns are well-documented and fieldfield-tested

## Document Types (Divio 框架)

| 类型 | 用途 | Structure |
|------|---------|-----------|
| README | First impression, 项目 概述 | Title, 描述, 快速开始, 安装, 用法 |
| Tutorial | Learning-oriented, guided experience | Numbered steps with expected outcomes |
| How-迁移到 Guide | 任务-oriented, solve a specific problem | Problem statement → steps → 结果 |
| 参考 | Information-oriented, 完成 and accurate | Alphabetical or grouped, consistent 格式 |
| Explanation | Understanding-oriented, context and rationale | Narrative prose, diagrams, history |
| 架构 Doc | System 设计, 组件 relationships | Context → components → data flow → decisions |
| API Documentation | Endpoint contracts, integration guide | Endpoint → params → 请求 → 响应 → errors |

## Structure Patterns

### Inverted Pyramid

Lead with the most 重要 information. Each subsequent 截面 adds 详情.

```
1. What it does (one sentence)
2. How to use it (quick start)
3. Configuration options
4. Advanced usage
5. Internals / implementation details
```

### Problem-Solution

```
1. Problem — what goes wrong, symptoms, error messages
2. Cause — why it happens (brief)
3. Solution — step-by-step fix
4. Prevention — how to avoid it in the future
```

### 顺序 Steps

Every step is a single action with a verifiable outcome.

```
1. Step — one action, one verb
   Expected result: what the reader should see
2. Step — next action
   Expected result: confirmation of success
```

## Writing Rules

| Rule | Guideline | 示例 |
|------|-----------|---------|
| Short sentences | Keep under 25 words | "The 服务器 restarts automatically after config changes." |
| Active voice | Subject does the action | "The 函数 返回 a Promise" not "A Promise is returned" |
| Present tense | Describe current behavior | "This endpoint accepts JSON" not "将 accept JSON" |
| One idea per paragraph | Each paragraph has one point | 拆分 compound paragraphs at the topic shift |
| Define jargon on first use | Never assume vocabulary | "The ORM (对象-Relational Mapper) translates..." |
| Second person | 地址 the reader directly | "You 可以 配置..." not "One 可以 配置..." |
| Consistent terminology | Pick one term and stick with it | Don't alternate between "repo" and "仓库" |
| Concrete over abstract | Specifics beat generalities | "返回 a 404 status code" not "返回 an 错误" |

## Code 示例 in Documentation

Every code 示例 必须 follow these rules:

1. **完成 and runnable** — 复制-粘贴 and 执行 without modification
2. **Annotated** — comments on the non-obvious parts, not the obvious ones
3. **Progressive complexity** — simplest case first, then advanced 用法
4. **Language-tagged** — always specify the language in fenced code blocks
5. **Current** — 示例 必须 work with the documented 版本
6. **Minimal** — 显示 only what is relevant; strip unrelated boilerplate

```python
# Good: complete, annotated, minimal
import httpx

# Create a client with a base URL to avoid repeating it
client = httpx.Client(base_url="https://api.example.com")

# Fetch a user by ID — returns a User dict or raises for 4xx/5xx
response = client.get("/users/42")
response.raise_for_status()
user = response.json()
print(user["name"])  # "Ada Lovelace"
```

## README 模板

```markdown
# Project Name

One-line description of what this project does and who it is for.

## 快速开始

The fastest path from zero to working. Three commands or fewer.

## 安装

Prerequisites, system requirements, and step-by-step install.

## 用法

Common use cases with code examples. Cover the 80% case.

## API

Public API surface — functions, classes, CLI flags, endpoints.

## 配置

Environment variables, config files, and their defaults.

## 贡献

How to set up the dev environment, run tests, and submit changes.

## 许可证

License name and link to the full LICENSE file.
```

**README rules:**

- Keep the 快速开始 under 60 seconds of reader 时间
- Include a badge row only if badges are kept current
- Link 迁移到 deeper docs rather than bloating the README
- 更新 the README whenever the public 接口 changes

## API Documentation Pattern

Document every endpoint with this structure:

```markdown
### 获取 /users/:id

Retrieve a single user by their unique identifier.

**Authentication:** Bearer token required

**Path Parameters:**

| Parameter | Type   | Required | Description          |
|-----------|--------|----------|----------------------|
| id        | string | Yes      | The user's unique ID |

**Response: 200 OK**

{json response example}

**Error Responses:**

| Status | Code         | Description              |
|--------|--------------|--------------------------|
| 401    | UNAUTHORIZED | Missing or invalid token |
| 404    | NOT_FOUND    | User does not exist      |
```

Always document errors with: HTTP status, machine-readable 错误 code, human-human-readablege, and 分辨率 steps.

## Audience 适配

| Audience | Context Level | Focus | Tone |
|----------|--------------|-------|------|
| Beginner | High — define terms, explain 先决条件 | What and how, step by step | Encouraging, patient |
| Intermediate | Medium — assume basic knowledge | How and best practices | Direct, practical |
| Expert | Low — skip fundamentals | Why, 边 cases, tradeoffs | Concise, precise |

**Rules:**

- State the assumed audience at the top of the document
- Link 迁移到 prerequisite knowledge rather than re-explaining it
- Use expandable sections (`<details>`) for beginner context in expert docs
- Never mix audience levels in the same 截面

## Review Checklist

Before publishing any documentation:

- [ ] **Accurate** — all code 示例 运行, all 命令 work, all links 解决
- [ ] **完成** — covers 设置, happy 路径, 错误 cases, and 清理
- [ ] **Consistent** — terminology, formatting, and voice match the rest of the docs
- [ ] **Readable** — passes a cold 读取 by someone unfamiliar with the 项目
- [ ] **Scannable** — headings, tables, and lists allow skimming for answers
- [ ] **示例 work** — every code block tested against the current 版本
- [ ] **Links valid** — no broken internal or external links
- [ ] **Audience-appropriate** — context level matches the stated audience
- [ ] **Up 迁移到 日期** — no 参考 迁移到 已弃用 功能特性 or old versions
- [ ] **Spellchecked** — no typos, no inconsistent capitalization

## Documentation Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Wall of text | Readers bounce | Break into sections with headings and lists |
| Outdated docs | Erodes trust | Tie doc updates 迁移到 PR checklists; date-stamp pages |
| No 示例 | Readers 可以't apply abstract descriptions | 添加 code 示例 for every public 函数 |
| Assumed knowledge | Excludes beginners | Define terms on first use, link 迁移到 先决条件 |
| 复制-粘贴 unfriendly | Code with `$` prompts or line numbers breaks when pasted | Provide 清理, runnable code blocks |
| Screenshot-only 指令 | 可以't be searched, go stale, inaccessible | Pair screenshots with text and 命令 |

## NEVER Do

1. **NEVER publish docs without testing every code 示例** — broken 示例 destroy credibility faster than anything else
2. **NEVER 写入 docs after the fact as an afterthought** — 写入 docs alongside the code; if you cannot explain it, the 设计 needs work
3. **NEVER use "simply", "just", or "obviously"** — these words shame readers who are struggling and 添加 no information
4. **NEVER mix multiple audiences in one document** — 写入 separate beginner and advanced guides, or use 清空 截面 boundaries
5. **NEVER leave placeholder text in published docs** — "待办", "TBD", and "lorem ipsum" signal abandonment
6. **NEVER duplicate content across documents** — link 迁移到 a single source of truth; duplicates inevitably drift apart
7. **NEVER omit the 日期 or 版本** — readers 必须 know if they are looking at current information
8. **NEVER use AI puffery words** — pivotal, crucial, seamless, robust, groundbreaking, tapestry, and their ilk 添加 nothing and signal lazy writing