---
name: glin-profanity-mcp
description: 为 AI 助手提供脏话检测(profanity detection)的 MCP 服务器。支持批量内容审查(content moderation)、审核报告审计(audit)和发布前文本分析。触发词：内容审查(content moderation)、脏话检测(profanity detection)、批量审核(batch audit)、文本分析(text analysis)
tags:
- AI
- 安全
---

# Glin Profanity MCP Server

为 Claude Desktop、Cursor 和 Windsurf 等 AI 助手提供脏话检测作为工具的 MCP (Model Context Protocol) 服务器。

**最适合:** AI 辅助内容审查工作流、批量审核、审计报告和发布前的内容验证。

## 安装

### Claude Desktop

添加到 `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "glin-profanity": {
      "command": "npx",
      "args": ["-y", "glin-profanity-mcp"]
    }
  }
}
```

### Cursor

添加到 `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "glin-profanity": {
      "command": "npx",
      "args": ["-y", "glin-profanity-mcp"]
    }
  }
}
```

## 可用工具

### 核心检测

| 工具 | 描述 |
|------|-------------|
| `check_profanity` | 检查文本中的脏话并返回详细结果 |
| `censor_text` | 用可配置的替换项审查脏话 |
| `batch_check` | 一次检查多个文本 (最多 100) |
| `validate_content` | 获取安全分数 (0-100) 并附带操作建议 |

### 分析

| 工具 | 描述 |
|------|-------------|
| `analyze_context` | 上下文感知分析 (医疗、游戏等) |
| `detect_obfuscation` | 检测 leetspeak 和 Unicode 技巧 |
| `explain_match` | 解释为什么文本被标记 |
| `compare_strictness` | 跨严格级别比较检测 |

### 实用工具

| 工具 | 描述 |
|------|-------------|
| `suggest_alternatives` | 建议干净的替代词 |
| `analyze_corpus` | 分析最多 500 个文本以获取统计信息 |
| `create_regex_pattern` | 生成用于自定义检测的正则表达式 |
| `get_supported_languages` | 列出所有 24 种支持的语言 |

### 用户跟踪

| 工具 | 描述 |
|------|-------------|
| `track_user_message` | 跟踪消息以识别重复违规者 |
| `get_user_profile` | 获取用户的审核历史 |
| `get_high_risk_users` | 列出违规率高的用户 |

## 示例提示

### 内容审查
```
"Check these 50 user comments and tell me which ones need moderation"
"Validate this blog post before publishing - use high strictness"
"Analyze this medical article with medical domain context"
```

### 批量操作
```
"Batch check all messages in this array and return only flagged ones"
"Generate a moderation audit report for these comments"
```

### 理解标记
```
"Explain why 'f4ck' was detected as profanity"
"Compare strictness levels for this gaming chat message"
```

### 内容清理
```
"Suggest professional alternatives for this flagged text"
"Censor the profanity but preserve first letters"
```

## 何时使用

**使用 MCP 服务器时:**
- AI 协助内容审查工作流
- 批量检查用户提交
- 生成审核报告
- 发布前内容验证
- 人类在环审核

**改用核心库时:**
- 自动化实时过滤 (hooks/中间件)
- 每条消息都需要检查而无需 AI 参与
- 性能关键应用 (< 1ms 响应)

## 资源

- npm: https://www.npmjs.com/package/glin-profanity-mcp
- GitHub: https://github.com/GLINCKER/glin-profanity/tree/release/packages/mcp
- 核心库: https://www.npmjs.com/package/glin-profanity
