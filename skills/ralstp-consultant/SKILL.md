---
name: ralstp-consultant
description: 基于 RALSTP (Recursive Agents and Landmarks Strategic-Tactical Planning) 的复杂问题分析框架。识别智能体(agents)、评估任务难度(entanglement/complexity)并建议分解策略。适用于多步骤工作流、依赖迁移与资源争用问题。触发词：RALSTP、战略战术规划(strategic-tactical planning)、任务分解(task decomposition)、agent 分析、复杂度评估(complexity assessment)
tags:
- AI
- Consul
---

# RALSTP Consultant

基于 **"Recursive Agents and Landmarks Strategic-Tactical Planning (RALSTP)"** 作者 Dorian Buksz, King's College London, 2024。

## 核心概念 (来自论文)

### 1. Agents Identification

**定义:** Agents 是具有 **dynamic types** 的对象，在目标状态搜索期间是活跃的。

**如何识别:**
- Dynamic type = 出现在任何 action 的 **effects** 中作为 predicate 的第一个参数
- Static type = 从不出现在 action effects 中
- 示例: 在 Driverlog 中, `truck` 和 `driver` 是 dynamic (它们在 `drive` action effects 中), 但 `location` 是 static

### 2. Passive Objects

不是 agents 的对象 — 被作用的事物但自身不作用。
- Packages, cargo, data, files, victims in RTAM

### 3. Agent Dependencies

**定义:** 基于 agents 为其他 agents 满足的 preconditions 之间的关系。

**类型:**
- **Independent** — 不相互依赖的 agents
- **Dependent** — 需要其他 agents 的 preconditions 被满足的 agents
- **Conflicting** — 相互干扰的 agents

### 4. Entanglement

**定义:** 当 agents 争夺共享资源时 (时间, 空间, 位置等)

**测量:**
- 共享 predicates 的数量
- 目标状态中的冲突频率

### 5. Landmarks

**定义:** 在任何有效计划中 **必须为真** 的事实 (从目标回溯到初始状态)。

**类型:**
- **Fact landmarks** — 必须成立的命题
- **Action landmarks** — 必须执行的动作
- **Relaxed landmarks** — 仅考虑 positive effects 的 landmarks (忽略 deletes)

### 6. Strategic vs Tactical

- **Strategic:** 抽象规划层级。解决 "什么需要先发生" 忽略细节。
- **Tactical:** 详细执行层级。解决 "具体如何做"。

### 7. Difficulty Metrics

来自论文，难度随以下因素增加:
- 目标状态中更多的 agents
- 更多纠缠的 agents (冲突依赖)
- 更多不在目标中的 inactive dynamic objects

**Buksz Complexity Score ≈ Agent Count × Entanglement Factor**

## Usage

对于任何复杂问题，只需描述它，我将应用 RALSTP:

```
RALSTP analyze: I need to migrate 1000 VMs from datacentre A to B with minimal downtime
```

## Output Format

```
## RALSTP Analysis

### Agents Identified
- [list agents and their types]

### Passive Objects  
- [list objects being acted upon]

### Dependency Graph
- [which agents depend on which]

### Difficulty Assessment
- Agent Count: X
- Entanglement: Low/Medium/High
- Estimated Complexity: [score]

### Strategic Phase
- [high-level plan ignoring details]

### Tactical Phase
- [detailed execution]

### Decomposition Suggestion
- Split by: [agent type / landmark / location]
- Parallelize: [what can run concurrently]
- Risks: [potential conflicts/entanglements]
```

## When to Use

**USE for:**
- 多步骤工作流与多个 actors
- 有依赖的迁移/任务
- 资源争用问题
- 复杂编排

**SKIP for:**
- 简单问答
- 单任务问题

## Reference

PhD Thesis: "Recursive Agents and Landmarks Strategic-Tactical Planning (RALSTP)" — Dorian Buksz, King's College London, 2024.
