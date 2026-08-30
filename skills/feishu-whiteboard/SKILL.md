---
name: feishu-whiteboard
description: 以编程方式创建和操作飞书白板，支持创建白板、添加节点和图形连接
tags:
- 飞书
---

# Feishu Whiteboard Skill

允许以编程方式创建和操作飞书白板。

## 配置
需要在环境或 `config.json` 中设置 `FEISHU_APP_ID` 和 `FEISHU_APP_SECRET`。
所需权限：`board:whiteboard:node:create`

## 用法

### 创建白板
```bash
node skills/feishu-whiteboard/create.js "My Architecture Diagram"
```
输出：包含 `whiteboard_id` 的 JSON。

### 添加节点（演示）
```bash
node skills/feishu-whiteboard/draw.js <whiteboard_id> demo
```
添加一个矩形和一个圆形，并用一条线连接。

### 编程用法

```javascript
const { createWhiteboard } = require('./create');
const { addNodes, createShape, createConnector } = require('./draw');

const board = await createWhiteboard("System Design");
const nodes = [
  createShape("web", "rect", 0, 0, 200, 100, "Web Server"),
  createShape("db", "cylinder", 0, 300, 100, 100, "Database"),
  createConnector("link1", "web", "db")
];
await addNodes(board.whiteboard_id, nodes);
```

## 故障排除
如果遇到 `404 page not found`，通常意味着您的租户未启用白板 API，或者端点 URL 已更改。当前实现使用 `/open-apis/board/v1/whiteboards`。
