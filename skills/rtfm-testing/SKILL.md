---
name: rtfm-testing
description: 文档质量验证方法论。通过生成零上下文新代理(agent)来测试文档是否真正可用，识别缺失步骤与错误假设。触发词：文档测试(doc testing)、RTFM、文档质量(documentation quality)、零上下文测试(cold start test)、可用性验证
tags:
- AI
- MongoDB
- 代码审查
- 测试
---

# RTFM Testing

一种文档质量方法论，通过生成新的 agent 来验证文档是否真正可用。

## The Problem

由构建者本人编写的文档几乎总是不完整的。他们会无意识地填补空白。他们假设了上下文。他们跳过了“显而易见”的步骤。

RTFM Testing 通过生成一个零上下文的新 agent 并提问：你能否仅使用文档完成此任务？来解决这个问题。

## When to Use

- Before publishing docs, READMEs, tutorials, or setup guides
- When users report confusion but you can't see why
- After major refactors to validate docs still work
- As part of CI for documentation-heavy projects

## How It Works

1. **Identify the task** — 某人在阅读文档后应该能够完成什么？
2. **Bundle the docs** — 收集所有相关文档（且仅收集这些）
3. **Spawn a fresh tester** — 使用 `sessions_spawn` 运行 TESTER.md prompt
4. **Analyze failures** — 每一个困惑点都是一个文档 bug
5. **Fix and repeat** — 更新文档，重新生成，重新测试，直到通过

## Usage

```
sessions_spawn(
  task: "Complete the following task using ONLY the provided documentation. [TASK DESCRIPTION]\n\n---\n\n[PASTE DOCS HERE]",
  agentId: "default",
  label: "rtfm-test"
)
```

Or use the full TESTER.md prompt for more structured output.

## Metrics

- **Cold Start Score** — 直到任务完成的生成周期数（越低 = 文档越好）
- **Gap Count** — 每次运行的 `[GAP]` 报告数量
- **Gap Categories** — Missing steps, unclear language, wrong assumptions, missing prerequisites

## Key Principles

1. **No hints** — 不要帮助 tester。让它失败。
2. **Literal reading** — Tester 不得推断或猜测
3. **Docs only** — 没有外部知识，没有“常识”
4. **Failures are signal** — 每一次绊倒都是可操作的反馈

## Files

- `SKILL.md` — 本文件
- `TESTER.md` — 新 agent 的 system prompt
- `GAPS.md` — 输出格式规范
