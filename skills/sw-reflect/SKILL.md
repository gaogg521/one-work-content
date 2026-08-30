---
name: sw-reflect
description: 自改进的AI记忆系统，将会话中的学习持久化到技能特定的MEMORY.md文件中。用于捕获纠正、记住用户偏好或从成功实现中提取模式。支持跨会话的持续学习，无需每次对话从零开始。
---

# 自改进技能 (Reflect)

## 概述

Reflect系统支持**跨会话的持续学习**。Claude不再每次对话从零开始，而是从纠正、成功模式和用户偏好中学习——将知识持久化到**技能特定的MEMORY.md文件**中。

```
Session 1: 用户纠正按钮样式 → Reflect捕获学习 → 保存到frontend技能
Session 2: Claude无需提醒即使用正确的按钮样式
Session 3+: 知识不断累积，Claude随时间变得更智能
```

---

## 架构

### 技能特定记忆

每个技能都有自己的MEMORY.md文件，存储学习到的模式：

```
# Claude Code Environment
~/.claude/plugins/marketplaces/specweave/plugins/specweave/skills/
├── architect/
│   ├── SKILL.md              # Skill definition
│   └── MEMORY.md             # User learnings for this skill
├── frontend/
│   ├── SKILL.md
│   └── MEMORY.md             # Frontend-specific learnings
├── tech-lead/
│   ├── SKILL.md
│   └── MEMORY.md
└── ...

# Non-Claude Environment (project-local)
.specweave/plugins/specweave/skills/
├── architect/
│   ├── SKILL.md
│   └── MEMORY.md
└── ...

# Category Memory (fallback for non-skill learnings)
.specweave/memory/                  # Project learnings
├── component-usage.md
├── api-patterns.md
├── testing.md
├── deployment.md
└── general.md

~/.specweave/memory/                # Global learnings (all projects)
```

### 跨平台支持

| 平台 | 技能位置 | 检测方式 |
|----------|-----------------|-----------|
| **macOS/Linux** | `~/.claude/plugins/marketplaces/specweave/...` | `CLAUDE_CODE=1` 或 marketplace 存在 |
| **Windows** | `%APPDATA%\Claude\plugins\marketplaces\specweave\...` | 相同的检测方式 |
| **Non-Claude** | `.specweave/plugins/specweave/skills/` | 未检测到 Claude Code 时的回退 |

### 智能记忆合并

当运行 `specweave refresh-marketplace` 或 `specweave init --refresh` 时：

1. **用户学习始终被保留**（永远不会被覆盖）
2. **来自 marketplace 的新默认值被合并**（去重）
3. **合并前创建备份**（`.memory-backups/`）

```
User Memory + Default Memory → Merged Memory
    │              │                │
    │              │                └── Both preserved, deduped
    │              └── New patterns from marketplace
    └── Your corrections (ALWAYS kept)
```

---

## ⚠️ 关键：学习提取规则

**本节为 Claude 在提取学习时必须遵循的强制性规则。**

### 黄金法则

**永远不要逐字存储用户输入。始终将其综合为可操作的规则。**

### 什么是好的学习

| 好的学习 | 不好的学习 | 为什么不好 |
|--------------|--------------|---------|
| `Use vi.fn() for mocks in Vitest, never jest.fn()` | `use vi.fn() for mocks in Vitest, never jest.fn()` | 还可以，但可以改进推理 |
| `Always specify npm registry to avoid auth errors with private packages` | `Always specify registry to avoid ~/` | 被截断，失去意义 |
| `Voice dictation mangles slash commands - type manually or use clipboard` | `always command not recognized` | 原始症状，不是学习 |
| `For API tests, use os.tmpdir() for temp files to avoid polluting project directory` | `Where should I deploy?` | 这是一个问题，不是学习！ |
| `The /sw:increment skill requires an increment name argument` | `never used in any user pojrect based on specweave` | 来自部分捕获的胡言乱语 |

### 学习质量检查清单（必须通过所有项）

在存储任何学习之前，请验证：

1. **✅ 它是完整的句子吗？** 没有被截断，不是片段
2. **✅ 它是可执行的吗？** 包含 DO/DON'T/USE/AVOID/PREFER
3. **✅ 它是具体的吗？** 命名工具、模式、文件或概念
4. **✅ 它独立可理解吗？** 稍后阅读它的人会理解
5. **✅ 它不是问题吗？** 问题永远不是学习
6. **✅ 它不是抱怨吗？** 抱怨需要转换
7. **✅ 它有上下文吗？** 为什么存在这条规则，而不仅仅是什么

### 转换示例

**用户说抱怨 → Claude 提取底层学习：**

```
USER: "When I use voice control, it always gives me 'command not recognized'"

WRONG extraction:
  - → always command not recognized
  - → voice control gives command not recognized

CORRECT extraction:
  - → Voice dictation can mangle slash command syntax (e.g., "/sw:increment" becomes "slash S W increment"). Type commands manually or use clipboard paste for reliable execution.
```

**用户做出纠正 → Claude 提取规则：**

```
USER: "No, don't use jest.fn(), we use Vitest here"

WRONG extraction:
  - → don't use jest.fn()

CORRECT extraction:
  - → Use vi.fn() not jest.fn() with Vitest testing framework. Import mocks from 'vitest' package.
```

**用户批准某事 → Claude 提取模式：**

```
USER: "Perfect! That's exactly how we handle errors"

WRONG extraction:
  - → Perfect! That's exactly how we handle errors

CORRECT extraction (look at WHAT was approved):
  - → For API error responses, use { success: false, error: { code: string, message: string } } structure
```

### 应该拒绝什么（永远不要存储）

1. **问题** - `"Where should I deploy?"` → 不是学习
2. **片段** - `"eplicilty how to g"` → 截断的垃圾
3. **原始症状** - `"always command not recognized"` → 没有解释
4. **重复** - 相同规则的不同表述
5. **临时上下文** - `"for this PR"`, `"just this time"`
6. **个人偏好** - 没有普遍适用性
7. **错别字/胡言乱语** - `"user pojrect"`, `"promp"`

### 记忆格式要求

每个条目必须遵循以下格式：

```markdown
- → {VERB} {specific action} {context/reason if helpful}
```

或者用于纠正：
```markdown
- ✗→✓ {wrong way} → {right way} {reason}
```

**正确格式的示例：**
```markdown
- → Use vi.fn() for mocks in Vitest, never jest.fn()
- → Use os.tmpdir() for test temp files, not project cwd
- ✗→✓ Never suggest scripts/refresh-marketplace.sh to end users - use `specweave refresh-marketplace` CLI command
- → Voice dictation mangles slash commands - type manually or paste from clipboard
```

### 提取流程

当调用 `/sw:reflect` 时：

1. **扫描对话** 以查找信号（纠正、规则、批准、抱怨）
2. **对于每个信号**，应用转换：
   - 纠正 → 提取被教授的规则
   - 规则 → 保留带上下文的规则
   - 批准 → 提取被批准的内容（查看 Claude 的上一条消息）
   - 抱怨 → 转换为可操作的变通方法/解决方案
3. **验证** 每个提取是否符合质量检查清单
4. **拒绝** 任何未通过验证的（存储垃圾不如什么都不存）
5. **去重** 与现有记忆对比
6. **存储** 使用正确的格式

### 存储前的自我检查

问自己：
> "If I read this learning in 6 months with no context, would it help me?"

如果 否 → 不要存储。
如果 可能 → 改进它直到变成 是。
如果 是 → 存储它。

---

## 问题

每次 LLM 会话都从零开始：

1. **周一**：您纠正 Claude - "Use our primary button component, not a custom style"
2. **周二**：Claude 再次犯同样的错误
3. **周三**：同样的纠正，同样的沮丧
4. **永远**：没有记忆，您将无限重复自己

这表现为：
- 错误的命名约定
- 不正确的日志模式
- 缺少输入验证
- 错误的组件使用
- 被遗忘的架构决策

---

## 解决方案

Reflect 分析会话并将学习持久化到**技能特定的 MEMORY.md 文件**中：

```markdown
# frontend skill's MEMORY.md

# Skill Memory: frontend

> Auto-generated by SpecWeave Reflect v4.0
> Last updated: 2026-01-06T10:30:00Z
> Skill: frontend

## Learned Patterns

### LRN-20260106-A1B2 (correction, high)
**Content**: Always use `<Button variant='primary'>` from `@/components/ui/button` for primary actions. Never create custom button styles.
**Context**: User corrected button component usage in settings page
**Triggers**: button, primary, action, component
**Added**: 2026-01-06
**Source**: session:2026-01-06
```

**关键优势**：学习与它们适用的技能一起存储，当该技能激活时自动加载。

---

## 工作原理

### 1. 信号检测（增强版 - v4.1）

Reflect 识别对话中的信号并**捕获完整上下文**：

**⚠️ 关键：上下文必须包含问题，而不仅仅是修复**

当用户解释问题时，例如：
```
User: "When I use voice control, it always gives me 'command not recognized'"
```

系统必须捕获：
- **CONTEXT**: "When using voice control with skill commands"（情况）
- **LEARNING**: "Voice dictation can mangle command syntax - type commands or use clipboard"（修复）
- **SKILL**: 如果提到了技能名称（例如，"the detector skill"），则路由到那里

**不要**只存储：`"always command not recognized"` ← 这失去了所有意义！

---

**纠正（高置信度）**
```
User: "No, don't use that button. Use our <Button variant='primary'> component."
      → CONTEXT: User corrected button component usage in settings page
      → LEARNING: Always use Button component with variant='primary' from design system
      → SKILL: frontend (auto-detected)
      → CONFIDENCE: high
```

**规则（高置信度）**
```
User: "Always use the logger module instead of console.log"
      → CONTEXT: User established logging convention for the project
      → LEARNING: Use logger module for all logging, never console.log
      → SKILL: tech-lead (auto-detected)
      → CONFIDENCE: high
```

**问题报告（高置信度）- 新增！**
```
User: "The detector skill doesn't recognize commands when I use voice input"
      → CONTEXT: Voice dictation causes command parsing issues
      → LEARNING: Voice input mangles command syntax - recommend typing or clipboard
      → SKILL: detector (explicit skill name detected!)
      → CONFIDENCE: high
```

**批准（中等置信度）**
```
User: "Perfect! That's exactly how our API patterns should look."
      → CONTEXT: User approved API response structure pattern
      → LEARNING: Continue using this API pattern structure with status, data, error fields
      → SKILL: backend (auto-detected)
      → CONFIDENCE: medium
```

### 2. 技能自动检测（增强版 - v4.1）

学习使用**基于优先级的检测系统**进行路由：

#### 优先级 1：显式技能名称提及（最高优先级）
如果用户按名称提及技能，则直接路由到该技能：
```
"the detector skill doesn't work" → detector skill
"increment-planner has a bug" → increment-planner skill
"service-connect is failing" → service-connect skill
```

**检测模式**：`(the\s+)?(\w+[-\w]*)\s+(skill|command|agent)`

#### 优先级 2：基于关键字的检测
如果没有提及显式技能，则使用关键字匹配：

| 技能 | 关键字 |
|-------|----------|
| `architect` | architecture, system design, adr, microservices, api design, schema |
| `tech-lead` | code review, best practices, refactoring, technical debt, solid |
| `qa-lead` | test strategy, qa, quality gates, regression, tdd, bdd |
| `security` | security, owasp, authentication, authorization, encryption |
| `frontend` | react, vue, component, ui, css, tailwind, button, form |
| `backend` | api, endpoint, route, rest, graphql, server, middleware |
| `database` | database, sql, query, schema, migration, prisma, postgres |
| `testing` | test, spec, mock, vitest, jest, playwright, cypress |
| `devops` | docker, kubernetes, ci/cd, pipeline, deploy, github actions |
| `infrastructure` | terraform, iac, aws, azure, gcp, serverless |
| `performance` | performance, optimization, profiling, caching, latency |
| `docs-writer` | documentation, readme, api docs, technical writing |

#### 优先级 3：分类回退
如果没有匹配的技能，则路由到分类记忆（`.specweave/memory/{category}.md`）

### 3. 学习格式

每个学习的结构：

```typescript
interface Learning {
  id: string;           // LRN-YYYYMMDD-XXXX
  timestamp: string;    // ISO 8601
  type: 'correction' | 'rule' | 'approval';
  confidence: 'high' | 'medium' | 'low';
  content: string;      // The actual learning
  context?: string;     // What triggered it
  triggers: string[];   // Keywords for matching
  source: string;       // session:YYYY-MM-DD
}
```

### 4. 记忆持久化

学习被写入技能特定的 MEMORY.md 文件：

```markdown
# Skill Memory: frontend

> Auto-generated by SpecWeave Reflect v4.0
> Last updated: 2026-01-06T10:30:00Z
> Skill: frontend

## Learned Patterns

### LRN-20260106-A1B2 (correction, high)
**Content**: Always use `<Button variant='primary'>` from design system
**Context**: User corrected button usage
**Triggers**: button, primary, component
**Added**: 2026-01-06
**Source**: session:2026-01-06

### LRN-20260105-C3D4 (rule, high)
**Content**: Use PascalCase for component files: `UserProfile.tsx`
**Triggers**: component, naming, file, tsx
**Added**: 2026-01-05
**Source**: session:2026-01-05
```

---

## 用法

### 手动反思

完成工作后，手动触发反思：

```bash
# 反思当前会话（自动检测技能）
/sw:reflect

# 针对特定技能进行反思
/sw:reflect --skill frontend

# 带聚焦提示的反思
/sw:reflect "Focus on the database query patterns we discussed"
```

### 技能特定反思

将学习直接路由到技能：

```bash
# 添加学习到 frontend 技能
/sw:reflect --skill frontend "Always use shadcn Button component"

# 添加学习到 testing 技能
/sw:reflect --skill testing "Use vi.fn() not jest.fn() with Vitest"

# 添加学习到 architect 技能
/sw:reflect --skill architect "Prefer event-driven over request-response"
```

### 查看技能记忆

```bash
# 列出所有技能及其记忆数量
/sw:reflect-status

# 查看特定技能的学习
cat ~/.claude/plugins/marketplaces/specweave/plugins/specweave/skills/frontend/MEMORY.md
```

### 状态仪表板输出（用于 `/sw:reflect-status`）

生成反思状态仪表板时，请遵循以下增强格式：

#### 第 1 节：配置（与之前相同）
显示反思启用状态、自动反思、日期、阈值。

#### 第 2 节：🎯 学习重点 - 反思学习什么

**关键**：本节必须清楚显示每个分类**学习什么**。

对于 `.specweave/memory/` 中的每个记忆文件：
1. **统计学习数量**（以 `- ` 或 `- ✗→✓` 开头的行）
2. **计算百分比**，占总学习数
3. **生成可视化条**（10 个块：`■` 表示填充，`□` 表示空）
4. **添加描述**，解释此分类捕获什么

**格式**：
```
Project Skills (.specweave/memory/):
  • general.md         12 learnings  ■■■■■■□□□□ 40%
    └─ Project conventions, file organization, tooling preferences

  • testing.md          8 learnings  ■■■■□□□□□□ 27%
    └─ Test patterns, mocking, framework usage (Vitest, Playwright)
```

**分类描述**（使用这些确切的描述）：

| 文件 | 描述 |
|------|-------------|
| `general.md` | Project conventions, file organization, tooling preferences |
| `testing.md` | Test patterns, mocking, framework usage (Vitest, Playwright) |
| `api-patterns.md` | API design, endpoint patterns, REST/GraphQL conventions |
| `database.md` | Query patterns, schema design, ORM usage, migrations |
| `git.md` | Commit messages, branching, Git workflows |
| `logging.md` | Logger usage, log levels, structured logging |
| `component-usage.md` | UI component patterns, styling, component composition |
| `deployment.md` | Deploy commands, CI/CD, service configuration |
| `security.md` | Auth patterns, validation, secrets management |
| `structure.md` | File/module organization, import patterns |

#### 第 3 节：最近活动

显示最后修改的文件并提取带有置信度级别的最近学习。

#### 第 4 节：命令

显示可用命令及上下文感知提示（例如，启用时显示 "already on"）。

#### 第 5 节：摘要段落

以简单的英文摘要结尾，例如：
```
The reflection system is actively learning from your corrections. Auto-reflection
is enabled, so learnings will be automatically captured when you end sessions.

You have 30 learnings across 5 categories with recent activity in general
project rules and API patterns.
```

### 自动反思

在会话结束时启用自动反思：

```bash
# 启用自动反思（通过 stop hook）
/sw:reflect-on

# 禁用自动反思
/sw:reflect-off

# 检查反思状态
/sw:reflect-status
```

启用后，stop hook 自动：
1. 分析会话记录
2. 提取纠正和批准
3. 自动检测相关技能
4. 更新技能 MEMORY.md 文件
5. 对非技能学习回退到分类记忆

---

## 更新期间的记忆合并

### `specweave refresh-marketplace` 时会发生什么

1. **步骤 1**：下载最新的 marketplace
2. **步骤 2**：安装插件
3. **步骤 3**：将技能复制到安装位置
4. **步骤 4**：**合并技能记忆**
   - 读取用户现有的 MEMORY.md 文件
   - 读取 marketplace 中的任何新默认学习
   - 合并：用户学习 + 新默认值（去重）
   - 写入合并结果
   - 在 `.memory-backups/` 中创建备份
5. **步骤 5**：更新指令文件

### 合并规则

| 场景 | 结果 |
|----------|--------|
| 用户有学习，marketplace 没有 | **用户学习被保留** |
| Marketplace 有学习，用户没有 | **学习被添加** |
| 两者都有相似的学习 | **保留用户的版本**（去重） |
| 两者都有不同的学习 | **两者都保留** |

### 去重策略

如果满足以下条件，则认为学习是重复的：
- 内容是另一个的子字符串（重叠 >50%）
- 触发器有 >50% 的关键字重叠
- 相同的 ID（完全匹配）

---

## 配置

### 全局设置

在 `~/.claude/settings.json` 中：

```json
{
  "reflect": {
    "enabled": true,
    "autoReflect": false,
    "confidenceThreshold": "medium",
    "maxLearningsPerSession": 10,
    "maxLearningsPerSkill": 50
  }
}
```

### 项目设置

在 `.specweave/config.json` 中：

```json
{
  "reflect": {
    "enabled": true,
    "autoReflect": true,
    "categories": [
      "component-usage",
      "api-patterns",
      "testing",
      "deployment",
      "security",
      "database"
    ]
  }
}
```

---

## 置信度级别

| 级别 | 信号类型 | 示例 | 操作 |
|-------|------------|---------|--------|
| **High** | 显式纠正 | "No, use X instead of Y" | 自动添加到技能记忆 |
| **High** | 显式规则 | "Always do X" | 自动添加到技能记忆 |
| **Medium** | 批准/确认 | "Perfect!" | 以较低优先级添加 |
| **Low** | 观察 | 模式运行良好 | 排队等待审核 |

---

## 分类回退

当学习不匹配任何技能时，它会进入分类记忆：

| 分类 | 描述 | 触发器 |
|----------|-------------|----------|
| `component-usage` | UI 组件模式 | button, component, ui, style |
| `api-patterns` | API 设计和端点 | api, endpoint, route, rest |
| `database` | 查询模式、schema | query, database, sql, schema |
| `testing` | 测试模式和覆盖率 | test, spec, coverage, mock |
| `deployment` | 部署命令和配置 | deploy, wrangler, vercel, ci |
| `security` | 认证、验证、密钥 | auth, security, validation |
| `structure` | 文件/模块组织 | file, path, import, module |
| `general` | 其他所有内容 | (fallback) |

分类记忆位置：
- **项目**：`.specweave/memory/{category}.md`
- **全局**：`~/.specweave/memory/{category}.md`

---

## 与自动模式集成

当启用反思运行 `/sw:auto` 时：

```
1. 启动自动会话
      ↓
2. Claude 执行任务
      ↓
3. 用户做出纠正（如果有）
      ↓
4. 会话完成（所有任务完成）
      ↓
5. Stop hook 触发
      ↓
6. Reflect 分析记录
      ↓
7. 从学习中自动检测技能
      ↓
8. 按技能更新 MEMORY.md 文件
      ↓
9. 会话以摘要结束
```

---

## API 参考 (TypeScript)

```typescript
import {
  // Path resolution
  getSkillsDirectory,
  getSkillMemoryPath,
  listSkills,
  skillExists,
  isClaudeCodeEnvironment,

  // Memory operations
  readMemoryFile,
  writeMemoryFile,
  addLearning,
  mergeMemoryFiles,

  // Reflection management
  detectSkill,
  processSignals,
  reflectOnSkill,
  getSkillLearnings,
  getReflectionStats,
} from 'specweave/core/reflection';

// Add learning to a skill
reflectOnSkill('frontend', [
  { content: 'Use Button component from design system', type: 'correction' }
]);

// Get all learnings for a skill
const learnings = getSkillLearnings('frontend');

// Get stats across all skills
const stats = getReflectionStats();
console.log(`Total learnings: ${stats.totalLearnings}`);
```

---

## 最佳实践

### 对于纠正

**良好的纠正（高信号）**：
```
"Never use that approach. Always use X because..."
"Don't create custom components. We have a design system..."
"Wrong pattern. The correct way is..."
```

**薄弱的纠正（低信号）**：
```
"Hmm, maybe try something else?"
"That doesn't look quite right"
```

### 对于批准

**强烈的批准（被捕获）**：
```
"Perfect! That's exactly how we do it."
"This is the right pattern, well done."
"Yes, always follow this approach."
```

**中性的（不被捕获）**：
```
"OK"
"Sure"
"Proceed"
```

### 记忆组织

1. **技能特定** 学习进入技能的 MEMORY.md
2. **分类** 学习进入 `.specweave/memory/{category}.md`
3. **全局** 学习进入 `~/.specweave/memory/`

---

## 隐私与安全

- 记忆文件仅包含**模式和学习**，不包含原始对话
- 永远不会存储敏感数据（凭证、密钥）
- 如果需要，记忆文件可以被 gitignored
- 可用的清除命令：
  - `/sw:reflect-clear` - 清除所有学习
  - `/sw:reflect-clear --skill frontend` - 清除特定技能
  - `/sw:reflect-clear --learning LRN-XXX` - 删除特定学习

---

## 故障排除

### 学习未持久化

1. 检查反思是否已启用：`/sw:reflect-status`
2. 验证技能目录是否存在：
   ```bash
   # Claude Code
   ls ~/.claude/plugins/marketplaces/specweave/plugins/specweave/skills/

   # Non-Claude
   ls .specweave/plugins/specweave/skills/
   ```
3. 检查文件权限
4. 查看日志：`.specweave/logs/reflect/`

### 错误的技能检测

强制路由到特定技能：
```bash
/sw:reflect --skill frontend "Use Button component"
```

### 记忆未加载

1. 验证技能是否存在 MEMORY.md
2. 检查技能是否已激活（关键字匹配）
3. 在 marketplace 刷新后重启 Claude Code

### 回滚学习

```bash
# 查看备份
ls ~/.claude/plugins/.../skills/frontend/.memory-backups/

# 从备份恢复
cp ~/.claude/plugins/.../skills/frontend/.memory-backups/MEMORY-2026-01-05T10-30-00.md \
   ~/.claude/plugins/.../skills/frontend/MEMORY.md
```

---

## 从 v3.0 迁移

如果您有旧集中式格式的学习：

1. **集中式记忆文件仍然受支持** 作为分类回退
2. **新学习自动进入技能特定的 MEMORY.md**
3. **无需迁移** - 两个系统协同工作

要手动迁移旧学习：
```bash
# 查看旧的集中式记忆
cat .specweave/memory/component-usage.md

# 添加到特定技能
/sw:reflect --skill frontend "Learning content from old file"
```

---

## 摘要

Reflect v4.0 实现**纠正一次，到处应用**：

1. 在会话期间进行纠正
2. Reflect 捕获并路由到技能
3. 未来会话加载技能记忆
4. Claude 自动应用学习到的模式
5. Marketplace 更新保留您的学习

**没有嵌入。没有向量数据库。** 只是按技能组织的、累积知识的 markdown 文件。
