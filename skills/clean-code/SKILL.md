---
name: clean-code
description: 务实的编码标准，强调简洁、直接、不过度工程化，不写不必要的注释。触发词：整洁代码(clean code)、编码标准(coding standards)、重构(refactoring)、代码质量(code quality)。
allowed-tools: Read, Write, Edit
version: 2.0
priority: CRITICAL
tags:
- 代码审查
---

# Clean Code - 务实的 AI 编码标准

> **关键技能** - 保持**简洁、直接、以解决方案为导向**。

---

## 核心原则

| 原则 | 规则 |
|-----------|------|
| **SRP** | 单一职责 - 每个函数/类只做一件事 |
| **DRY** | 不要重复自己 - 提取重复代码，复用 |
| **KISS** | 保持简单 - 使用最简单的可行方案 |
| **YAGNI** | 你不会需要它 - 不要构建未使用的功能 |
| **童子军** | 让代码比你发现时更干净 |

---

## 命名规则

| 元素 | 规范 |
|---------|------------|
| **变量** | 揭示意图：`userCount` 而不是 `n` |
| **函数** | 动词 + 名词：`getUserById()` 而不是 `user()` |
| **布尔值** | 疑问形式：`isActive`、`hasPermission`、`canEdit` |
| **常量** | SCREAMING_SNAKE：`MAX_RETRY_COUNT` |

> **规则：** 如果你需要注释来解释一个名字，那就重命名它。

---

## 函数规则

| 规则 | 描述 |
|------|-------------|
| **小** | 最多 20 行，理想情况下 5-10 行 |
| **单一职责** | 只做一件事，把它做好 |
| **单一层次** | 每个函数只包含一个抽象层次 |
| **参数少** | 最多 3 个参数，优先 0-2 个 |
| **无副作用** | 不要意外修改输入 |

---

## 代码结构

| 模式 | 应用 |
|---------|-------|
| **卫语句** | 对边界情况提前返回 |
| **扁平 > 嵌套** | 避免深层嵌套（最多 2 层） |
| **组合** | 将小的函数组合在一起 |
| **就近原则** | 将相关代码放在一起 |

---

## AI 编码风格

| 情况 | 行动 |
|-----------|--------|
| 用户请求功能 | 直接编写 |
| 用户报告 bug | 修复它，不要解释 |
| 需求不明确 | 询问，不要假设 |

---

## 反模式（不要）

| ❌ 模式 | ✅ 修复 |
|-----------|-------|
| 每行都写注释 | 删除显而易见的注释 |
| 为一行代码创建 helper | 内联代码 |
| 为 2 个对象创建工厂 | 直接实例化 |
| utils.ts 里只有 1 个函数 | 把代码放在使用它的地方 |
| "首先导入..." | 直接写代码 |
| 深层嵌套 | 卫语句 |
| 魔法数字 | 命名常量 |
| 上帝函数 | 按职责拆分 |

---

## 🔴 编辑任何文件之前（先思考！）

**在修改文件之前，问自己：**

| 问题 | 原因 |
|----------|-----|
| **什么导入了这个文件？** | 它们可能会中断 |
| **这个文件导入了什么？** | 接口变更 |
| **什么测试覆盖了它？** | 测试可能会失败 |
| **这是一个共享组件吗？** | 多个地方会受影响 |

**快速检查：**
```
要编辑的文件：UserService.ts
└── 谁导入了它？→ UserController.ts、AuthController.ts
└── 它们也需要修改吗？→ 检查函数签名
```

> 🔴 **规则：** 在同一个任务中编辑文件 + 所有依赖文件。
> 🔴 **不要留下中断的导入或缺失的更新。**

---

## 总结

| 应该 | 不应该 |
|----|-------|
| 直接写代码 | 写教程 |
| 让代码自解释 | 添加显而易见的注释 |
| 立即修复 bug | 先解释修复 |
| 内联小东西 | 创建不必要的文件 |
| 命名清晰 | 使用缩写 |
| 保持函数短小 | 写 100+ 行的函数 |

> **记住：用户想要能工作的代码，而不是编程课程。**

---

## 🔴 完成前自检（强制）

**在说"任务完成"之前，验证：**

| 检查 | 问题 |
|-------|----------|
| ✅ **目标达成？** | 我是否准确完成了用户要求？ |
| ✅ **文件已编辑？** | 我是否修改了所有必要的文件？ |
| ✅ **代码可用？** | 我是否测试/验证了变更？ |
| ✅ **无错误？** | Lint 和 TypeScript 通过？ |
| ✅ **没有遗漏？** | 有没有遗漏边界情况？ |

> 🔴 **规则：** 如果有任何检查失败，在完成前修复它。

---

## 验证脚本（强制）

> 🔴 **关键：** 每个 agent 只在自己的工作完成后运行自己技能的脚本。

### Agent → 脚本映射

| Agent | 脚本 | 命令 |
|-------|--------|---------|
| **frontend-specialist** | UX 审计 | `python .agent/skills/frontend-design/scripts/ux_audit.py .` |
| **frontend-specialist** | A11y 检查 | `python .agent/skills/frontend-design/scripts/accessibility_checker.py .` |
| **backend-specialist** | API 验证器 | `python .agent/skills/api-patterns/scripts/api_validator.py .` |
| **mobile-developer** | 移动端审计 | `python .agent/skills/mobile-design/scripts/mobile_audit.py .` |
| **database-architect** | Schema 验证 | `python .agent/skills/database-design/scripts/schema_validator.py .` |
| **security-auditor** | 安全扫描 | `python .agent/skills/vulnerability-scanner/scripts/security_scan.py .` |
| **seo-specialist** | SEO 检查 | `python .agent/skills/seo-fundamentals/scripts/seo_checker.py .` |
| **seo-specialist** | GEO 检查 | `python .agent/skills/geo-fundamentals/scripts/geo_checker.py .` |
| **performance-optimizer** | Lighthouse | `python .agent/skills/performance-profiling/scripts/lighthouse_audit.py <url>` |
| **test-engineer** | 测试运行器 | `python .agent/skills/testing-patterns/scripts/test_runner.py .` |
| **test-engineer** | Playwright | `python .agent/skills/webapp-testing/scripts/playwright_runner.py <url>` |
| **任何 agent** | Lint 检查 | `python .agent/skills/lint-and-validate/scripts/lint_runner.py .` |
| **任何 agent** | 类型覆盖 | `python .agent/skills/lint-and-validate/scripts/type_coverage.py .` |
| **任何 agent** | i18n 检查 | `python .agent/skills/i18n-localization/scripts/i18n_checker.py .` |

> ❌ **错误：** `test-engineer` 运行 `ux_audit.py`
> ✅ **正确：** `frontend-specialist` 运行 `ux_audit.py`

---

### 🔴 脚本输出处理（读取 → 总结 → 询问）

**运行验证脚本时，你必须：**

1. **运行脚本** 并捕获所有输出
2. **解析输出** - 识别错误、警告和通过项
3. **向用户总结** 使用以下格式：

```markdown
## 脚本结果：[script_name.py]

### ❌ 发现错误 (X 项)
- [文件:行] 错误描述 1
- [文件:行] 错误描述 2

### ⚠️ 警告 (Y 项)
- [文件:行] 警告描述

### ✅ 通过 (Z 项)
- 检查 1 通过
- 检查 2 通过

**我应该修复这 X 个错误吗？**
```

4. **等待用户确认** 后再修复
5. **修复后** → 重新运行脚本确认

> 🔴 **违规：** 运行脚本并忽略输出 = 任务失败。
> 🔴 **违规：** 未经询问自动修复 = 不允许。
> 🔴 **规则：** 始终先读取输出 → 总结 → 询问 → 然后修复。
