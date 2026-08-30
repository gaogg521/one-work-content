---
name: sw-tdd-workflow
description: 检测 TDD 意图并引导 red-green-refactor 循环的 TDD 发现中心。适用于先写测试、实施 TDD 工作流或学习 test-first 开发实践的场景，分发至对应 TDD 命令。
---

# TDD 工作流 - 发现与协调技能

## 目的

此技能在 SpecWeave 中作为 Test-Driven Development (TDD) 的**发现中心**。它：
- ✅ 检测用户何时想用 TDD 实现功能
- ✅ 询问用户对 TDD 工作流执行的偏好
- ✅ 路由到适当的 TDD 工具 (命令 vs 代理)
- ✅ 提供 TDD 教育和最佳实践

**不是完整的 TDD 实现** - 委托给：
- `tdd-orchestrator` 代理 (深度 TDD 专业知识)
- `/sw:tdd:cycle` 命令 (强制的 red-green-refactor)
- 独立阶段命令 (`/sw:tdd:red`, `/sw:tdd:green`, `/sw:tdd:refactor`)

---

## 何时激活

**当用户提到以下时自动激活**：
- "implement with TDD"
- "use test-driven development"
- "red-green-refactor"
- "write tests first"
- "test-first approach"
- "Kent Beck style"
- "TDD discipline"

**示例触发器**：
```
User: "Implement authentication with TDD"
User: "Use test-driven development for this feature"
User: "Let's do red-green-refactor for the payment module"
```

---

## 工作流

### 步骤 1：检测 TDD 意图

激活时，确认用户的 TDD 意图：

```
"I detected you want to use Test-Driven Development (TDD).

TDD follows the red-green-refactor cycle:
🔴 RED: Write a failing test first
🟢 GREEN: Write minimal code to make it pass
🔵 REFACTOR: Improve code while keeping tests green

Would you like to:"
```

### 步骤 2：提供 TDD 选项

**使用 AskUserQuestion 工具** 呈现选择：

```typescript
Question: "How would you like to implement TDD for this feature?"
Options:
  1. "Guided TDD Workflow (/sw:tdd:cycle)"
     Description: "Full red-green-refactor cycle with gates between phases.
                   Can't proceed to GREEN without RED test. Most rigorous."

  2. "Expert TDD Agent (tdd-orchestrator)"
     Description: "Deep TDD expertise with flexible workflow.
                   Best for complex scenarios, property-based testing, legacy code."

  3. "Manual TDD (I'll guide myself)"
     Description: "I'll implement TDD discipline myself.
                   You provide TDD advice when needed."
```

### 步骤 3：基于选择路由

**选项 1：引导式 TDD 工作流**
```bash
Invoke: /sw:tdd:cycle

此命令编排：
1. /sw:tdd:red    - 编写失败测试 (在 red 前阻塞)
2. /sw:tdd:green  - 实现最小代码 (在 green 前阻塞)
3. /sw:tdd:refactor - 安全重构 (测试必须保持 green)

好处：
- 强制执行纪律 (门防止跳过阶段)
- 非常适合初学者或学习 TDD 的团队
- 与 SpecWeave increment 工作流集成
```

**选项 2：专家 TDD 代理**
```bash
Invoke: tdd-orchestrator agent (via Task tool)

此代理提供：
- 多代理 TDD 工作流协调
- 基于属性的测试 (QuickCheck, Hypothesis)
- 用于测试质量的变异测试
- 带安全网的遗留代码重构
- BDD/ATDD 集成
- AI 辅助测试生成

好处：
- 灵活的工作流 (非刚性门)
- 高级技术 (基于属性、变异)
- 最适合有经验的 TDD 实践者
- 处理复杂场景
```

**选项 3：手动 TDD**
```bash
提供 TDD 最佳实践：

"I'll implement your feature while following TDD principles.
I'll ensure:
- Tests written before implementation
- Minimal code to pass tests
- Refactoring with test coverage
- Clear red→green→refactor progression

I'll notify you at each phase transition."
```

---

## TDD 最佳实践 (参考)

### Red Phase 🔴
- 编写最简单的失败测试
- 测试应编译但断言失败
- 关注 WHAT，不是 HOW
- 一次一个测试

### Green Phase 🟢
- 编写最小代码让测试通过
- 接受 "fake it till you make it"
- 最初硬编码值可接受
- 快速到达 green

### Refactor Phase 🔵
- 改进代码结构
- 提取方法、移除重复
- 测试必须保持 green
- 每次重构后提交

### 应避免的 TDD 反模式
- ❌ 在测试前写实现
- ❌ 在实现前写多个测试
- ❌ 在 GREEN 阶段过度工程
- ❌ 在没有通过测试的情况下重构
- ❌ 跳过重构阶段

---

## 与 SpecWeave 集成

**在 Increment 工作流中**：
```
/sw:inc "Authentication feature" → spec.md created
↓
User: "Implement with TDD"
↓
tdd-workflow skill activates → offers options
↓
User chooses: Guided TDD Workflow
↓
/sw:tdd:cycle invoked
  ↓
  Phase 1: RED   - tests.md 更新为失败测试
  Phase 2: GREEN - tasks.md 实现
  Phase 3: REFACTOR - 代码改进
↓
Increment tasks 以 TDD 纪律完成
```

**与 spec-driven-debugging 配合**：
```
发现 Bug → spec-driven-debugging 激活
↓
修复提案包含："Add regression test first (TDD RED phase)"
↓
tdd-workflow 可以协调 test-first bug 修复
```

---

## 命令参考

### 完整循环
- `/sw:tdd:cycle` - 完整的 red-green-refactor 编排

### 独立阶段
- `/sw:tdd:red` - 仅 RED 阶段 (编写失败测试)
- `/sw:tdd:green` - 仅 GREEN 阶段 (让测试通过)
- `/sw:tdd:refactor` - 仅 REFACTOR 阶段 (改进代码)

### 何时使用每个

**使用 /sw:tdd:cycle 当**：
- ✅ 从零开始新功能
- ✅ 学习 TDD 或教授团队
- ✅ 想要强制纪律 (门)
- ✅ 在基于 increment 的工作流中工作

**使用独立命令当**：
- ✅ 已经在 TDD 循环中间
- ✅ 需要重复一个阶段 (例如，多次重构)
- ✅ 想要对循环更精细的控制
- ✅ 与其他工作流集成

**使用 tdd-orchestrator 代理当**：
- ✅ 复杂场景 (基于属性的测试)
- ✅ 遗留代码重构
- ✅ 需要多代理协调
- ✅ 需要高级 TDD 技术

---

## 示例交互

### 示例 1：TDD 初学者

**User**: "I want to implement user registration with TDD"

**tdd-workflow**:
```
Great! TDD (Test-Driven Development) follows red-green-refactor:
🔴 RED: Write failing test
🟢 GREEN: Make it pass
🔵 REFACTOR: Improve code

[Presents AskUserQuestion with 3 options]
```

**User selects**: "Guided TDD Workflow"

**tdd-workflow**:
```
Perfect! I'm invoking /sw:tdd:cycle to orchestrate your TDD workflow.

🔴 Starting RED phase...
Write a failing test for user registration. Let me create the test skeleton:

[Creates test file with failing test]

✅ Test written and failing. Run: npm test
❌ Expected failure confirmed.

Ready to proceed to GREEN phase? (type 'yes' or run /sw:tdd:green)
```

---

### 示例 2：有经验的 TDD 用户

**User**: "Refactor auth module with TDD safety nets"

**tdd-workflow**:
```
I see you want to refactor with TDD discipline.

[Presents AskUserQuestion]
```

**User selects**: "Expert TDD Agent"

**tdd-workflow**:
```
Invoking tdd-orchestrator agent for advanced refactoring support...

[tdd-orchestrator agent takes over with:]
- Characterization tests for legacy code
- Golden master testing
- Seam identification
- Incremental refactoring with safety nets
```

---

### 示例 3：TDD 循环中途

**User**: "I'm in GREEN phase, tests passing, ready to refactor"

**tdd-workflow**:
```
Great! Tests are passing. Time for REFACTOR phase.

You can:
1. Run /sw:tdd:refactor (guided refactoring)
2. Refactor manually (I'll verify tests stay green)

Which would you prefer?
```

**User**: "I'll refactor manually"

**tdd-workflow**:
```
Perfect! I'll monitor your refactoring:

✅ Tests passing before refactor
[User refactors code]
✅ Tests still passing after refactor
✅ TDD cycle maintained

Ready for next feature? Run /sw:tdd:cycle or continue manually.
```

---

## 配置

**可选**：在 `.specweave/config.yaml` 中自定义 TDD 偏好：

```yaml
tdd:
  default_workflow: "cycle"  # 选项: "cycle", "agent", "manual"
  auto_activate: true         # 在新功能上自动提供 TDD
  gates_enabled: true         # 在 cycle 模式中强制执行阶段门
  mutation_testing: false     # 启用变异测试 (需要设置)
```

---

## 成功标准

**此技能成功当**：
- ✅ 在适当时向用户提供 TDD 工作流
- ✅ TDD 选择是明确的 (不是假设的)
- ✅ 清晰路由到适当工具 (命令 vs 代理)
- ✅ TDD 教育嵌入工作流
- ✅ 对初学者和专家足够灵活
- ✅ 与 SpecWeave increments 无缝集成

---

## 相关技能与代理

**技能**：
- `spec-driven-debugging` - Bug 修复可以使用 TDD 方法
- `increment-planner` - Increments 可以将 TDD 指定为方法论
- `e2e-playwright` - E2E 测试可以为验收测试遵循 TDD

**代理**：
- `tdd-orchestrator` - 深度 TDD 专业知识 (由此技能调用)
- `qa-lead` - 测试策略与 TDD 原则重叠

**命令**：
- `/sw:tdd:cycle` - 完整的 red-green-refactor 编排
- `/sw:tdd:red`, `/sw:tdd:green`, `/sw:tdd:refactor` - 独立阶段

---

## 总结

**tdd-workflow** 是一个轻量级发现技能，它：

1. ✅ **检测** 用户消息中的 TDD 意图
2. ✅ **询问** 用户对 TDD 执行级别的偏好
3. ✅ **路由** 到适当工具 (引导命令 vs 专家代理)
4. ✅ **教育** TDD 原则和最佳实践
5. ✅ **集成** 与 SpecWeave increment 工作流

**不能替代**：
- `tdd-orchestrator` 代理 (深度专业知识)
- `/sw:tdd-*` 命令 (工作流执行)

**相反，它是入口点**，帮助用户为其上下文选择正确的 TDD 工具。

---

**Keywords**: TDD, test-driven development, red-green-refactor, test-first, Kent Beck, TDD cycle, property-based testing, mutation testing, refactoring, test discipline
