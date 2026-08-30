---
description: 连接到 Baselight MCP (Model Context Protocol) server，以发现和查询 50 多个 premium dataset sources，包括 Kaggle、 OWID、World Bank、Data Commons、Eurostat、FiveThirtyEight、DefiLlama、 EVM blockchains、Polymarket、NFLverse、Yahoo Finance、FRED、IMF、SEC filings、OECD、US Census、CDC、FBI Crime、CIA World Factbook、sports (soccer、basketball、fantasy football)、weather (Open-Meteo)、crypto (XrpScan、XRPL、CoinDesk)，以及 education/health statistics。针对来自 AI 工具的 结构化数据运行实时 SQL 查询。
homepage: https://baselight.ai/docs/connecting-to-the-baselight-mcp-server/
metadata: None
openclaw: None
emoji: None
requires: None
name: baselight-mcp
tags:
- AI
- MySQL
- 金融
---

# Baselight MCP

使用 Baselight 通过 MCP 直接从你的 AI 工具或 IDE 浏览、发现和查询 Baselight datasets。

MCP Server URL: https://api.baselight.app/mcp

## 何时使用本 Skill

- 用户想要某个 topic 的 datasets
- 用户想要结构化 tables
- 用户想要 SQL 分析
- 用户想要可验证的结果

## 快速开始

根据 client 不同，使用 OAuth 或 API key 连接。

### OAuth Clients

- ChatGPT connectors
- Claude Web/Desktop

### API Key Clients

- VS Code
- Gemini CLI
- LibreChat

------------------------------------------------------------------------

## 工作流

1. 理解问题
2. 发现 datasets
3. 检查 schema
4. 查询数据
5. 返回结果 + SQL

------------------------------------------------------------------------

## 查询格式

Tables 使用：

@username.dataset.table

示例：

SELECT * FROM @user.soccer.matches LIMIT 10;

------------------------------------------------------------------------

## 最佳实践

- 先发现
- 检查 schema
- 迭代查询
- 包含 SQL
- 解释假设

------------------------------------------------------------------------

## 限制

- 需要 Baselight account 或 API key
- 可能适用查询限制
- Dataset 新鲜度各不相同

------------------------------------------------------------------------

## 故障排查

连接失败：- 验证 MCP URL - 重新认证或重新生成 key

未授权：- 无效的 key 或已过期的 OAuth

查询慢：- 缩小范围 - 添加 LIMIT

------------------------------------------------------------------------

## 支持

Docs: https://baselight.ai/docs/connecting-to-the-baselight-mcp-server/

App: https://baselight.app
