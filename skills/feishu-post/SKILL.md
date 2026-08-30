---
name: feishu-post
description: 向飞书发送富文本（Post）消息，支持原生表情、类 Markdown 解析和混合图片/链接的长文本
tags:
- 飞书
---

# Feishu Post (RichText) Skill

向飞书发送富文本（Post）消息。
此格式与卡片不同。它支持原生富文本元素，但在布局上不如卡片灵活。
它更适合混合图片/链接的长文本。

## 先决条件

- 先安装 `feishu-common`。
- 此 skill 通过 `utils/feishu-client.js` 依赖 `../feishu-common/index.js`。

## 功能
- **原生表情支持**：自动将 `[微笑]`、`[得意]` 等转换为飞书原生表情标签。
- **类 Markdown 解析**：支持简单的换行和段落。
- **富文本**：使用飞书的 Post 内容结构。

## 用法

```bash
node skills/feishu-post/send.js --target "ou_..." --text-file "temp/msg.md" --title "可选标题"
```

## 选项
- `-t, --target <id>`: 目标 ID（用户 `ou_...` 或聊天 `oc_...`）。
- `-x, --text <text>`: 文本内容（支持 `\n` 换行和 `[emoji]` 标签）。
- `-f, --text-file <path>`: 从文件读取内容。
- `--title <text>`: 帖子的标题。
- `--reply-to <id>`: 要回复的消息 ID。

## 表情列表
支持的表情包括：`[微笑]`、`[色]`、`[亲亲]`、`[大哭]`、`[强]`、`[加油]` 等。
完整映射请参见 `emoji-map.js`。
