---
name: skillcraft
description: 设计和构建 OpenClaw 技能。当要求 \"制作/构建/制作技能\"、将临时功能提取到技能中，或打包脚本/指令以供重用时使用。涵盖 OpenClaw 特定的集成（工具调用、内存、消息路由、cron、canvas、节点）和 ClawHub 发布。
metadata:
  openclaw:
    emoji: 🧶
tags:
- 即时通讯
- 自动化
---

# Skillcraft — OpenClaw Skill Designer

创建 OpenClaw 技能的 opinionated 指南。专注于 **OpenClaw 特定集成** —— 消息路由、cron 调度、内存持久性、通道格式化、frontmatter 门控 —— 而不是通用编程建议。

**文档：** <https://docs.openclaw.ai/tools/skills> · <https://docs.openclaw.ai/tools/creating-skills>

## 模型说明

此技能是为前沿类模型（Opus、Sonnet）编写的。如果你正在运行更便宜的模型并发现某个阶段规范不足，请自行扩展 —— 设计序列是一个支架，而不是脚本。更便宜的模型应该：

- 在架构设计之前更仔细地阅读 `{baseDir}/patterns/` 中的模式文件
- 在阶段 2（能力发现）上花费更多时间 —— 显式枚举 OpenClaw 功能
- 在阶段 4（规范）中更加系统化 —— 在实施之前写出完整结构
- 在不确定任何 OpenClaw 功能时查阅 <https://docs.openclaw.ai>

---

## 设计序列

### 阶段 0：清单（仅提取）

如果从零开始构建则跳过。在将现有功能（脚本、TOOLS.md 部分、对话模式、重复指令）打包到技能中时使用。

收集存在的内容、所在位置、哪些有效、哪些脆弱。然后继续阶段 1。

### 阶段 1：问题理解

与用户一起解决：

1. **此技能做什么？**（一句话）
2. **何时应该加载？** 示例短语、任务中触发器、计划触发器
3. **成功是什么样的？** 每个示例的具体结果

### 阶段 2：能力发现

#### 通用性

尽早询问：**这是针对你的设置，还是应该在任何 OpenClaw 实例上工作？**

| Choice | Implications |
|--------|-------------|
| **Universal** | 通用路径，无本地假设，ClawHub-ready |
| **Particular** | 可以引用本地技能、工具、工作空间配置 |

#### 技能协同（仅特定）

扫描系统提示中的 `<available_skills>` 以查找互补功能。阅读有前景的技能以理解组合机会。

#### OpenClaw 功能

根据技能的需求查看文档。组合地思考 —— OpenClaw 的原语以强大的方式组合。要检查的关键文档：

| Need | Doc |
|------|-----|
| Messages | `/concepts/messages` |
| Cron/scheduling | `/automation/cron-jobs` |
| Subagents | `/tools/subagents` |
| Browser | `/tools/browser` |
| Canvas UI | `/tools/` (canvas) |
| Node devices | `/nodes/` |
| Slash commands | `/tools/slash-commands` |

有关组合这些的灵感，请参见 `{baseDir}/patterns/composable-examples.md`。

### 阶段 3：架构

基于阶段 1-2，识别哪些模式适用：

| If the skill... | Pattern |
|-----------------|---------|
| Wraps a CLI tool | `{baseDir}/patterns/cli-wrapper.md` |
| Wraps a web API | `{baseDir}/patterns/api-wrapper.md` |
| Monitors and notifies | `{baseDir}/patterns/monitor.md` |

加载所有适用的并综合。大多数技能组合模式。

**脚本与指令拆分：** 脚本处理确定性机制（API 调用、数据收集、文件处理）。SKILL.md 指令处理判断（解释结果、选择方法、组合输出）。边界是：不太智能的系统能否可靠地完成此操作？如果是 → 脚本。

### 阶段 4：设计规范

向用户展示提议的架构以进行审查：

1. **技能结构** —— 文件和目录
2. **SKILL.md 大纲** —— 章节和关键内容
3. **组件** —— 脚本、模块、包装器
4. **状态** —— 无状态、会话有状态或持久（以及它存在于何处）
5. **OpenClaw 集成** —— 哪些功能，它们如何交互
6. **秘密** —— 环境变量、密钥链、配置文件（在设置部分中记录，从不硬编码）

**状态位置：**
- `<workspace>/memory/` —— 用户面向的上下文
- `{baseDir}/state.json` —— 技能内部状态（随技能一起移动）
- `<workspace>/state/<skill>.json` —— 公共工作空间区域中的技能状态

如果提取：包括迁移说明（什么移动，什么工作空间文件需要更新）。

**验证：** 它是否处理所有阶段 1 示例？有任何矛盾吗？边缘情况？

迭代直到用户满意。这是设计问题廉价地浮现的地方。

### 阶段 5：实施

**默认：同一会话。** 与用户一起逐步完成规范，并在每一步进行审查。仅将子代理移交保留给复杂的脚本子组件 —— SKILL.md 和集成逻辑保留在主会话中。

1. 创建技能目录 + SKILL.md 骨架（frontmatter + 章节）
2. 脚本（如果有） —— 让它们工作并经过测试
3. SKILL.md 正文 —— 完整指令
4. 针对阶段 1 示例进行测试

如果提取：更新工作空间文件，清理旧位置，验证独立操作。

---

## 制作 Frontmatter

Frontmatter 决定可发现性和门控。格式遵循带有 OpenClaw 扩展的 [AgentSkills](https://agentskills.io) 规范。

```yaml
---
name: my-skill
description: [description optimised for discovery — see below]
homepage: https://github.com/user/repo  # optional
metadata: {"openclaw":{"emoji":"🔧","requires":{"bins":["tool"],"env":["API_KEY"]},"primaryEnv":"API_KEY","install":[...]}}
---
```

**关键：** `metadata` 必须是**单行** JSON 对象（解析器限制）。

### 描述 —— 为发现而写

描述决定技能是否被加载。包括：
- **核心能力** —— 它做什么
- **触发关键词** —— 用户会说到的术语
- **上下文** —— 它适用的情况

测试：代理是否会为你阶段 1 的每个示例短语选择此技能？

### Frontmatter 键

| Key | Purpose |
|-----|---------|
| `name` | 技能标识符（必需） |
| `description` | 发现文本（必需） |
| `homepage` | 文档/仓库的 URL |
| `user-invocable` | `true`/`false` —— 作为斜杠命令公开（默认：true） |
| `disable-model-invocation` | `true`/`false` —— 从模型提示中排除（默认：false） |
| `command-dispatch` | `tool` —— 绕过模型，直接分派到工具 |
| `command-tool` | 直接分派的工具名称 |
| `command-arg-mode` | `raw` —— 将原始参数转发给工具 |

### 元数据门控

OpenClaw 在加载时使用 `metadata.openclaw` 过滤技能：

| Field | Effect |
|-------|--------|
| `always: true` | 跳过所有门，始终加载 |
| `emoji` | 在 macOS Skills UI 中显示 |
| `os` | 平台过滤器（`darwin`、`linux`、`win32`） |
| `requires.bins` | 所有必须存在于 PATH 上 |
| `requires.anyBins` | 至少一个必须存在 |
| `requires.env` | 环境变量必须存在或在配置中 |
| `requires.config` | 配置路径必须为真值 |
| `primaryEnv` | 映射到 `skills.entries.<name>.apiKey` |
| `install` | 自动设置的安装程序规范（brew/node/go/uv/download） |

**沙盒说明：** `requires.bins` 在加载时检查**主机**。如果沙盒化，二进制文件也必须存在于容器内。

### 令牌预算

每个符合条件的技能都会向系统提示添加约 97 个字符 + 名称 + 描述 + 位置路径。保持描述信息丰富但不臃肿 —— 每个字符在每次轮次上都会花费令牌。

### 安装规范

```json
"install": [
  {"id": "brew", "kind": "brew", "formula": "tap/tool", "bins": ["tool"], "label": "Install via brew"},
  {"id": "npm", "kind": "node", "package": "tool", "bins": ["tool"]},
  {"id": "uv", "kind": "uv", "package": "tool", "bins": ["tool"]},
  {"id": "go", "kind": "go", "package": "github.com/user/tool@latest", "bins": ["tool"]},
  {"id": "dl", "kind": "download", "url": "https://...", "archive": "tar.gz"}
]
```

## 路径约定

| Token | Meaning |
|-------|---------|
| `{baseDir}` | 此技能的目录（OpenClaw 在运行时解析） |
| `<workspace>/` | 代理的工作空间根目录 |

- 对技能内部引用（脚本、状态、模式）使用 `{baseDir}`
- 对工作空间文件（TOOLS.md、memory/ 等）使用 `<workspace>/`
- 从不硬编码绝对路径 —— 工作空间是可移植的
- 对于子代理场景，在任务描述中包括路径上下文（沙盒挂载不同）

## 参考

- 模式文件：`{baseDir}/patterns/`（cli-wrapper、api-wrapper、monitor、composable-examples）
- OpenClaw 文档：<https://docs.openclaw.ai/tools/skills>
- ClawHub：<https://clawhub.com>
