---
name: feishu-search
description: 飞书（Lark）资源的统一搜索接口，支持搜索消息、文档和日历
tags:
- 搜索
- 飞书
---

# Feishu Search Skill

飞书（Lark）资源的统一搜索接口。

## 先决条件

- 先安装 `feishu-common`。
- 此 skill 依赖 `../feishu-common/index.js` 获取令牌和 API 认证。

## 功能

- **搜索消息**：在私聊和群聊中查找聊天记录。
- **搜索文档**：定位文档、表格和 bitable。
- **搜索日历**：（计划中）查找事件。

## 用法

```bash
# 搜索消息
node skills/feishu-search/index.js search_messages --query "bug report" --limit 10

# 搜索文档
node skills/feishu-search/index.js search_docs --query "Q3 Roadmap"
```

## 配置

需要标准飞书环境变量（`FEISHU_APP_ID`、`FEISHU_APP_SECRET`）或有效的 `FEISHU_TOKEN`。
此 skill 使用来自 `../feishu-common/index.js` 的共享认证。
