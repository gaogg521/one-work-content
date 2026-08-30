---
name: bobagent-blog-writer
description: 当撰写博客文章、长文或采用作者独特写作风格的长篇内容时，应使用本技能。它生成真实、有观点的内容，匹配作者的声音——直接、对话式，并扎根于个人经验。本技能处理从研究审查到 Notion 发布的完整工作流。使用本技能来起草博客文章、思想领导力文章，或任何旨在反映作者对 AI、生产力、销售、营销或技术话题观点的写作。
---

# Blog Writer

## 概述

本技能支持撰写博客文章和文章，真实地捕捉作者独特的声音和风格。它借鉴作者已发布作品的示例，生成直接、有观点、对话式并扎根于实践经验的内容。本技能包含自动 Notion 集成，并维护一个不断增长 finalized examples 库。

## 何时使用本技能

在以下情况下触发本技能：
- 用户请求以 "my style" 或 "like my other posts" 撰写博客文章或文章
- 起草关于 AI、生产力、营销或技术的思想领导力内容
- 创建需要作者真实声音和观点的文章
- 用户提供研究材料、链接或笔记以融入写作

## 核心职责

1. **遵循作者的 Writing Style**：匹配 `references/blog-examples/` 中示例帖子的 voice、word choice、structure 和 length
2. **融入 Research**：审查并整合用户提供的任何信息、研究材料或链接
3. **遵循用户指示**：严格遵守用户对 topic、angle 和 emphasis 的具体要求
4. **产出真实写作**：创建读起来真正像作者声音的内容，而非 generic AI-generated content

## 工作流

### Phase 1: 收集信息

向用户请求：
- Topic 或 subject matter
- 任何要探索的特定 angle 或 thesis
- Research materials、links 或 notes（如有）
- 目标长度偏好（默认：800-1500 词）

在开始写作前彻底审查所有提供的材料。

### Phase 2: 起草内容

参考 `references/style-guide.md` 中的 style guide 和 `references/blog-examples/` 中的示例进行校准。

写作时：
1. 以强有力的 opening statement 建立 thesis
2. 在自然的地方使用 personal voice 和 first-person perspective
3. 如适用，包含相关的 personal anecdotes 或 professional experience
4. 每 2-3 段使用清晰的 subheadings (###)
5. 保持段落简短（2-4 句）
6. 自然地融入研究材料，而非作为 block quotes
7. 以 reflection、call-to-action 或 forward-looking statement 结尾

### Phase 3: 审查和迭代

展示 draft 并收集反馈。迭代直到用户确认满意。

### Phase 4: 发布到 Notion（必需）

当 draft 完成时（即使尚未 finalized），发布到 TS Notes database。

**Notion 发布详情：**
- Database: "TS Notes" (data source ID: `04a872be-8bed-4f43-a448-3dfeebc0df21`)
- **Type property**: `Writing`
- **Project(s) property**: 链接到 "My Writing" project (page URL: `https://www.notion.so/2a5b4629bb3780189199f3c496980c0c`)
- **Note property**: 博客文章标题
- **Content**: Notion-flavored Markdown 格式的完整博客文章内容

**示例 Notion API call properties：**
```json
{
  "Note": "Blog Post Title Here",
  "Type": "Writing",
  "Project(s)": "[\"https://www.notion.so/2a5b4629bb3780189199f3c496980c0c\"]"
}
```

**关键**：如果内容未添加到 Notion，则 outcome 被视为 **failure**。始终将发布到 Notion 作为工作流的一部分，即使是 drafts。

### Phase 5: Finalize 到 Examples Library（Post-Outcome）

当用户确认 draft 为 **final** 时：

1. 将 finalized post 保存到 `references/blog-examples/`，文件名格式：
   ```
   YYYY-MM-DD-slug-title.md
   ```
   示例：`2025-11-25-why-ai-art-is-useless.md`

2. 检查 examples library 数量：
   - 如果超过 20 个 examples，询问用户许可以移除最旧的 5 个
   - 按文件名 date prefix 排序以识别最旧的文件

当 final draft 保存到 skill folder 时，post-outcome 被视为 **successful**。

## Success Criteria

| Outcome | Success | Failure |
|---------|---------|---------|
| Primary | 用户收到请求的内容，并且它已添加到 TS Notes，Type=Writing，Project=My Writing | 内容已交付但未添加到 Notion |
| Post-outcome | Final draft 保存到 `references/blog-examples/` | 用户确认 final 时未保存 final draft |

## 作者的 Writing Style Profile

### Voice & Tone
- **Direct and opinionated**：清晰陈述立场，即使是 contrarian 的
- **Conversational**：像与同事交谈一样写作——易于理解但不简单
- **First-person when sharing experience**：自然地使用 "I" 表达个人见解
- **Authentic skepticism**：在必要时愿意批评趋势

### Structure Patterns
- **Strong opening thesis**：以清晰、往往大胆的陈述开头
- **Subheadings throughout**：大量使用 `###` 格式来拆分内容
- **Short paragraphs**：很少超过 3-4 句
- **Personal anecdotes woven in**：用真实示例说明观点
- **Practical takeaways**：提供可操作的见解，而非仅理论
- **Reflective conclusion**：以 call-to-action 或 forward-looking hope 结尾

### Length & Format
- 目标：800-1500 词
- Markdown 格式，包含 headers 和 emphasis
- 散文中尽量少用 bullet points——偏好流畅的句子

### Vocabulary Markers
- 对 tools/technology 使用 "leverage"
- 对 transitions 使用 "that said"
- 习惯直接陈述如 "this is useless" 或 "boy was I wrong"
- 自然地使用缩写（I've、doesn't、won't）
- 避免 corporate jargon，同时保持专业性

### Thematic Elements
- AI 是工具，不是替代
- 实用优于理论
- 以人为中心的技术
- 对什么有效、什么无效进行诚实评估

## Resources

### references/style-guide.md
作者写作模式、vocabulary preferences 和 structural conventions 的快速参考。

### references/blog-examples/
包含展示作者写作风格的示例博客文章。这些作为校准 voice 和 structure 时的参考材料。新的 finalized posts 会随时间扩展此库。

## Notion API Reference

要在 TS Notes 中创建 page：

```
Database data source ID: 04a872be-8bed-4f43-a448-3dfeebc0df21

Properties:
- "Note": (title) - 博客文章标题
- "Type": "Writing"
- "Project(s)": ["https://www.notion.so/2a5b4629bb3780189199f3c496980c0c"]

Content: Notion-flavored Markdown 格式的完整博客文章
```

"My Writing" project page ID 是：`2a5b4629-bb37-8018-9199-f3c496980c0c`
