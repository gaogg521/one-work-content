---
name: skill-creator
description: 创建有效 skill 的指南。当用户想要创建新 skill（或更新现有 skill）以扩展 Claude 的能力，提供专业领域知识、工作流或工具集成时，应使用此 skill。
license: 完整条款见 LICENSE.txt
---

# Skill Creator

此 skill 提供创建有效 skill 的指导。

## 关于 Skills

Skills 是模块化的、自包含的包，通过提供专业知识、工作流和工具来扩展 Claude 的能力。将它们视为特定领域或任务的"入职指南"——它们将 Claude 从通用 agent 转变为具备模型无法完全拥有的程序性知识的专业 agent。

### Skills 提供什么

1. 专业工作流 —— 特定领域的多步骤程序
2. 工具集成 —— 与特定文件格式或 API 协同工作的指令
3. 领域专业知识 —— 公司特定的知识、schema、业务逻辑
4. 捆绑资源 —— 用于复杂和重复任务的脚本、参考资料和资产

## 核心原则

### 简洁是关键

上下文窗口是一种公共资源。Skills 与 Claude 需要的其他所有内容共享上下文窗口：系统提示、对话历史、其他 Skills 的元数据，以及实际的用户请求。

**默认假设：Claude 已经很聪明。** 只添加 Claude 还没有的上下文。对每条信息提出质疑："Claude 真的需要这个解释吗？"以及"这段文字是否值得它的 token 成本？"

优先使用简洁示例而非冗长解释。

### 设置适当的自由度

将特定性与任务的脆弱性和可变性相匹配：

**高自由度（基于文本的指令）**：当多种方法都有效、决策取决于上下文或由启发式方法指导方法时使用。

**中等自由度（带参数的伪代码或脚本）**：当存在首选模式、允许一些变化或配置影响行为时使用。

**低自由度（特定脚本，参数少）**：当操作脆弱且容易出错、一致性至关重要或必须遵循特定序列时使用。

将 Claude 想象成在探索一条路径：有悬崖的窄桥需要特定的护栏（低自由度），而开阔的田野允许多条路线（高自由度）。

### Skill 的解剖结构

每个 skill 由一个必需的 SKILL.md 文件和可选的捆绑资源组成：

```
skill-name/
├── SKILL.md (必需)
│   ├── YAML frontmatter 元数据 (必需)
│   │   ├── name: (必需)
│   │   └── description: (必需)
│   └── Markdown 指令 (必需)
└── Bundled Resources (可选)
    ├── scripts/          - 可执行代码 (Python/Bash/等)
    ├── references/       - 根据需要加载到上下文中的文档
    └── assets/           - 用于输出中的文件（模板、图标、字体等）
```

#### SKILL.md (必需)

每个 SKILL.md 包含：

- **Frontmatter** (YAML)：包含 `name` 和 `description` 字段。这是 Claude 读取以确定何时使用该 skill 的唯一字段，因此非常重要的是要清楚全面地描述 skill 是什么以及何时应该使用它。
- **Body** (Markdown)：使用 skill 的指令和指导。仅在 skill 触发后加载（如果有的话）。

#### 捆绑资源 (可选)

##### 脚本 (`scripts/`)

可执行代码（Python/Bash/等），用于需要确定性可靠性或反复重写的任务。

- **何时包含**：当相同的代码被反复重写或需要确定性可靠性时
- **示例**：用于 PDF 旋转任务的 `scripts/rotate_pdf.py`
- **好处**：token 高效、确定性、可能无需加载到上下文中即可执行
- **注意**：脚本可能仍需要被 Claude 读取以进行修补或环境特定的调整

##### 参考资料 (`references/`)

文档和参考材料，旨在根据需要加载到上下文中，以告知 Claude 的过程和思考。

- **何时包含**：当 Claude 在工作时应该参考的文档
- **示例**：财务 schema 的 `references/finance.md`、公司 NDA 模板的 `references/mnda.md`、公司政策的 `references/policies.md`、API 规范的 `references/api_docs.md`
- **用例**：数据库 schema、API 文档、领域知识、公司政策、详细工作流指南
- **好处**：保持 SKILL.md 精简，仅在 Claude 确定需要时加载
- **最佳实践**：如果文件很大（>10k 字），在 SKILL.md 中包含 grep 搜索模式
- **避免重复**：信息应该存在于 SKILL.md 或 references 文件中，而不是两者都有。优先将详细信息放在 references 文件中，除非它确实是 skill 的核心——这可以保持 SKILL.md 精简，同时使信息可发现而不会占用上下文窗口。只在 SKILL.md 中保留基本的程序性指令和工作流指导；将详细的参考资料、schema 和示例移动到 references 文件中。

##### 资产 (`assets/`)

不打算加载到上下文中，而是在 Claude 产生的输出中使用的文件。

- **何时包含**：当 skill 需要将在最终输出中使用的文件时
- **示例**：品牌资产的 `assets/logo.png`、PowerPoint 模板的 `assets/slides.pptx`、HTML/React 样板代码的 `assets/frontend-template/`、字体的 `assets/font.ttf`
- **用例**：模板、图像、图标、样板代码、字体、被复制或修改的示例文档
- **好处**：将输出资源与文档分离，使 Claude 能够在无需将它们加载到上下文中的情况下使用文件

#### 不应包含在 Skill 中的内容

Skill 应仅包含直接支持其功能的基本文件。不要创建多余的文档或辅助文件，包括：

- README.md
- INSTALLATION_GUIDE.md
- QUICK_REFERENCE.md
- CHANGELOG.md
- 等等。

Skill 应仅包含 AI agent 完成当前工作所需的信息。它不应包含关于创建它的过程的辅助上下文、设置和测试程序、面向用户的文档等。创建额外的文档文件只会增加混乱和困惑。

### 渐进式披露设计原则

Skills 使用三级加载系统来高效管理上下文：

1. **元数据（name + description）** - 始终在上下文中（~100 字）
2. **SKILL.md body** - 当 skill 触发时（<5k 字）
3. **捆绑资源** - 由 Claude 根据需要加载（无限制，因为脚本可以在不读入上下文窗口的情况下执行）

#### 渐进式披露模式

将 SKILL.md body 保持在基本内容内，并少于 500 行以最小化上下文膨胀。当接近此限制时，将内容拆分为单独的文件。在拆分时，从 SKILL.md 中引用它们并清楚描述何时阅读它们，这一点非常重要，以确保 skill 的读者知道它们存在以及何时使用它们。

**关键原则：** 当 skill 支持多种变体、框架或选项时，只在 SKILL.md 中保留核心工作流和选择指导。将变体特定的细节（模式、示例、配置）移动到单独的 reference 文件中。

**模式 1：带参考资料的高级指南**

```markdown
# PDF 处理

## 快速开始

使用 pdfplumber 提取文本：
[code example]

## 高级功能

- **表单填充**：完整指南请参阅 [FORMS.md](FORMS.md)
- **API 参考**：所有方法请参阅 [REFERENCE.md](REFERENCE.md)
- **示例**：常见模式请参阅 [EXAMPLES.md](EXAMPLES.md)
```

Claude 仅在需要时加载 FORMS.md、REFERENCE.md 或 EXAMPLES.md。

**模式 2：按领域组织**

对于具有多个领域的 Skills，按领域组织内容以避免加载不相关的上下文：

```
bigquery-skill/
├── SKILL.md (概览和导航)
└── reference/
    ├── finance.md (收入、计费指标)
    ├── sales.md (机会、管道)
    ├── product.md (API 使用、功能)
    └── marketing.md (活动、归因)
```

当用户询问销售指标时，Claude 只读取 sales.md。

同样，对于支持多种框架或变体的 skill，按变体组织：

```
cloud-deploy/
├── SKILL.md (工作流 + 提供商选择)
└── references/
    ├── aws.md (AWS 部署模式)
    ├── gcp.md (GCP 部署模式)
    └── azure.md (Azure 部署模式)
```

当用户选择 AWS 时，Claude 只读取 aws.md。

**模式 3：条件细节**

显示基本内容，链接到高级内容：

```markdown
# DOCX 处理

## 创建文档

对于新文档，使用 docx-js。
请参阅 [DOCX-JS.md](DOCX-JS.md)。

## 编辑文档

对于简单编辑，直接修改 XML。

**对于修订模式**：请参阅 [REDLINING.md](REDLINING.md)
**对于 OOXML 细节**：请参阅 [OOXML.md](OOXML.md)
```

Claude 仅在用户需要这些功能时才读取 REDLINING.md 或 OOXML.md。

**重要指南：**

- **避免深层嵌套引用** - 保持引用文件与 SKILL.md 只有一级深度。所有引用文件都应直接从 SKILL.md 链接。
- **结构化较长的引用文件** - 对于超过 100 行的文件，在顶部包含目录，以便 Claude 在预览时可以看到完整范围。

## Skill 创建过程

Skill 创建涉及以下步骤：

1. 通过具体示例理解 skill
2. 规划可复用的 skill 内容（脚本、参考资料、资产）
3. 初始化 skill（运行 init_skill.py）
4. 编辑 skill（实现资源并编写 SKILL.md）
5. 打包 skill（运行 package_skill.py）
6. 基于实际使用进行迭代

按顺序执行这些步骤，除非有明确的理由说明它们不适用，否则不要跳过。

### 步骤 1：通过具体示例理解 Skill

仅当 skill 的使用模式已经被清楚理解时才跳过此步骤。即使在处理现有 skill 时，它仍然很有价值。

要创建一个有效的 skill，需要清楚了解 skill 将如何被使用的具体示例。这种理解可以来自直接的用户示例或经过用户反馈验证的生成示例。

例如，在构建 image-editor skill 时，相关问题包括：

- "image-editor skill 应该支持什么功能？编辑、旋转，还有其他吗？"
- "你能举一些这个 skill 将如何被使用的例子吗？"
- "我可以想象用户会要求'去除这张图片的红眼'或'旋转这张图片'。你还能想到其他使用方式吗？"
- "用户会说什么来触发这个 skill？"

为避免让用户不知所措，避免在一条消息中问太多问题。从最重要的问题开始，并根据需要跟进以获得更好的效果。

当对 skill 应支持的功能有了清晰的感觉时，此步骤结束。

### 步骤 2：规划可复用的 Skill 内容

要将具体示例转化为有效的 skill，请通过以下方式分析每个示例：

1. 考虑如何从头开始执行该示例
2. 确定在反复执行这些工作流时，哪些脚本、参考资料和资产会有帮助

示例：在构建一个处理"帮我旋转这个 PDF"等查询的 `pdf-editor` skill 时，分析表明：

1. 旋转 PDF 每次都需要重写相同的代码
2. 在 skill 中存储一个 `scripts/rotate_pdf.py` 脚本会很有帮助

示例：在设计一个处理"帮我构建一个 todo app"或"帮我构建一个追踪步数的 dashboard"等查询的 `frontend-webapp-builder` skill 时，分析表明：

1. 编写前端 webapp 每次都需要相同的 HTML/React 样板代码
2. 在 skill 中存储一个包含样板 HTML/React 项目文件的 `assets/hello-world/` 模板会很有帮助

示例：在构建一个处理"今天有多少用户登录了？"等查询的 `big-query` skill 时，分析表明：

1. 查询 BigQuery 每次都需要重新发现表 schema 和关系
2. 在 skill 中存储一个记录表 schema 的 `references/schema.md` 文件会很有帮助

为了确定 skill 的内容，分析每个具体示例以创建要包含的可复用资源列表：脚本、参考资料和资产。

### 步骤 3：初始化 Skill

此时，是时候实际创建 skill 了。

仅当正在开发的 skill 已经存在且需要迭代或打包时才跳过此步骤。在这种情况下，继续下一步。

从头开始创建新 skill 时，始终运行 `init_skill.py` 脚本。该脚本方便地生成一个新的模板 skill 目录，自动包含 skill 所需的一切，使 skill 创建过程更加高效和可靠。

用法：

```bash
scripts/init_skill.py <skill-name> --path <output-directory>
```

该脚本：

- 在指定路径创建 skill 目录
- 生成一个具有正确 frontmatter 和 TODO 占位符的 SKILL.md 模板
- 创建示例资源目录：`scripts/`、`references/` 和 `assets/`
- 在每个目录中添加可以自定义或删除的示例文件

初始化后，根据需要自定义或删除生成的 SKILL.md 和示例文件。

### 步骤 4：编辑 Skill

编辑（新生成的或现有的）skill 时，请记住，skill 是为另一个 Claude 实例使用的。包含有益且对 Claude 不明显的信息。考虑哪些程序性知识、领域特定的细节或可复用资产可以帮助另一个 Claude 实例更有效地执行这些任务。

#### 学习经过验证的设计模式

根据你的 skill 需求参考这些有用的指南：

- **多步骤过程**：有关顺序工作流和条件逻辑的详细信息，请参阅 references/workflows.md
- **特定输出格式或质量标准**：有关模板和示例模式的详细信息，请参阅 references/output-patterns.md

这些文件包含有效 skill 设计的成熟最佳实践。

#### 从可复用的 Skill 内容开始

要开始实现，请从上面确定的可复用资源开始：`scripts/`、`references/` 和 `assets/` 文件。请注意，此步骤可能需要用户输入。例如，在实现 `brand-guidelines` skill 时，用户可能需要提供要存储在 `assets/` 中的品牌资产或模板，或要存储在 `references/` 中的文档。

添加的脚本必须通过实际运行来测试，以确保没有 bug 并且输出符合预期。如果有很多类似的脚本，只需要测试一个有代表性的样本，以确保它们都能正常工作，同时平衡完成时间。

任何 skill 不需要的示例文件和目录都应该删除。初始化脚本在 `scripts/`、`references/` 和 `assets/` 中创建示例文件以演示结构，但大多数 skill 不需要所有这些。

#### 更新 SKILL.md

**写作指南：** 始终使用祈使/不定式形式。

##### Frontmatter

编写带有 `name` 和 `description` 的 YAML frontmatter：

- `name`：skill 名称
- `description`：这是 skill 的主要触发机制，帮助 Claude 理解何时使用该 skill。
  - 包括 Skill 做什么以及何时使用它的特定触发器/上下文。
  - 将所有"何时使用"信息放在这里 - 不要放在 body 中。body 只在触发后加载，因此 body 中的"何时使用此 Skill"部分对 Claude 没有帮助。
  - docx skill 的示例描述："Comprehensive document creation, editing, and analysis with support for tracked changes, comments, formatting preservation, and text extraction. Use when Claude needs to work with professional documents (.docx files) for: (1) Creating new documents, (2) Modifying or editing content, (3) Working with tracked changes, (4) Adding comments, or any other document tasks"

不要在 YAML frontmatter 中包含任何其他字段。

##### Body

编写使用 skill 及其捆绑资源的指令。

### 步骤 5：打包 Skill

一旦 skill 开发完成，它必须被打包成一个可分发的 .skill 文件，与用户共享。打包过程首先自动验证 skill，以确保它满足所有要求：

```bash
scripts/package_skill.py <path/to/skill-folder>
```

可选的输出目录指定：

```bash
scripts/package_skill.py <path/to/skill-folder> ./dist
```

打包脚本将：

1. **自动验证** skill，检查：

   - YAML frontmatter 格式和必填字段
   - Skill 命名约定和目录结构
   - 描述的完整性和质量
   - 文件组织和资源引用

2. **如果验证通过则打包** skill，创建一个以 skill 命名的 .skill 文件（例如，`my-skill.skill`），其中包含所有文件并保持正确的目录结构以供分发。.skill 文件是一个带有 .skill 扩展名的 zip 文件。

如果验证失败，脚本将报告错误并退出而不创建包。修复任何验证错误，然后再次运行打包命令。

### 步骤 6：迭代

测试 skill 后，用户可能会请求改进。通常这发生在使用 skill 之后，对 skill 的表现有新鲜记忆时。

**迭代工作流：**

1. 在真实任务中使用 skill
2. 注意到困难或低效之处
3. 确定应如何更新 SKILL.md 或捆绑资源
4. 实施更改并再次测试
