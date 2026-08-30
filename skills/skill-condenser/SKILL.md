---
name: skill-condenser
description: 使用具有技能感知格式的 Chain-of-Density 压缩冗长的 SKILL.md 文件。当技能超过 200 行或需要简洁重构时使用。
license: Apache-2.0
metadata:
  author: agentic-insights
  version: '1.0'
tags:
- 开发
---

# Skill Condenser

使用具有技能格式感知能力的 CoD 压缩 SKILL.md 文件。针对 2-3 次传递进行优化（不是 5 次），因为技能是结构化的，而不是散文。

## 何时使用

- SKILL.md 超过 200 行
- 技能包含散文段落而不是项目符号
- 将冗长的文档重构为简洁风格

## 过程

1. 阅读要压缩的技能
2. 使用技能格式上下文运行 2-3 次 `cod-iteration` 迭代
3. 每次迭代：提取关键实体，压缩为项目符号/表格
4. 输出：保持结构的压缩技能

## 编排

### 迭代 1：结构提取

传递给 `cod-iteration`：

```
iteration: 1
target_words: [current_words * 0.6]
format_context: |
  OUTPUT FORMAT: Agent Skills SKILL.md
  - Use ## headers for sections
  - Bullet lists, not prose paragraphs
  - Tables for comparisons/options
  - Code blocks for commands
  - No filler phrases ("this skill helps you...")

text: [FULL SKILL.MD CONTENT]
```

### 迭代 2：实体密集化

```
iteration: 2
target_words: [iteration_1_words]
format_context: |
  SKILL.md TERSE RULES:
  - Each bullet = one fact
  - Combine related bullets with semicolons
  - Remove redundant examples (keep 1 best)
  - Tables compress better than lists for options

text: [ITERATION 1 OUTPUT]
source: [ORIGINAL SKILL.MD]
```

### 迭代 3（可选）：最终润色

仅当仍然 >150 行时：

```
iteration: 3
target_words: [iteration_2_words]
format_context: |
  FINAL PASS:
  - Move detailed content to references/ links
  - Keep only: Quick Start, Core Pattern, Troubleshooting
  - Each section <20 lines

text: [ITERATION 2 OUTPUT]
source: [ORIGINAL SKILL.MD]
```

## 预期输出格式

每次迭代返回：

```
Missing_Entities: "entity1"; "entity2"; "entity3"

Denser_Summary:
---
name: skill-name
description: ...
---
# Skill Name
[Condensed content in proper SKILL.md format]
```

## 技能特定实体

压缩技能时，优先处理这些实体类型：

| Entity Type | Keep | Remove |
|-------------|------|--------|
| Commands | `deploy.py --env prod` | Verbose explanations |
| Options | Table row | Paragraph per option |
| Errors | `Error → Fix` | Long troubleshooting prose |
| Examples | 1 best example | Multiple similar examples |
| Prerequisites | Bullet list | Explanation of why needed |

## 目标压缩

| Original | Target | Iterations |
|----------|--------|------------|
| 200-300 lines | 100-150 | 2 |
| 300-500 lines | 150-200 | 2-3 |
| 500+ lines | 200 + refs | 3 + refactor |

## 示例：压缩冗长部分

**Before** (45 words):
```markdown
## Configuration
The configuration system allows you to customize various aspects of the deployment.
You can set environment variables, adjust timeouts, and configure retry behavior.
Each setting has sensible defaults but can be overridden as needed.
```

**After** (18 words):
```markdown
## Configuration
| Setting | Default | Override |
|---------|---------|----------|
| `ENV` | prod | `--env dev` |
| `TIMEOUT` | 30s | `--timeout 60` |
| `RETRIES` | 3 | `--retries 5` |
```

## 与渐进式披露的集成

如果技能在 3 次迭代后仍然太大：

1. 保留在 SKILL.md 中：概述、快速开始、常见错误
2. 移动到 `references/`：API 详细信息、高级配置、示例
3. 使用链接更新 SKILL.md：`See [advanced config](references/config.md)`

## 约束

- 完全保留 frontmatter（不要压缩元数据）
- 保留所有 ## 节标题（结构很重要）
- 不要删除代码块（命令是实体）
- 每个工作流保持一个具体示例
