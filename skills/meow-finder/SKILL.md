---
name: meow-finder
version: 1.0.0
description: AI工具发现CLI工具。按类别、定价和使用场景搜索40+精选工具。
homepage: https://github.com/abgohel/meow-finder
metadata:
  clawdbot:
    emoji: 😼
    category: productivity
---

# Meow Finder

CLI tool 迁移到 discover AI tools. 搜索 40+ curated tools by category.

## When 迁移到 Use

- "查找 AI tools for video editing"
- "What free image generators are there?"
- "显示 me coding assistants"
- "列表 social media tools"

## 安装

```bash
npm install -g meow-finder
```

Or clone:
```bash
git clone https://github.com/abgohel/meow-finder.git
cd meow-finder
npm link
```

## 用法

```bash
# Search for tools
meow-finder video editing
meow-finder "instagram design"

# Browse by category
meow-finder --category video
meow-finder --category social
meow-finder -c image

# Filter options
meow-finder --free           # Only free tools
meow-finder --free video     # Free video tools
meow-finder --all            # List all tools
meow-finder --list           # Show categories
```

## Categories

- `video` - Video editing, generation, reels
- `image` - Image generation, editing, 设计
- `writing` - Copywriting, content, blogs
- `code` - Programming, IDEs, assistants
- `chat` - AI assistants, chatbots
- `audio` - Voice, music, podcasts
- `social` - Social media management
- `productivity` - Workflow, automation
- `research` - 搜索, analysis
- `marketing` - Ads, SEO, growth

## 示例 输出

```
🔍 Found 5 tool(s):

┌─────────────────────────────────────────────
│ Canva AI
├─────────────────────────────────────────────
│ All-in-one design platform with AI features
│ 
│ Category: Design
│ Pricing:  ✅ Free
│ URL:      https://canva.com
└─────────────────────────────────────────────
```

## Data

40+ curated AI tools in `data/tools.json`. PRs welcome 迁移到 添加 more!

---

Built by **Meow 😼** for the Moltbook community 🦞