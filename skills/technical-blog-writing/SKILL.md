---
name: technical-blog-writing
description: 技术博客文章写作，包含结构、代码示例和开发者受众惯例。 涵盖文章类型、代码格式化、解释深度和开发者特定的互动模式。 用于：工程博客、开发教程、技术写作、开发者内容、文档文章。 触发词：technical blog, dev blog, engineering blog, technical writing, developer tutorial, tech post, code tutorial, programming blog, developer content, technical article, engineering post, coding tutorial, technical content
allowed-tools: Bash(infsh *)
tags:
- 文档
---

# 技术博客写作

通过 [inference.sh](https://inference.sh) CLI 撰写面向开发者的技术博客文章。

## 快速开始

```bash
curl -fsSL https://cli.inference.sh | sh && infsh login

# 研究主题深度
infsh app run exa/search --input '{
  "query": "building REST API Node.js best practices 2024 tutorial"
}'

# 生成头图
infsh app run infsh/html-to-image --input '{
  "html": "<div style=\"width:1200px;height:630px;background:linear-gradient(135deg,#0f172a,#1e293b);display:flex;align-items:center;padding:60px;font-family:ui-monospace,monospace;color:white\"><div><p style=\"font-size:18px;color:#38bdf8;margin:0\">// engineering blog</p><h1 style=\"font-size:48px;margin:16px 0;font-weight:800;font-family:system-ui;line-height:1.2\">How We Reduced API Latency by 90% with Edge Caching</h1><p style=\"font-size:20px;opacity:0.6;font-family:system-ui\">A deep dive into our CDN architecture</p></div></div>"
}'
```

## 文章类型

### 1. 教程 / 操作指南

分步说明。读者应该能够跟着做并构建出东西。

```
结构：
1. 我们要构建什么（附截图/演示）
2. 前置条件
3. 步骤 1：环境搭建
4. 步骤 2：核心实现
5. 步骤 3：...
6. 完整代码（GitHub 链接）
7. 下一步 / 扩展
```

| 规则 | 原因 |
|------|-----|
| 先展示最终结果 | 读者知道是否值得继续阅读 |
| 明确列出前置条件 | 不要浪费错误受众的时间 |
| 每个代码块都应该可运行 | 复制-粘贴-运行就是测试 |
| 解释 "为什么" 而不仅仅是 "怎么做" | 解释原理的教程更容易被分享 |
| 包含错误处理 | 真实代码会有错误 |
| 链接到完整的代码仓库 | 教程结束后可供参考 |

### 2. 深度解析 / 原理说明

深入解释一个概念、技术或架构决策。

```
结构：
1. [概念] 是什么以及为什么你应该关心？
2. 它是如何工作的（简化的心智模型）
3. 它是如何工作的（详细机制）
4. 真实世界的例子
5. 权衡以及何时不使用它
6. 延伸阅读
```

### 3. 事后分析 / 事故报告

描述出了什么问题、为什么以及是如何修复的。

```
结构：
1. 摘要（发生了什么、影响、持续时间）
2. 事件时间线
3. 根因分析
4. 实施的修复
5. 我们正在做什么来防止再次发生
6. 经验教训
```

### 4. 基准测试 / 对比

数据驱动的工具、方法或架构对比。

```
结构：
1. 我们对比了什么以及为什么
2. 方法论（以便结果可复现）
3. 带图表/表格的结果
4. 分析（这些数字意味着什么）
5. 推荐（附带注意事项）
6. 原始数据 / 可复现说明
```

### 5. 架构 / 系统设计

解释一个系统是如何构建的以及为什么做出这些决策。

```
结构：
1. 我们需要解决的问题
2. 约束和需求
3. 考虑的选项
4. 选择的架构（附图表）
5. 我们接受的权衡
6. 结果和经验教训
```

## 面向开发者的写作规则

### 语气和风格

| 应该 | 不应该 |
|----|-------|
| 直接了当："Use connection pooling" | "You might want to consider using..." |
| 承认权衡："This adds complexity" | 假装你的解决方案是完美的 |
| 对团队决策使用 "we" | "I single-handedly architected..." |
| 具体数字："reduced p99 from 800ms to 90ms" | "significantly improved performance" |
| 引用来源和基准测试 | 做出无来源的断言 |
| 承认替代方案 | 假装你的方式是唯一的方式 |

### 开发者讨厌什么

```
❌ "In today's fast-paced world of technology..."（废话）
❌ "As we all know..."（如果我们都知道，你为什么还要写？）
❌ "Simply do X"（如果你在读教程，没有什么是简单的）
❌ "It's easy to..."（轻视读者的经验）
❌ "Obviously..."（如果很明显，就不要写）
❌ 技术内容中的营销语言
❌ 把重点埋在 3 段上下文之下
```

### 代码示例

| 规则 | 原因 |
|------|-----|
| 每个代码块都必须可运行 | 错误的示例会摧毁信任 |
| 展示完整、可工作的示例 | 没有上下文的片段毫无用处 |
| 在围栏代码块中包含语言标识符 | 语法高亮 |
| 在代码后展示输出/结果 | 读者验证理解 |
| 使用真实的变量名 | `calculateTotalRevenue` 而不是 `foo` |
| 在示例中包含错误处理 | 真实代码会处理错误 |
| 固定依赖版本 | "Works with React 18.2" 而不是 "React" |

```
良好的代码块格式：

```python
# 这段代码的作用（一行）
def calculate_retry_delay(attempt: int, base_delay: float = 1.0) -> float:
    """Exponential backoff with jitter."""
    delay = base_delay * (2 ** attempt)
    jitter = random.uniform(0, delay * 0.1)
    return delay + jitter

# 用法
delay = calculate_retry_delay(attempt=3)  # ~8.0-8.8 seconds
```
```

### 解释深度

| 受众信号 | 深度 |
|----------------|-------|
| "Getting started with X" | 解释一切，假设没有先验知识 |
| "Advanced X patterns" | 跳过基础，深入细节 |
| "X vs Y" | 假设对两者都熟悉，专注于差异 |
| "How we built X" | 技术受众，可以跳过基础 |

**在开头明确说明您假设的受众水平：**

```
"This post assumes familiarity with Docker and basic Kubernetes concepts.
If you're new to containers, start with [our intro post]."
```

## 博客文章结构

### 理想结构

```markdown
# 标题（包含主要关键字，说明结果）

[头图或图表]

**TL;DR:** [2-3 句话的摘要，包含关键要点]

## 问题 / 为什么这很重要
[设定读者为什么应该关心 —— 具体，而非泛泛而谈]

## 解决方案 / 我们是如何做的
[核心内容 —— 代码、架构、解释]

### 步骤 1：[第一件事]
[解释 + 代码 + 输出]

### 步骤 2：[第二件事]
[解释 + 代码 + 输出]

## 结果
[数字、基准测试、成果 —— 要具体]

## 权衡和局限性
[诚实地说明缺点 —— 建立信任]

## 结论
[关键要点 + 下一步该做什么]

## 延伸阅读
[3-5 个相关链接]
```

### 按类型划分的字数

| 类型 | 字数 | 原因 |
|------|-----------|-----|
| 快速提示 | 500-800 | 一个概念，一个示例 |
| 教程 | 1,500-3,000 | 分步指南需要细节 |
| 深度解析 | 2,000-4,000 | 彻底探索 |
| 架构文章 | 2,000-3,500 | 图表分担了一些负担 |
| 基准测试 | 1,500-2,500 | 数据和图表承担重任 |

## 图表和视觉元素

### 何时使用图表

| 场景 | 图表类型 |
|----------|-------------|
| 请求流 | 序列图 |
| 系统架构 | 方框-箭头图 |
| 决策逻辑 | 流程图 |
| 数据模型 | ER 图 |
| 性能对比 | 柱状图/折线图 |
| 前后对比 | 并排展示 |

```bash
# 生成架构图
infsh app run infsh/html-to-image --input '{
  "html": "<div style=\"width:1200px;height:600px;background:#0f172a;display:flex;align-items:center;justify-content:center;padding:40px;font-family:system-ui;color:white\"><div style=\"display:flex;gap:40px;align-items:center\"><div style=\"background:#1e293b;border:2px solid #334155;border-radius:8px;padding:24px;text-align:center;width:160px\"><p style=\"font-size:14px;color:#94a3b8;margin:0\">Client</p><p style=\"font-size:18px;font-weight:bold;margin:8px 0 0\">React App</p></div><div style=\"color:#64748b;font-size:32px\">→</div><div style=\"background:#1e293b;border:2px solid #3b82f6;border-radius:8px;padding:24px;text-align:center;width:160px\"><p style=\"font-size:14px;color:#94a3b8;margin:0\">Edge</p><p style=\"font-size:18px;font-weight:bold;margin:8px 0 0\">CDN Cache</p></div><div style=\"color:#64748b;font-size:32px\">→</div><div style=\"background:#1e293b;border:2px solid #334155;border-radius:8px;padding:24px;text-align:center;width:160px\"><p style=\"font-size:14px;color:#94a3b8;margin:0\">API</p><p style=\"font-size:18px;font-weight:bold;margin:8px 0 0\">Node.js</p></div><div style=\"color:#64748b;font-size:32px\">→</div><div style=\"background:#1e293b;border:2px solid #334155;border-radius:8px;padding:24px;text-align:center;width:160px\"><p style=\"font-size:14px;color:#94a3b8;margin:0\">Database</p><p style=\"font-size:18px;font-weight:bold;margin:8px 0 0\">PostgreSQL</p></div></div></div>"
}'

# 生成基准测试图表
infsh app run infsh/python-executor --input '{
  "code": "import matplotlib.pyplot as plt\nimport matplotlib\nmatplotlib.use(\"Agg\")\n\nfig, ax = plt.subplots(figsize=(12, 6))\nfig.patch.set_facecolor(\"#0f172a\")\nax.set_facecolor(\"#0f172a\")\n\ntools = [\"Express\", \"Fastify\", \"Hono\", \"Elysia\"]\nrps = [15000, 45000, 62000, 78000]\ncolors = [\"#64748b\", \"#64748b\", \"#3b82f6\", \"#64748b\"]\n\nax.barh(tools, rps, color=colors, height=0.5)\nfor i, v in enumerate(rps):\n    ax.text(v + 1000, i, f\"{v:,} req/s\", va=\"center\", color=\"white\", fontsize=14)\n\nax.set_xlabel(\"Requests per second\", color=\"white\", fontsize=14)\nax.set_title(\"HTTP Framework Benchmark (Hello World)\", color=\"white\", fontsize=18, fontweight=\"bold\")\nax.tick_params(colors=\"white\", labelsize=12)\nax.spines[\"top\"].set_visible(False)\nax.spines[\"right\"].set_visible(False)\nax.spines[\"bottom\"].set_color(\"#334155\")\nax.spines[\"left\"].set_color(\"#334155\")\nplt.tight_layout()\nplt.savefig(\"benchmark.png\", dpi=150, facecolor=\"#0f172a\")\nprint(\"Saved\")"
}'
```

## 分发

### 开发者在哪里阅读

| 平台 | 格式 | 如何发布 |
|----------|--------|-------------|
| 您的博客 | 完整文章 | 主要 —— 拥有您的内容 |
| Dev.to | 交叉发布（canonical URL 指向您的博客） | Markdown 导入 |
| Hashnode | 交叉发布（canonical URL） | Markdown 导入 |
| Hacker News | 链接提交 | Show HN 用于项目，tell HN 用于故事 |
| Reddit (r/programming, r/webdev, etc.) | 链接或讨论 | 遵守 subreddit 规则 |
| Twitter/X | 线程摘要 + 链接 | 参见 twitter-thread-creation 技能 |
| LinkedIn | 改编版本 + 链接 | 参见 linkedin-content 技能 |

```bash
# 交叉发布线程到 X
infsh app run x/post-create --input '{
  "text": "New blog post: How We Reduced API Latency by 90%\n\nThe short version:\n→ Moved computation to edge\n→ Aggressive cache-control headers\n→ Eliminated N+1 queries\n\np99 went from 800ms to 90ms.\n\nFull deep dive with code: [link]"
}'
```

## 常见错误

| 错误 | 问题 | 修复 |
|---------|---------|-----|
| 没有 TL;DR | 忙碌的开发者还没看到重点就离开了 | 在顶部放 2-3 句话的摘要 |
| 代码示例损坏 | 摧毁所有可信度 | 发布前测试每个代码块 |
| 没有固定版本 | 6 个月后代码就坏了 | "Works with Node 20, React 18.2" |
| "Simply do X" | 居高临下，傲慢 | 删除 "simply"、"just"、"easily" |
| 架构没有图表 | 用文字墙描述系统 | 一张图表 > 500 字的描述 |
| 营销语气 | 开发者会立即失去兴趣 | 直接、技术、诚实 |
| 没有权衡部分 | 读起来像有偏见的营销 | 始终讨论缺点 |
| 内容前有巨大的引言 | 读者跳出 | 在 2-3 段内进入正题 |
| 依赖未固定 | 教程对未来的读者失效 | 固定版本，注明写作日期 |
| 没有 "延伸阅读" | 死胡同，没有上下文 | 3-5 个链接以加深理解 |

## 相关技能

```bash
npx skills add inferencesh/skills@seo-content-brief
npx skills add inferencesh/skills@content-repurposing
npx skills add inferencesh/skills@og-image-design
```

浏览所有应用：`infsh app list`
