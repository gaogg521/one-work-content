---
name: first-principles-decomposer
description: 将任何问题分解到基本真理，然后从原子层面重建解决方案。当用户说 \"firstp\"、\"first principles\"、\"from scratch\"、\"what are we assuming\"、\"break this down\"、\"atomic\"、\"fundamental truth\"、\"physics thinking\"、\"Elon method\"、\"bedrock\"、\"ground up\"、\"core problem\"、\"strip away\"，或挑战关于做事方式的假设时使用。
tags:
- 核心
---

# First Principles Decomposer

## 何时使用
- 设计新产品或功能
- 在复杂问题上感到卡住
- 现有解决方案似乎过于复杂
- 需要挑战假设
- 启动任何新项目或 initiative

## 流程

### 阶段 1：识别假设
问："我假设了哪些可能并不成立的事情？"
列出当前方法中嵌入的每一个 assumption。

### 阶段 2：分解到原子
对每一个 assumption，问："这里最基本的真理是什么？"
不断问 "why?" 直到触及 bedrock facts。

### 阶段 3：从真理重建
仅从已验证的 fundamentals 出发，问：
"解决核心需求的最简单方案是什么？"

## 交互流程

当用户调用本 skill 时：

1. **澄清问题**（最多 1-2 个问题）
2. **浮现假设** - 列出被认为理所当然的内容
3. **分解到 fundamentals** - 展示 atomic truths
4. **重建解决方案** - 从 ground up 构建
5. **对比** - 展示这与 conventional approach 有何不同

## 输出格式

```
PROBLEM: [陈述的问题]

识别的假设：
1. [assumption] → 挑战：[为什么这可能是错的]
2. [assumption] → 挑战：[为什么这可能是错的]

基本真理：
• [bedrock fact 1]
• [bedrock fact 2]
• [bedrock fact 3]

重建的解决方案：
[仅基于 fundamentals 的新方法]

与传统方法对比：
[这与显而易见的做法有何不同]
```

## 示例触发词
- "Break down our parent communication problem from first principles"
- "I want to rethink how we do [X] from the ground up"
- "What are we assuming about [problem] that might be wrong?"

## 集成

本 skill 可与以下技能复合：
- **inversion-strategist** - 从 fundamentals 重建后，invert 以找出什么会保证新方法失败
- **second-order-consequences** - 预测实施重建解决方案后的 downstream effects
- **pre-mortem-analyst** - 通过想象其失败来 stress-test 重建的解决方案
- **six-thinking-hats** - 将六种视角应用于验证每一个 identified fundamental truth

## Skill Metadata
**Created**: 2026-01-06
**Last Updated**: 2026-01-06
**Author**: Artem
**Version**: 1.0

---
详见 references/framework.md 了解详细方法论
详见 references/examples.md 了解 Artem 专属示例
详见 references/integrated-frameworks.md 了解 Stanford Design Thinking + MIT Systems Engineering 组合
