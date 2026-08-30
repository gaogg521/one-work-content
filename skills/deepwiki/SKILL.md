---
name: deepwiki
description: 查询 DeepWiki MCP server 以获取 GitHub 仓库文档、wiki 结构和 AI 驱动的问题解答。
homepage: https://docs.devin.ai/work-with-devin/deepwiki-mcp
tags:
- AI
---

# DeepWiki

使用本技能通过 DeepWiki MCP server 访问公共 GitHub 仓库的文档。你可以搜索仓库 wiki、获取结构，并提出基于仓库文档的复杂问题。

## 命令

### Ask Question
提出关于 GitHub 仓库的任何问题，并获得 AI 驱动的、基于上下文的回答。
```bash
node ./scripts/deepwiki.js ask <owner/repo> "your question"
```

### Read Wiki Structure
获取 GitHub 仓库的文档主题列表。
```bash
node ./scripts/deepwiki.js structure <owner/repo>
```

### Read Wiki Contents
查看 GitHub 仓库 wiki 中特定路径的文档。
```bash
node ./scripts/deepwiki.js contents <owner/repo> <path>
```

## 示例

**询问 Devin 的 MCP 用法：**
```bash
node ./scripts/deepwiki.js ask cognitionlabs/devin "How do I use MCP?"
```

**获取 React 文档的结构：**
```bash
node ./scripts/deepwiki.js structure facebook/react
```

## 注意事项
- Base Server: `https://mcp.deepwiki.com/mcp`
- 仅适用于公共仓库。
- 无需认证。
