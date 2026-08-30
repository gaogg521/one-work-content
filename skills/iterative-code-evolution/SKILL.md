---
name: iterative-code-evolution
description: 通过结构化的分析(analyze)-变异(mutate)-评估(evaluate)循环系统地改进代码。改编自 ALMA，用于迭代代码质量、优化实现、调试顽固问题或演进设计。触发词：代码演进(code evolution)、迭代优化(iterative optimization)、代码质量(code quality)、ALMA
tags:
- 代码审查
- 性能优化
---

# Iterative Code Evolution

一种通过严谨的 reflect → mutate → verify → score 循环来改进代码的结构化方法，改编自 ALMA 研究框架，用于 meta-learning code designs。

## When to Use This Skill

- 迭代表现不够好（性能、正确性、设计）的代码
- 在多轮变更中优化实现
- 调试简单修复不断失败的顽固或反复出现的问题
- 通过结构化实验演进系统设计
- 任何你已经尝试了 2 种以上方法并需要关于下一步尝试什么的纪律的任务
- 构建或改进 prompts、pipelines、agents 或任何受益于迭代优化的 "program"

## When NOT to Use This Skill

- 简单的一次性代码生成（直接写）
- 有明确解决方案的机械性任务（refactoring、formatting、migrations）
- 当用户已经明确指定了要更改什么时

## Core Concepts

### The Evolution Loop

每个改进循环遵循以下序列：

```
┌─────────────────────────────────────────────────────┐
│  1. ANALYZE  — 对当前代码进行结构化诊断              │
│  2. PLAN     — 优先的、具体的变更                    │
│  3. MUTATE   — 实施变更                              │
│  4. VERIFY   — 运行它，检查错误                      │
│  5. SCORE    — 衡量相对于 baseline 的改进            │
│  6. ARCHIVE  — 记录尝试了什么以及发生了什么          │
│                                                     │
│  带着新知识回到 1                                    │
└─────────────────────────────────────────────────────┘
```

### The Evolution Log

在项目根目录的 `.evolution/log.json` 中跟踪所有迭代。这是使每个循环都比上一个更智能的 memory。

```json
{
  "baseline": {
    "description": "Initial implementation before evolution began",
    "score": 0.0,
    "timestamp": "2025-01-15T10:00:00Z"
  },
  "variants": {
    "v001": {
      "parent": "baseline",
      "description": "Added input validation and error handling",
      "changes_made": [
        {
          "what": "Added type checks on all public methods",
          "why": "Runtime crashes from malformed input in 3/10 test cases",
          "priority": "High"
        }
      ],
      "score": 0.6,
      "delta": "+0.6 vs parent",
      "timestamp": "2025-01-15T10:30:00Z",
      "learned": "Input validation was the primary failure mode — most other logic was sound"
    },
    "v002": {
      "parent": "v001",
      "description": "Refactored parsing logic to handle edge cases",
      "changes_made": [
        {
          "what": "Rewrote parse_input() to use state machine instead of regex",
          "why": "Regex approach failed on nested structures (seen in test cases 7,8)",
          "priority": "High"
        }
      ],
      "score": 0.85,
      "delta": "+0.25 vs parent",
      "timestamp": "2025-01-15T11:00:00Z",
      "learned": "State machine approach generalizes better than regex for this grammar"
    }
  },
  "principles_learned": [
    "Input validation fixes give the biggest early gains",
    "Regex-based parsing breaks on recursive structures — prefer state machines",
    "Small targeted changes score better than large rewrites"
  ]
}
```

## The Process in Detail

### Phase 1: ANALYZE — Structured Diagnosis

在更改任何东西之前，对当前代码及其输出进行结构化分析。这是最重要的阶段 — 它可以防止浪费的 mutations。

**Step 1 — 从过去的编辑中学习**（第一次迭代时跳过）

回顾 evolution log。对于每个之前的变更：
- score 是提高了还是降低了？
- 什么模式使它成功或失败？
- 提取 2-3 个要采纳的 principles 和 2-3 个要避免的 pitfalls

**Step 2 — 组件级评估**

对于每个有意义的组件（function、class、module、pipeline stage），为其打标签：

| Label | Meaning |
|-------|---------|
| **Working** | 产生正确的输出，未观察到问题 |
| **Fragile** | 在 happy path 上工作，但在 edge cases 或特定输入上失败 |
| **Broken** | 产生错误的输出或错误 |
| **Redundant** | 在其他地方重复了逻辑，增加了复杂度而没有价值 |
| **Missing** | 一个需要的组件但尚不存在 |

对于每个标签，写一行 *why* 的解释 — 关联到特定的 test outputs 或观察到的行为。

**Step 3 — 质量和一致性检查**

寻找跨领域问题：
- **Data flow**：组件之间是否传递结构化数据，还是依赖隐式状态？
- **Error handling**：错误是被捕获并处理，还是被静默吞掉？
- **Duplication**：相同的逻辑是否在多个地方重复？
- **Hardcoding**：是否存在 magic numbers、hardcoded paths 或 environment-specific assumptions？
- **Generalization**：哪些部分适用于新输入，哪些部分 overfitted 到 test cases？

**Step 4 — 产生优先建议**

基于 Steps 1-3，产生具体的变更。每个建议必须有：

```
- PRIORITY: High | Medium | Low
- WHAT: 变更的精确描述（代码级别，不模糊）
- WHY: 关联到 Steps 1-3 中的特定观察
- RISK: 如果此变更实施不正确，可能会出什么问题
```

**规则：每个建议必须关联到一个观察。** 不允许 "this might help" 的建议 — 只允许基于你在代码或输出中实际看到的内容的变更。

**规则：每轮限制为 3 个建议。** 一次超过 3 个变更会使无法将改进或回归归因于特定变更。

### Phase 2: PLAN — Select What to Change

从分析中挑选 1-3 个建议。选择原则：

- **High priority first** — 先修复 broken 的东西，再优化 working 的东西
- **One theme per cycle** — 不要混合不相关的变更（例如，不要在同一次 mutation 中修复 parsing AND refactor error handling）
- **Prefer targeted over sweeping** — 对一个 function 的手术式变更胜过三个 module 的重写
- **If stuck, explore** — 如果最后 2 轮以上循环在同一组件上显示收益递减，选择一个不同的组件来修改（这是 ALMA 的 "visit penalty" 原则 — 不要在同一件事上持续打磨）

### Phase 3: MUTATE — Implement Changes

编写新代码。关键纪律：

- **只更改计划中的内容。** 抵制住 "再修复一件事" 的冲动。
- **Preserve interfaces.** 除非计划明确要求，否则不要更改 function signatures 或 return types。
- **Comment the rationale.** 在每个变更附近添加简短的注释，引用 evolution cycle（例如，`# evo-v003: switched to state machine per edge case failures`）

### Phase 4: VERIFY — Run and Check

针对用于评分的相同 inputs/tests 执行修改后的代码。

**If it crashes (up to 3 retries):**

使用 reflection-fix protocol：
1. 阅读完整的 error traceback
2. 识别 **root cause**（不是 symptom）
3. 只修复 **root cause** — 不要进行不相关的改进
4. Re-run

3 次重试失败后，**revert to parent variant** 并记录失败：
```json
{
  "attempted": "Description of what was tried",
  "failure_mode": "The error that couldn't be resolved",
  "learned": "Why this approach doesn't work"
}
```

这些失败数据很有价值 — 它可以防止重新尝试相同的 broken approach。

**If it runs but produces wrong output:**

不要立即重试。带着新的输出回到 Phase 1 (ANALYZE)。错误的输出是诊断数据。

### Phase 5: SCORE — Measure Improvement

将新 variant 的性能与其 parent 进行比较（而不仅仅是 baseline）。评分取决于上下文：

| Context | Score Method |
|---------|-------------|
| Tests exist | Pass rate: tests_passed / total_tests |
| Performance optimization | Metric delta (latency, throughput, memory) |
| Code quality | Weighted checklist (correctness, edge cases, readability) |
| User feedback | Binary: better/worse/same per the user's judgment |
| LLM/prompt output quality | Sample outputs graded against criteria |

**Always compute delta vs. parent.** 这是你了解哪些变更有帮助、哪些有伤害的方式。

### Phase 6: ARCHIVE — Log and Learn

更新 `.evolution/log.json`：
1. 记录新的 variant，包括 parent、description、changes、score、delta
2. 写一个 `learned` 字段：关于这个循环教会了你什么的一句话
3. 如果 score 提高了，将 underlying principle 添加到 `principles_learned`
4. 如果 score 降低了，将 failure mode 作为 pitfall 添加到 `principles_learned`

## Variant Management

### When to Branch vs. Modify

- **Modify in place**（同一个文件，新版本）：当变更显然是增量式的（修复 bug、添加检查、调整参数）
- **Branch**（复制到新文件）：当尝试根本不同的方法（不同的 algorithm、不同的 architecture、不同的 strategy）

将分支保留在 `.evolution/variants/` 中，并使用描述性的名称。evolution log 跟踪哪个是 active 的。

### Selection: Which Variant to Iterate On

如果你有多个 variants，使用以下方式选择下一个要改进的：

```
score(variant) = normalized_reward - 0.5 * log(1 + visit_count)
```

其中：
- `normalized_reward` = 相对于 baseline 的 variant score（0-1 范围）
- `visit_count` = 该 variant 被选中进行迭代的次数

这在 **exploitation**（迭代最佳 variant）和 **exploration**（尝试最近未被触及的 variants）之间取得了平衡。它可以防止陷入局部最优。

## Quick Reference: Analysis Template

执行 Phase 1 时，按以下方式组织你的思考：

```markdown
## Evolution Cycle [N] — Analysis

### Lessons from Previous Cycles
- Cycle [N-1] changed [X], score went [up/down] by [amount]
- Principle: [what we learned]
- Pitfall: [what to avoid]

### Component Assessment
| Component | Status | Evidence |
|-----------|--------|----------|
| function_a() | Working | All test cases pass |
| function_b() | Fragile | Fails on empty input (test #4) |
| class_C | Broken | Returns None instead of dict |

### Cross-Cutting Issues
- [Issue 1 with specific evidence]
- [Issue 2 with specific evidence]

### Planned Changes (max 3)
1. **[High]** WHAT: ... | WHY: ... | RISK: ...
2. **[Medium]** WHAT: ... | WHY: ... | RISK: ...
```

## Example: Full Evolution Cycle

**Context:** 用户要求改进一个 web scraper，它在 40% 的目标页面上失败。

**Cycle 1 — Analysis:**
- Component assessment: `parse_html()` 是 Broken（在没有 `<article>` 标签的页面上崩溃），`fetch_page()` 是 Working，`extract_links()` 是 Fragile（遗漏 relative URLs）
- Cross-cutting: 没有 error handling — 一个 bad page 会杀死整个 batch
- Past edits: 无（第一次循环）
- Plan: [High] 在 `parse_html()` 中为没有 `<article>` 的页面添加 fallback selectors

**Cycle 1 — Mutate:** 添加 cascading selector logic：尝试 `<article>`，回退到 `<main>`，回退到 `<body>`。

**Cycle 1 — Verify:** 运行无崩溃。

**Cycle 1 — Score:** Pass rate 40% → 72%。Delta: +32%。

**Cycle 1 — Archive:** Learned: "Most failures were selector misses, not logic errors. Fallback chains are high-value."

**Cycle 2 — Analysis:**
- Lessons: Fallback selectors 带来了 +32%。Principle: 在修复逻辑之前处理 structural variation。
- Component assessment: `parse_html()` 现在是 Working。`extract_links()` 仍然是 Fragile — relative URLs 未解析。
- Plan: [High] 在 `extract_links()` 中使用 `urljoin` 解析 relative URLs

**Cycle 2 — Mutate:** 添加 base URL resolution。

**Cycle 2 — Score:** 72% → 88%。Delta: +16%。

**Cycle 2 — Archive:** Learned: "URL resolution was second-biggest failure mode. Always normalize URLs at extraction time."

## Key Principles

- **Every change must link to an observation** — 没有推测性修复
- **Max 3 changes per cycle** — 准确归因改进
- **Log everything** — 失败的尝试与成功的尝试一样有价值
- **Score against parent, not just baseline** — 跟踪边际改进
- **Explore when stuck** — 如果同一组件上 2 轮以上循环显示收益递减，移动到一个不同的组件
- **Revert on 3 failed retries** — 不要陷入困境；记录失败并尝试不同的方法
- **Principles compound** — evolution log 的 `principles_learned` 列表是最有价值的 artifact；它编码了对 *这个特定代码库* 有效的方法
