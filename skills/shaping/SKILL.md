---
name: shaping
description: 产品和功能开发的 Shape Up 方法论。当协作塑造解决方案时使用 — 迭代问题定义（需求）和解决方案选项（shapes），将系统 breadboarding 为 affordances 和 wiring，并 slicing 为垂直的实现增量。触发词包括 \"shape this feature\"、\"breadboard the system\"、\"let's shape\"、\"slice this into increments\"、\"fit check\"、\"define requirements\"，或任何使用 Shape Up 方法论的产品/功能范围讨论。
tags:
- 管理
---

# 塑造与面包板（Shaping & Breadboarding）

定义问题、探索解决方案和规划实施的结构化方法论。基于 [Shape Up](https://basecamp.com/shapeup) 改编，适用于与 LLM 协作。

来源：[rjs/shaping-skills](https://github.com/rjs/shaping-skills) by [@rjs](https://github.com/rjs) (Ryan Singer, [Shape Up](https://basecamp.com/shapeup) 的作者)

## 合二为一的技能

**Shaping** — 在承诺实施之前迭代问题（需求）和解决方案（shapes）。将你需要什么与你可能如何构建它分开，通过 fit checks 来看什么已解决、什么未解决。

**Breadboarding** — 将系统映射为 UI affordances、code affordances 和 wiring。在一个视图中展示用户可以做什么以及底层如何工作。适用于 slicing 为垂直范围。

## 何时使用

- 探索新功能或产品方向
- 在构建之前比较解决方案方法
- 映射现有系统以理解变更落在何处
- 将选定的解决方案分解为垂直实现 slices
- 任何 "我们应该构建 X 还是 Y?" 的讨论

## 入口点

- **从 R（需求）开始** — 描述问题、痛点、约束。构建需求并让 shapes 浮现。
- **从 S（方案）开始** — 勾勒一个已有的解决方案。将其捕获为 shape 并边做边提取需求。

没有必需的顺序。R 和 S 在整个过程中相互告知。

## 核心符号

| 层级 | 符号 | 含义 | 关系 |
|-------|----------|---------|--------------|
| 需求 | R0, R1, R2... | 问题约束 | 集合 R 的成员 |
| 方案 | A, B, C... | 解决方案选项 | 从 S 中选择一个 |
| 组件 | C1, C2, C3... | shape 的组成部分 | 在 shape 内组合 |
| 替代方案 | C3-A, C3-B... | 某个组件的实现方式 | 每个组件选一个 |

## 阶段

```
Shaping → Slicing
```

- **Shaping**: 探索问题/解决方案空间，选择并细化一个 shape
- **Slicing**: 分解为可演示 UI 的垂直 slices

## 关键动作

- **Populate R** — 收集浮现的需求
- **Sketch a shape** — 提出高层次方法
- **Detail** — 将 shape 分解为 components 或具体的 affordances
- **Check fit** — 构建决策矩阵 (R × S)，仅二进制 ✅/❌
- **Breadboard** — 映射到带有 wiring 的 UI/Code affordances
- **Spike** — 调查未知项
- **Slice** — 将 breadboarded shape 分解为垂直增量

## 详细参考

如需完整的方法论、符号规则、示例和流程：

- **Shaping 参考**：参见 [references/shaping.md](references/shaping.md) —— 完整的 shaping 方法论，包括 fit checks、parts、spikes、documents、multi-level consistency
- **Breadboarding 参考**：参见 [references/breadboarding.md](references/breadboarding.md) —— 完整的 breadboarding 流程、affordance tables、places、wiring、Mermaid 约定、chunking、slicing

在进入工作对应阶段时加载相关参考。

## 快速参考：Fit Check 格式

```markdown
| Req | Requirement | Status | A | B | C |
|-----|-------------|--------|---|---|---|
| R0 | 完整需求文本 | 核心目标 | ✅ | ✅ | ✅ |
| R1 | 完整需求文本 | 必须有 | ✅ | ❌ | ✅ |
```

- 始终显示完整需求文本，不要缩写
- 仅二进制：✅ 或 ❌。fit checks 中不使用 ⚠️
- 解释放在表格下方的 Notes 区域

## 快速参考：Affordance 表格

**UI Affordances**: `# | Place | Component | Affordance | Control | Wires Out | Returns To`
**Code Affordances**: 相同列
**Controls**: click, type, call, observe, write, render
**Wires Out** (solid →): 控制流 —— 调用、触发、写入
**Returns To** (dashed -.->): 数据流 —— 返回值、读取

## 快速参考：Slicing

- 每个 slice 必须以 **可演示的 UI** 结束
- 最多 9 个 slices
- 每个 slice 展示一个正在工作的 mechanism
- 格式：`V1: Name` —— affordances，demo statement
