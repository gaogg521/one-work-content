---
name: skill-writer
description: 为 ClawdHub/MoltHub 编写高质量的代理技能（SKILL.md 文件）。在从头开始创建新技能、构建技能内容、编写有效的 frontmatter 和描述、选择章节模式或遵循代理可消费技术文档的最佳实践时使用。
metadata:
  clawdbot:
    emoji: ✍️
    requires:
      anyBins:
      - npx
    os:
    - linux
    - darwin
    - win32
tags:
- 文档
---

# Skill Writer

为 ClawdHub 注册表编写结构良好、有效的 SKILL.md 文件。涵盖技能格式规范、frontmatter 模式、内容模式、示例质量和常见反模式。

## 何时使用

- 从头开始创建新技能
- 将技术内容构建为代理技能
- 编写注册表正确索引的 frontmatter
- 为不同类型的技能选择章节组织
- 在发布前审查你自己的技能

## SKILL.md 格式

技能是具有 YAML frontmatter 的单个 Markdown 文件。代理按需加载它并遵循其指令。

```markdown
---
name: my-skill-slug
description: One-sentence description of when to use this skill.
metadata: {"clawdbot":{"emoji":"🔧","requires":{"anyBins":["tool1","tool2"]},"os":["linux","darwin","win32"]}}
---

# Skill Title

One-paragraph summary of what this skill covers.

## When to Use

- Bullet list of trigger scenarios

## Main Content Sections

### Subsection with examples

Code blocks, commands, patterns...

## Tips

- Practical advice bullets
```

## Frontmatter Schema

### `name` (required)

技能的 slug 标识符。必须与你发布时使用的匹配。

```yaml
name: my-skill
```

规则：
- 小写，连字符分隔：`csv-pipeline`、`git-workflows`
- 无空格，无下划线
- 保持简短和描述性（1-3 个单词）
- 发布前检查 slug 冲突：`npx molthub@latest search "your-slug"`

### `description` (required)

最重要的字段。这是：
1. 注册表用于语义搜索索引的内容（向量嵌入）
2. 代理读取以决定是否激活技能的内容
3. 用户在浏览搜索结果时看到的内容

```yaml
# GOOD: Specific triggers and scope
description: Write Makefiles for any project type. Use when setting up build automation, defining multi-target builds, managing dependencies between tasks, creating project task runners, or using Make for non-C projects (Go, Python, Docker, Node.js). Also covers Just and Task as modern alternatives.

# BAD: Vague, no triggers
description: A skill about Makefiles.

# BAD: Too long (gets truncated in search results)
description: This skill covers everything you need to know about Makefiles including variables, targets, prerequisites, pattern rules, automatic variables, phony targets, conditional logic, multi-directory builds, includes, silent execution, and also covers Just and Task as modern alternatives to Make for projects that use Go, Python, Docker, or Node.js...
```

有效描述的模式：
```
[What it does]. Use when [trigger 1], [trigger 2], [trigger 3]. Also covers [related topic].
```

### `metadata` (required)

带有 `clawdbot` 模式的 JSON 对象：

```yaml
metadata: {"clawdbot":{"emoji":"🔧","requires":{"anyBins":["make","just"]},"os":["linux","darwin","win32"]}}
```

字段：
- **`emoji`**：在注册表列表中显示的单个表情符号
- **`requires.anyBins`**：技能需要的 CLI 工具数组（至少一个必须可用）
- **`os`**：支持的平台数组：`"linux"`、`"darwin"` (macOS)、`"win32"` (Windows)

仔细选择 `requires.anyBins`：
```yaml
# Good: lists the actual tools the skill's commands use
"requires": {"anyBins": ["docker", "docker-compose"]}

# Bad: lists generic tools every system has
"requires": {"anyBins": ["bash", "echo"]}

# Good for skills that work via multiple tools
"requires": {"anyBins": ["make", "just", "task"]}
```

## 内容结构

### "When to Use" 部分

始终在此标题段落之后立即包含此内容。它告诉代理（和用户）此技能适用的特定场景。

```markdown
## When to Use

- Automating build, test, lint, deploy commands
- Defining dependencies between tasks (build before test)
- Creating a project-level task runner
- Replacing long CLI commands with short targets
```

规则：
- 4-8 个项目符号
- 每个项目符号都是一个具体的场景，而不是抽象的概念
- 以动词或动名词开头："Automating..."、"Debugging..."、"Converting..."
- 不要逐字重复描述字段

### 主要内容部分

按任务组织，而不是按概念。代理需要找到针对特定情况的正确命令。

```markdown
## GOOD: Organized by task
## Encode and Decode
### Base64
### URL Encoding
### Hex

## BAD: Organized by abstraction
## Theory of Encoding
## Encoding Types
## Advanced Topics
```

### 代码块

每个部分都应该至少有一个代码块。没有代码块的技能是观点，而不是工具。

````markdown
## GOOD: Concrete, runnable example
```bash
# Encode a string to Base64
echo -n "Hello, World!" | base64
# SGVsbG8sIFdvcmxkIQ==
```

## BAD: Abstract description
Base64 encoding converts binary data to ASCII text using a 64-character alphabet...
````

代码块最佳实践：
- **始终指定语言**（`bash`、`python`、`javascript`、`yaml`、`sql` 等）
- **在命令下方以注释形式显示输出**
- **使用 realistic values**，而不是 `foo`/`bar`（使用 `myapp`、`api-server`、真实 IP 格式）
- **首先包含最常见的情况**，然后是变体
- **为不明显的标志或参数添加内联注释**

### 多语言覆盖

如果技能跨语言适用，请使用一致的章节结构：

```markdown
## Hashing

### Bash
```bash
echo -n "Hello" | sha256sum
```

### JavaScript
```javascript
const crypto = require('crypto');
crypto.createHash('sha256').update('Hello').digest('hex');
```

### Python
```python
import hashlib
hashlib.sha256(b"Hello").hexdigest()
```
```

顺序：Bash 优先（最通用），然后按主题的流行程度排序。

### "Tips" 部分

在每个技能末尾以 Tips 部分结束。这些是提炼的智慧 —— 节省数小时调试时间的东西。

```markdown
## Tips

- The number one Makefile bug: using spaces instead of tabs for indentation.
- SHA-256 is the standard for integrity checks. MD5 is fine for dedup but broken for cryptographic use.
- Never schedule critical cron jobs between 1:00-3:00 AM if DST applies.
```

规则：
- 5-10 个项目符号
- 每个提示都是一个独立的见解（不依赖于其他提示）
- 优先考虑陷阱和非明显行为，而不是基本建议
- 没有 "始终使用最佳实践" 的陈词滥调

## 技能类型和模板

### CLI 工具参考

用于关于特定工具或命令系列的技能。

```markdown
---
name: tool-name
description: [What tool does]. Use when [scenario 1], [scenario 2].
metadata: {"clawdbot":{"emoji":"🔧","requires":{"anyBins":["tool-name"]}}}
---

# Tool Name

[One paragraph: what it does and why you'd use it.]

## When to Use
- [4-6 scenarios]

## Quick Reference
[Most common commands with examples]

## Common Operations
### [Operation 1]
### [Operation 2]

## Advanced Patterns
### [Pattern 1]

## Troubleshooting
### [Common error and fix]

## Tips
```

### 语言/框架参考

用于关于特定语言或框架中的模式的技能。

```markdown
---
name: pattern-name
description: [Pattern] in [language/framework]. Use when [scenario 1], [scenario 2].
metadata: {"clawdbot":{"emoji":"📐","requires":{"anyBins":["runtime"]}}}
---

# Pattern Name

## When to Use

## Quick Reference
[Cheat sheet / syntax summary]

## Patterns
### [Pattern 1 — with full example]
### [Pattern 2 — with full example]

## Cross-Language Comparison (if applicable)

## Anti-Patterns
[What NOT to do, with explanation]

## Tips
```

### 工作流/流程指南

用于关于多步流程的技能。

```markdown
---
name: workflow-name
description: [Workflow description]. Use when [scenario 1], [scenario 2].
metadata: {"clawdbot":{"emoji":"🔄","requires":{"anyBins":["tool1","tool2"]}}}
---

# Workflow Name

## When to Use

## Prerequisites
[What needs to be set up first]

## Step-by-Step
### Step 1: [Action]
### Step 2: [Action]
### Step 3: [Action]

## Variations
### [Variation for different context]

## Troubleshooting

## Tips
```

## 反模式

### 过于抽象

```markdown
# BAD
## Error Handling
Error handling is important for robust applications. You should always
handle errors properly to prevent unexpected crashes...

# GOOD
## Error Handling
```bash
# Bash: exit on any error
set -euo pipefail

# Trap for cleanup on exit
trap 'rm -f "$TMPFILE"' EXIT
```
```

### 过于狭窄

```markdown
# BAD: Only useful for one specific case
---
name: react-useeffect-cleanup
description: How to clean up useEffect hooks in React
---

# GOOD: Broad enough to be a real reference
---
name: react-hooks
description: React hooks patterns. Use when working with useState, useEffect, useCallback, useMemo, custom hooks, or debugging hook-related issues.
---
```

### 没有示例的文本墙

如果任何部分超过 10 行而没有代码块，则文本太重。用示例分解它。

### 缺少交叉引用

如果你的技能提到另一个具有自己技能的工具或概念，请记下它：

```markdown
# For Docker networking issues, see the `container-debug` skill.
# For regex syntax details, see the `regex-patterns` skill.
```

### 过时的命令

验证每个命令在当前工具版本上是否有效。常见陷阱：
- Docker Compose: `docker-compose` (v1) vs. `docker compose` (v2)
- Python: `pip` vs. `pip3`, `python` vs. `python3`
- Node.js: CommonJS (`require`) vs. ESM (`import`)

## 大小指南

| Metric | Target | Too Short | Too Long |
|---|---|---|---|
| Total lines | 300-550 | < 150 | > 700 |
| Sections | 5-10 | < 3 | > 15 |
| Code blocks | 15-40 | < 8 | > 60 |
| Tips | 5-10 | < 3 | > 15 |

低于 150 行的技能可能缺少示例。超过 700 行的技能应该分成两个技能。

## 发布清单

在发布之前，验证：

1. **Frontmatter 是有效的 YAML** —— 通过粘贴到 YAML 验证器中进行测试
2. **描述以技能的作用开头** —— 不是 "This skill..." 或 "A skill for..."
3. **每个部分至少有一个代码块** —— 主要内容中没有纯文本部分
4. **命令实际上有效** —— 在干净的环境中测试
5. **没有留下占位符值** —— 搜索 `TODO`、`FIXME`、`example.com` 用作真实 URL
6. **Slug 可用** —— `npx molthub@latest search "your-slug"` 返回无精确匹配
7. **`requires.anyBins` 列出真正的依赖项** —— 技能命令实际调用的工具
8. **存在 Tips 部分** —— 包含 5+ 可操作的、非明显的项目符号

## 发布

```bash
# 发布新技能
npx molthub@latest publish ./skills/my-skill \
  --slug my-skill \
  --name "My Skill" \
  --version 1.0.0 \
  --changelog "Initial release"

# 更新现有技能
npx molthub@latest publish ./skills/my-skill \
  --slug my-skill \
  --name "My Skill" \
  --version 1.1.0 \
  --changelog "Added new section on X"

# 验证它已发布
npx molthub@latest search "my-skill"
```

## Tips

- `description` 字段是你的技能搜索排名。在其上花费比任何单个内容部分更多的时间。包括用户会搜索的特定动词和名词。
- 以最常见的用例开头。如果 80% 的用户需要 "how to encode Base64"，请将其放在 "how to convert between MessagePack and CBOR" 之前。
- 每个代码示例都应该是可复制的。如果它需要未显示的设置，请添加设置。
- 为代理编写，而不是为人类。代理需要它可以逐步遵循的明确指令。避免 "you might want to consider" —— 说 "do X when Y"。
- 通过要求代理在真实任务上使用它来测试你的技能。如果代理无法遵循指令产生正确的结果，则该技能需要改进。
- 即使对于语言特定的技能，也优先使用 `bash` 代码块来执行命令。代理通常通过 shell 操作，bash 块表示 "run this"。
- 不要重复 `--help` 已经提供的内容。专注于模式、组合和 `--help` 不教的非明显内容。
- 语义化地版本化你的技能：补丁用于拼写错误修复，次要用于新部分，主要用于重组。注册表跟踪版本历史。
