---
name: minimax-mcp
description: 用于网页搜索(web search)和图像理解(image understanding)的 MiniMax MCP 服务器(server)。支持通过 MiniMax API 搜索、分析图像和从 URL 提取内容。触发词：MiniMax、网页搜索(web search)、图像分析(image analysis)、MCP 服务器(server)
metadata: None
openclaw: None
emoji: 🔍
install:
- default: Mainland China (minimaxi.com)
  id: region
  kind: select
  label: 选择区域
  options:
  - Global (minimax.io)
  - Mainland China (minimaxi.com)
- description: 'Global: https://www.minimax.io/platform/user-center/basic-information/interface-key
    | China: https://platform.minimaxi.com/user-center/basic-information/interface-key'
  envVar: MINIMAX_API_KEY
  id: api_key
  kind: input
  label: MiniMax API Key
  secret: true
- bins:
  - uvx
  formula: uv
  id: uv
  kind: brew
  label: 安装 uv (MCP server 所需)
primaryEnv: MINIMAX_API_KEY
requires: None
bins:
- uvx
env:
- MINIMAX_API_KEY
- MINIMAX_API_HOST
tags:
- API
- Web
- 搜索
---

# MiniMax MCP Skill

## 概述

Official MiniMax Model Context Protocol (MCP) server for coding-plan users, providing AI-powered search and vision analysis capabilities.

## 功能

| 工具 | 功能 | 支持的格式 |
|------|----------|-------------------|
| **web_search** | 网页搜索，返回结构化结果（标题、链接、摘要） | - |
| **understand_image** | AI 图像分析和内容识别 | JPEG, PNG, WebP |

## 触发场景

当用户说以下话时使用此 skill：
- "Search for xxx" / "Look up xxx"
- "Look at this image" / "Analyze this picture"
- "What's in this image" / "Describe this photo"
- "Extract content from URL" / "Fetch this webpage"

## 快速开始

### 1. 获取 API Key

| 区域 | API Key URL | API Host |
|--------|-------------|----------|
| 🇨🇳 China | platform.minimaxi.com | https://api.minimaxi.com |
| 🇺🇳 Global | minimax.io | https://api.minimax.io |

### 2. 配置 mcporter（推荐）

```bash
# 添加 MCP server
mcporter config add minimax \
  --command "uvx minimax-coding-plan-mcp -y" \
  --env MINIMAX_API_KEY="your-key" \
  --env MINIMAX_API_HOST="https://api.minimaxi.com"

# 测试连接
mcporter list
```

### 3. 直接使用

```bash
# 搜索
mcporter call minimax.web_search query="keywords"

# 分析图像
mcporter call minimax.understand_image prompt="Describe this image" image_source="image-url-or-path"
```

## 使用示例

参见 [references/examples.md](references/examples.md)

## 环境变量

| 变量 | 必填 | 描述 |
|----------|----------|-------------|
| `MINIMAX_API_KEY` | ✅ | 你的 MiniMax API Key |
| `MINIMAX_API_HOST` | ✅ | API endpoint |

## 重要提示

⚠️ **API Key 必须与 host 区域匹配！**

| 区域 | API Key 来源 | API Host |
|--------|---------------|----------|
| Global | minimax.io | https://api.minimax.io |
| China | minimaxi.com | https://api.minimaxi.com |

如果你收到 "Invalid API key" 错误，请检查你的 Key 和 Host 是否来自同一区域。

## 故障排除

- **"uvx not found"**：安装 uv - `brew install uv` 或 `curl -LsSf https://astral.sh/uv/install.sh | sh`
- **"Invalid API key"**：确认 API Key 和 Host 来自同一区域
- **Image download failed**：确保图像 URL 可公开访问，支持 JPEG/PNG/WebP

## 相关资源

- GitHub: https://github.com/MiniMax-AI/MiniMax-Coding-Plan-MCP
- MiniMax Platform: https://platform.minimaxi.com (China) / https://www.minimax.io (Global)
