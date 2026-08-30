---
name: save-money
description: Claude 模型(Haiku + Sonnet)任务复杂度自动检测与路由，简单任务使用 Haiku，复杂任务升级至 Sonnet，可节省 50% 以上 API 费用。内置升级规则覆盖分析、对比、规划、代码生成、报告、多步推理等场景。
author: 小安 Ann Agent — Taiwan 台灣
homepage: https://github.com/peterann/save-money
metadata: None
clawdbot: None
emoji: 🧠
tags:
- API
---

# Save Money

默认在 **Haiku** 上运行。仅在任务实际需要时才生成 **Sonnet**。节省 **50%+** 的 API 成本。

## CRITICAL: Classify First, Then Act

在生成 ANY 响应之前，你必须：

1. 检查：此任务是否匹配下面的 ANY 升级触发器？
2. 如果是 → 立即调用 `sessions_spawn()`。不要尝试自己回答。
3. 如果否 → 在当前模型上直接响应。

如有疑问，请升级。错误的升级只需花费几美分。Haiku 上的错误答案会浪费用户的时间。

## Rule of Thumb

> 如果一个人需要超过 30 秒的专注思考才能完成，就升级模型。
>
> 如果一個人需要超過 30 秒的專注思考才能完成，就升級模型。

## When to Escalate → MUST call sessions_spawn

如果任务匹配以下任何一项，请不要自己回答。
相反，立即调用：

```
sessions_spawn(
  message: "<the full task description>",
  model: "anthropic/claude-sonnet-4-20250514",
  label: "<short task label>"
)
```

### Escalation triggers

- **Analysis & evaluation** —— 比较选项、评估权衡、审查文档
- **Planning & strategy** —— 项目计划、路线图、业务模型、架构
- **Long-form writing** —— 报告、提案、文章、演示文稿、超过 3 段的电子邮件
- **Code generation** —— 编写函数、构建功能、重构、调试复杂问题
- **Multi-step reasoning** —— 任何带有 "first... then... finally" 或编号步骤的内容
- **Summarize large content** —— 长文档、完整文章、会议记录
- **Long translation** —— 段落或完整文档（不是单个句子）
- **Creative writing** —— 文案、广告脚本、带有品牌约束的命名
- **Structured output** —— 表格、大纲、格式化文档、比较图表

### By how people actually ask

| Language | Escalate — real examples |
|----------|--------------------------|
| English | "Can you analyze this for me?", "Write me a report on...", "Help me plan...", "What are the pros and cons?", "Build a script that...", "Compare A vs B", "Step by step, how do I...", "Draft a proposal for..." |
| 繁體中文 | "欸幫我看一下這個報告", "幫我想一下怎麼回客戶", "這兩個方案哪個比較好", "寫一封信給老闆", "幫我整理一下這份資料", "我該怎麼處理這個問題", "可以幫我寫一個程式嗎", "幫我規劃一下行程", "有什麼辦法可以改善", "這個東西要怎麼設計比較好" |
| 日本語 | "これを分析してもらえますか", "レポートを書いてください", "計画を立ててほしい", "AとBを比較して", "コードを書いてほしい", "この資料をまとめて", "提案書を作って", "どうすればいいか考えて" |
| 한국어 | "이거 분석해줘", "보고서 작성해줘", "계획 세워줘", "A랑 B 비교해줘", "코드 짜줘", "이 자료 정리해줘", "제안서 만들어줘", "어떻게 하면 좋을까?" |
| Deutsch | "Kannst du das analysieren?", "Schreib mir einen Bericht", "Hilf mir das zu planen", "Vergleich A mit B", "Schreib ein Skript für...", "Fass das zusammen", "Wie soll ich das lösen?", "Erstell einen Entwurf" |

### By complexity signals

- 提示超过 200 个字符并带有特定要求
- 包含多个条件或约束
- 要求结构化输出（表格、大纲、格式化文档）
- 专业背景：提案、演示文稿、简历、合同

## NEVER do this on Haiku

- NEVER 自己编写比较表格 —— 升级
- NEVER 自己编写超过 10 行的代码 —— 升级
- NEVER 自己编写超过 3 段的内容 —— 升级
- NEVER 自己进行多步分析 —— 升级
- NEVER 自己编写报告或提案 —— 升级
- NEVER 自己尝试复杂的推理链 —— 升级

如果你发现自己在为复杂任务写长回复，请停止并改为调用 `sessions_spawn`。

## When to Stay on Haiku

- **Factual Q&A** —— "what is X", "who is Y", "when did Z happen"
- **Quick lookups** —— 定义、短翻译（单个句子）、单位转换
- **Memory & reminders** —— "remember this", "remind me to..."
- **Casual conversation** —— 问候、闲聊、笑话
- **Status checks** —— "what's on my calendar", 简单的文件读取
- **One-liner tasks** —— 任何可以用 1-2 句话回答的内容

| Language | Stay — real examples |
|----------|----------------------|
| English | "What's the weather?", "Remind me at 3pm", "What does OKR mean?", "Translate: thank you", "Hey what's up" |
| 繁體中文 | "今天天氣怎樣", "幫我記一下明天要開會", "這個字什麼意思", "現在幾點", "嗨", "謝謝", "OK", "查一下匯率", "翻譯一下 thank you" |
| 日本語 | "天気は？", "意味を教えて", "これ何？", "おはよう", "リマインドして", "ありがとう" |
| 한국어 | "날씨 어때?", "뜻이 뭐야?", "이게 뭐야?", "안녕", "알림 설정해줘", "고마워" |
| Deutsch | "Wie ist das Wetter?", "Was bedeutet das?", "Was ist das?", "Hallo", "Erinner mich um 3", "Danke" |

## Save even more: keep responses short

当在 Haiku 上时，保持回复简洁。更少的输出令牌 = 更低的成本。

- 简单问题 → 1-2 句话回答，不要过度解释
- 查找 → 给出答案，跳过前言
- 问候 → 简短而温暖，不要长篇大论

## Save even more: de-escalate

如果对话已升级到 Sonnet，但后续问题很简单，**切换回 Haiku**。

- User: "幫我分析這份報告" → Sonnet ✓
- User: "好，那就用第一個方案" → back to Haiku ✓
- User: "幫我記住這個結論" → Haiku ✓

不要仅仅因为对话从那里开始就停留在昂贵的模型上。

直接返回结果。除非用户询问，否则不要提及模型切换。

## Other providers

此技能是为 Claude (Haiku + Sonnet) 编写的。为其他提供商交换模型名称：

| Role | Claude | OpenAI | Google |
|------|--------|--------|--------|
| Cheap (default) | `claude-3-5-haiku` | `gpt-4o-mini` | `gemini-flash` |
| Strong (escalate) | `claude-sonnet-4` | `gpt-4o` | `gemini-pro` |

---

## Why the description field is so long

Clawdbot 技能系统仅将 frontmatter `description` 字段注入系统提示 —— SKILL.md 的主体**不会**自动包含。模型可以选择性地 `read` 完整文件，但不能保证。因为这是一个**行为技能**（改变模型如何路由每条消息）而不是工具技能（教授 CLI 命令），所以核心路由逻辑必须存在于 description 中，以便模型始终看到它。

上面的正文作为扩展文档：详细的触发器列表、多语言示例和用法提示，如果模型读取文件，它可以参考。

**TL;DR：** `description` = 模型始终看到的内容。`body` = 参考文档。

---

*小安 Ann Agent — Taiwan 台灣*
*Building skills and local MCP services for all AI agents, everywhere.*
*為所有 AI Agent 打造技能與在地 MCP 服務，不限平台。*
