---
name: pr-review
description: 在发布 PR 前查找并修复代码问题。启动 5 个并行分析 agent 进行 bug、安全、性能、指南合规性与质量检查，自动修复高置信度问题，也支持按路径审计现有代码。需要 Git。触发词：PR 审查(PR review)、代码审计(code audit)、预检(pre-check)、Git diff、代码质量(code quality)
metadata: None
openclaw: None
requires: None
bins:
- git
tags:
- Git
- AI
- 代码审查
- 安全
---

# Pre-Review

在发布 PR **之前** 查找并修复问题，而不是之后。

传统的 review 工具在已发布的 PR 上发布评论，形成一个修复然后更新的循环。Pre-review 反转了这个流程：在本地分析，直接修复，发布干净的代码。

## 用法

```
/pr-review                    # 将当前分支的变更与 main/master 对比进行 review
/pr-review src/api/ src/auth/ # 审计特定目录
/pr-review **/*.ts            # 审计匹配模式的文件
/pr-review --audit            # 通过智能优先级审计整个代码库
```

两种模式，一个命令：

| 模式 | 触发条件 | 范围 | 修复阈值 | 最适合 |
|------|---------|-------|---------------|----------|
| **Diff** | 无参数，在有变更的分支上 | 仅变更的文件 | >= 70 | 打开 PR 之前 |
| **Audit** | 路径、模式或 `--audit` | 指定文件或整个代码库 | >= 80 (保守) | 安全 review、代码库健康 |

## 指令

精确遵循以下步骤。

### Step 1: 检测模式和范围

创建一个 todo list 来跟踪所有步骤的进度。

**未提供参数：**
- 运行 `git diff main...HEAD --name-only` (如果 main 不存在则尝试 master)
- 如果存在变更：**Diff 模式** — 仅 review 分支变更
- 如果没有变更：告知用户 "No changes found. Use `/pr-review <path>` to audit existing code." 并停止

**提供了路径或模式：**
- 解析为实际文件
- 如果 > 50 个文件，询问用户缩小范围或确认
- **Audit 模式**

**`--audit` 标志：**
- 识别源目录 (src/, lib/, app/, 等)
- 排除：node_modules, dist, build, vendor, .git, coverage
- 向用户提议范围以确认
- **Audit 模式**

### Step 2: 发现项目指南 (Haiku Agent)

启动一个 Haiku agent (Task tool, model: haiku) 来查找并读取：
- 根目录 CLAUDE.md 和相关目录中的 CLAUDE.md 文件
- .eslintrc, .prettierrc, tsconfig.json, biome.json 或类似文件
- CONTRIBUTING.md、代码风格指南
- package.json (技术栈上下文)

返回指南和技术栈的摘要。

### Step 3: 构建上下文

**Diff 模式** — 启动一个 Haiku agent 来：
- 运行 `git diff main...HEAD` (完整 diff；如果 main 不存在则使用 master)
- 返回结构化摘要：变更的文件、变更性质、增加/删除的行数

**Audit 模式** — 启动一个 Haiku agent 来按风险对文件分类：

| 优先级 | 示例 |
|----------|---------|
| **High** | Auth, payments, DB queries, API endpoints, input validation, file handlers, crypto |
| **Medium** | Business logic, services, utilities, state management |
| **Low** | Tests (除非请求), config-only, types/interfaces, constants |

返回优先级文件列表。分析聚焦于高优先级和中优先级文件。

### Step 4: 并行深度分析 (5 个 Sonnet Agents)

在**单条消息中启动 5 个 Sonnet agents** (Task tool, model: sonnet, subagent_type: general-purpose)。

每个 agent 接收：来自 Step 2 的指南摘要、来自 Step 3 的上下文，以及将问题作为结构化列表返回的指令，包含文件路径、行号、严重级别 (critical/important/minor)、类别、描述、建议修复和置信度分数 (0-100)。

#### Diff 模式 — Agent 分配

**Agent #1 — 指南合规性：**
根据项目指南 (CLAUDE.md, eslint, prettier, 等) 审计变更：
- Naming conventions
- Required and forbidden code patterns
- Documentation requirements
- Anti-patterns specific to the project

**Agent #2 — Bug 扫描器：**
扫描变更的代码中的真实 bug (不是风格)：
- Logic errors, off-by-one
- Null/undefined handling
- Race conditions, resource leaks
- Error handling gaps
- Async/await mistakes

**Agent #3 — Git 历史和回归：**
读取修改文件的 git blame 和 log。当检测到回归或问题时返回发现：
- Regressions (re-introducing previously fixed bugs)
- Breaking changes to stable, long-lived code
- Patterns intentionally established in prior commits being undone
- Changes that contradict context from recent commit history

**Agent #4 — 安全和性能：**
- Injection vulnerabilities (SQL, XSS, command, path traversal)
- Exposed secrets, auth issues, unsafe operations
- N+1 queries, unnecessary loops, memory leaks, API misuse

**Agent #5 — 质量和测试：**
- Missing or inadequate tests for new functionality
- Dead code introduced
- Duplicated logic that should be extracted
- Inconsistent error handling
- Comments that became stale due to code changes

#### Audit 模式 — Agent 分配

**Agent #1 — 安全审计：**
对目标文件进行深度安全扫描：
- SQL/NoSQL injection, XSS, command injection, path traversal
- Hardcoded secrets and credentials, weak cryptography
- Authentication bypasses, authorization flaws, SSRF
- Insecure deserialization

**Agent #2 — Bug 检测：**
扫描 bug 和逻辑错误：
- Null/undefined handling, off-by-one errors
- Race conditions, resource leaks (memory, file handles, connections)
- Infinite loops, unreachable code, dead code paths
- Type coercion issues, async/await mistakes, error swallowing

**Agent #3 — 数据流分析：**
追踪数据在应用中的流动：
- Unvalidated user input reaching sensitive operations
- Data leaks (logging PII, exposing internals in errors)
- Missing input sanitization at boundaries
- Trust boundary violations

**Agent #4 — 性能和资源：**
- N+1 query patterns, missing pagination
- Unbounded loops over user-controlled data
- Memory accumulation, blocking operations in async context
- Inefficient algorithms (O(n^2) when O(n) is possible)
- Missing caching for repeated expensive operations

**Agent #5 — 代码质量和可维护性：**
- Functions exceeding 50 lines, nesting deeper than 4 levels
- High cyclomatic complexity
- Duplicated logic across files
- Inconsistent patterns, outdated idioms
- Critical TODOs and FIXMEs that indicate unfinished work

### Step 5: 去重和评分 (Haiku Agent)

启动一个 Haiku agent 来处理 Step 4 的所有结果：

1. **去重** — 移除被多个 agent 发现的问题 (相同文件 + 相同行范围 = 一个问题)
2. **合并** — 合并具有相同根本原因的相关问题
3. **重新评分** — 根据完整上下文调整置信度：

| 分数 | 含义 |
|-------|---------|
| 90-100 | Critical bug or vulnerability. Clear evidence. Must fix. |
| 70-89 | Real issue that will cause problems. Should fix. |
| 50-69 | Code smell or potential issue. Needs human judgment. |
| < 50 | Minor, stylistic, or likely false positive. |

**丢弃阈值：**
- Diff 模式：丢弃低于 70 的 (你写的，上下文是新鲜的)
- Audit 模式：丢弃低于 50 的 (更广泛地报告，保守地修复)

### Step 6: 自动修复

对达到阈值的问题直接应用修复：
- **Diff 模式：** 修复分数 **>= 70** 的问题
- **Audit 模式：** 修复分数 **>= 80** 的问题

对于每个修复：
1. 读取包含问题的文件
2. 使用 Edit tool 应用修复
3. 验证修复保留了周围代码和意图

按文件分组修复以最小化编辑。

**永远不要自动修复：**
- 需要架构变更的问题
- 具有多种有效方法的不明确修复
- 测试文件中的问题 (仅报告)
- 低于模式阈值的问题

### Step 7: 报告

生成报告并展示给用户。

**Diff 模式：**

```
## Pre-Review Complete

### Issues Found and Fixed: X

1. **file:line** - Description
   - Severity: critical/important/minor | Confidence: XX
   - Category: security/bug/performance/quality/guidelines
   - Fix applied: What was changed

### Manual Review Required: Y
(Issues with confidence >= 70 that could not be auto-fixed: requires architectural change, ambiguous fix, or in test file)

1. **file:line** - Description (confidence: XX)
   - Reason: Why it needs manual review
   - Suggested approach: Description

### Files Modified: Z
- path/to/file1.ts
- path/to/file2.ts

### Recommendations
- Tests to run
- Areas needing human judgment
```

**Audit 模式：**

```
## Code Audit Report

### Summary
- Files Audited: X | Issues Found: Y | Fixed: Z | Manual Review: W

### Critical Issues (Fixed)
1. **file:line** - Description
   - Category: security/bug/performance
   - Fix applied: What was changed

### Critical Issues (Manual Review Required)
(Confidence >= 80 but requires architectural change, ambiguous fix, or in test file)
1. **file:line** - Description
   - Reason: Why it was not auto-fixed
   - Recommended: Suggested approach

### Important Issues (Confidence 50-79)
1. **file:line** - Description (confidence: XX)

### Security Summary
- Hardcoded credentials: none found (or: X instances found)
- Injection risks: none found (or: X potential risks)
- XSS vulnerabilities: none found (or: X potential)
- Input validation: adequate (or: X gaps found)

### Files Modified
- path/to/file1.ts

### Recommendations
- Priority items for manual review
- Tests to add
```

### Step 8: 下一步

向用户提供：
- "Run tests to verify fixes?"
- "Review changes with `git diff`?"
- "Commit the fixes?"
- (仅 Audit 模式) "Audit additional directories?"

## 指南

**应该做：**
- 直接在代码中修复问题，而不仅仅是报告它们
- 保留代码意图并匹配现有模式
- 按文件分组相关修复以最小化编辑
- 保持保守：不确定时，报告而不是修复

**不应该做：**
- 用推测性修复破坏工作代码
- 在 diff 模式下修复预先存在的问题 — 只修复变更的内容
- 除非项目指南要求，否则只做风格变更
- 重构没有实际 bug 的工作代码
- 审计 node_modules, vendor, dist, 或生成的代码

## 需要避免的误报

- 当前变更未引入的预先存在的问题 (diff 模式)
- linter 或 type checker 会捕获的问题 (假设 CI 处理这些)
- 高级工程师在真实 review 中不会标记的挑剔
- 看起来不寻常但是正确的故意模式
- 当前变更范围之外的代码 (diff 模式)
- 未基于项目指南的一般质量意见
