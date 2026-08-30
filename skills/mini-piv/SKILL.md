---
name: mini-piv
description: 轻量级 PIV 工作流 - 以发现为驱动的功能构建器。无需 PRD。快速提问，生成 PRP，通过验证循环执行。适用于中小型功能，跳过 PRD 仪式。
user-invocable: True
disable-model-invocation: True
metadata:
  openclaw:
    emoji: zap
    homepage: https://github.com/SmokeAlot420/ftw
    requires:
      bins:
      - git
    os:
    - darwin
    - linux
tags:
- 图像生成
- 开发
---

# Mini PIV Ralph - 轻量级功能构建器

## 参数：$ARGUMENTS

解析参数：
```
FEATURE_NAME = $ARGUMENTS[0] or null（将在发现阶段询问用户）
PROJECT_PATH = $ARGUMENTS[1] or 当前工作目录
```

---

## 理念：快速且高质量

> "当你只是想构建点什么，而无需先写 PRD。"

同样高质量的流水线（执行 → 验证 → 调试），但始于快速对话而非 PRD。

**你是编排者** —— 保持精简，为繁重任务生成全新的子 agent。

**子 agent 生成：** 使用 `sessions_spawn` 工具创建全新的子 agent 会话。每次生成都是非阻塞的 —— 你将通过 announce 步骤接收结果。在继续下一步之前，等待每个 agent 的结果。

---

## 按角色必读

| 角色 | 说明 |
|------|-------------|
| Orchestrator | 仅本文件 |
| Research Agent | {baseDir}/references/codebase-analysis.md + {baseDir}/references/generate-prp.md |
| Executor | {baseDir}/references/piv-executor.md + {baseDir}/references/execute-prp.md |
| Validator | {baseDir}/references/piv-validator.md |
| Debugger | {baseDir}/references/piv-debugger.md |

---

## 可视化工作流

```
┌──────────────────────────────────────────────────────────┐
│ 1. 发现 → 提出 3-5 个问题                                 │
│ 2. 研究与 PRP → 代码库分析 + PRP 生成                     │
│ 3. 执行 → 实现 PRP                                        │
│ 4. 验证 → 通过 / 发现差距 / 需要人工介入                   │
│ 5. 调试循环 → 修复差距（最多 3 次）                        │
│ 6. 提交 → feat(mini): {description}                       │
└──────────────────────────────────────────────────────────┘
```

---

## 步骤 1：发现阶段

### 1a. 确定功能名称

如果未提供：询问用户或从上下文推断。规范化为 kebab-case。

### 1b. 检查现有 PRP

```bash
ls -la PROJECT_PATH/PRPs/ 2>/dev/null | grep -i "mini-{FEATURE_NAME}"
```

如果存在，询问：“覆盖、重命名，还是跳过至执行？”

### 1c. 提出发现性问题

在一条对话消息中呈现：

```
我有几个快速问题，以便我能正确构建：

1. **这个功能是做什么的？** 快速概述。
2. **它在代码库的哪个位置？** 文件、文件夹、组件？
3. **是否有特定的库、模式或现有代码需要遵循？**
4. **“完成”是什么样的？** 1-3 个具体的成功标准。
5. **有什么明确不在范围内的？**
```

根据功能类型（UI、API、合约、集成）进行调整。

### 1d. 结构化发现答案

```yaml
feature:
  name: {FEATURE_NAME}
  scope: {Q1}
  touchpoints: {Q2}
  dependencies: {Q3}
  success_criteria: {Q4}
  out_of_scope: {Q5}
```

---

## 步骤 2：研究与 PRP 生成

使用 `sessions_spawn` 生成一个**全新的子 agent**：

```
MINI PIV: 研究与 PRP 生成
====================================

项目根目录：{PROJECT_PATH}
功能名称：{FEATURE_NAME}

## 发现输入
{粘贴结构化 YAML}

## 步骤 1：代码库分析
阅读 {baseDir}/references/codebase-analysis.md 了解流程。
保存至：{PROJECT_PATH}/PRPs/planning/mini-{FEATURE_NAME}-analysis.md

## 步骤 2：生成 PRP（分析上下文仍加载）
阅读 {baseDir}/references/generate-prp.md 了解流程。

### 发现 → PRP 转换
| 发现 | PRP 章节 |
|-----------|-------------|
| 范围 (Q1) | 目标 + 内容 |
| 接触点 (Q2) | 实现任务位置 |
| 依赖 (Q3) | 上下文 YAML、已知陷阱 |
| 成功标准 (Q4) | 成功标准 + 验证 |
| 范围外 (Q5) | 内容章节中的排除项 |

使用模板：PRPs/templates/prp_base.md
输出至：{PROJECT_PATH}/PRPs/mini-{FEATURE_NAME}.md

自行完成两个步骤。不要生成子 agent。
```

**等待完成。**

---

## 步骤 3：生成执行器

使用 `sessions_spawn` 生成一个全新的子 agent：

```
执行器任务 - Mini PIV
============================

阅读 {baseDir}/references/piv-executor.md 了解你的角色。
阅读 {baseDir}/references/execute-prp.md 了解执行流程。

PRP: {PROJECT_PATH}/PRPs/mini-{FEATURE_NAME}.md
项目：{PROJECT_PATH}

遵循：加载 PRP → 彻底规划 → 执行 → 验证 → 核查
输出执行摘要。
```

---

## 验证规模决策

在生成完整验证器之前，评估：
- **修改文件 < 5 个，代码行 < 100，无外部 API** → 快速验证（作为编排者自行审查更改）
- **否则** → 生成完整验证器子 agent（步骤 4）

## 步骤 4：生成验证器

使用 `sessions_spawn` 生成一个全新的子 agent：

```
验证器任务 - Mini PIV
=============================

阅读 {baseDir}/references/piv-validator.md 了解你的流程。

PRP: {PROJECT_PATH}/PRPs/mini-{FEATURE_NAME}.md
项目：{PROJECT_PATH}
执行器摘要：{SUMMARY}

独立验证所有需求。
输出验证报告，包含评分、检查项、差距。
```

**处理结果：** 通过 → 提交 | 发现差距 → 调试器 | 需要人工介入 → 询问用户

---

## 步骤 5：调试循环（最多 3 次迭代）

使用 `sessions_spawn` 生成一个全新的子 agent：

```
调试器任务 - Mini PIV - 迭代 {I}
============================================

阅读 {baseDir}/references/piv-debugger.md 了解你的方法论。

项目：{PROJECT_PATH}
PRP: {PROJECT_PATH}/PRPs/mini-{FEATURE_NAME}.md
差距：{GAPS}
错误：{ERRORS}

修复根本原因。每次修复后运行测试。
输出修复报告。
```

调试器之后：重新验证 → 通过（提交）或循环（最多 3 次）或上报。

---

## 步骤 6：智能提交

```bash
cd PROJECT_PATH && git status && git diff --stat
git add -A
git commit -m "feat(mini): implement {FEATURE_NAME}

- {bullet 1}
- {bullet 2}

Built via Mini PIV Ralph

Built with FTW (First Try Works) - https://github.com/SmokeAlot420/ftw"
```

---

## 完成

```
## MINI PIV RALPH 完成

功能：{FEATURE_NAME}
项目：{PROJECT_PATH}

### 产物
- PRP: PRPs/mini-{FEATURE_NAME}.md
- 分析：PRPs/planning/mini-{FEATURE_NAME}-analysis.md

### 实现
- 验证周期：{N}
- 调试迭代：{M}

### 更改的文件
{list}

所有需求已验证并通过。
```

---

## 错误处理

- **执行器阻塞**：询问用户以获取指导
- **验证器需要人工介入**：询问用户以获取指导
- **3 次调试循环已耗尽**：上报并列出持续存在的问题

### 子 Agent 超时/失败
当子 agent 超时或失败时：
1. 检查部分工作（已创建的文件、已编写的测试）
2. 使用简化、更短的提示重试一次
3. 如果重试失败，向用户上报已完成的内容

---

## 快速参考

| 场景 | 使用此方案 |
|----------|----------|
| 小型/中型功能，无 PRD | **Mini PIV** |
| 大型功能，分阶段 | 完整 PIV (/piv) |

### 文件命名
```
PRPs/mini-{feature-name}.md                  # PRP
PRPs/planning/mini-{feature-name}-analysis.md # 分析
```
