---
name: feishu-message
description: 飞书消息操作的统一工具包，支持获取消息、发送音频、创建群聊和列出置顶消息
tags:
- 飞书
---

# Feishu Message Skill

飞书消息操作的统一工具包，为常见任务提供单一 CLI 入口。

## 用法

通过 `index.js` 使用统一 CLI：
```bash
node skills/feishu-message/index.js <command> [options]
```

## 命令

### 1. 获取消息 (`get`)
通过 ID 获取消息内容。支持合并消息的递归获取。
```bash
node skills/feishu-message/index.js get <message_id> [--raw] [--recursive]
```
示例：
```bash
node skills/feishu-message/index.js get om_12345 --recursive
```

### 2. 发送音频 (`send-audio`)
将音频文件作为语音气泡发送。
```bash
node skills/feishu-message/index.js send-audio --target <id> --file <path> [--duration <ms>]
```
- `--target`: User OpenID (`ou_`) 或 ChatID (`oc_`)。
- `--file`: 音频文件路径 (mp3/wav 等)。
- `--duration`: （可选）时长，单位为毫秒。

### 3. 创建群聊 (`create-chat`)
使用指定用户创建新群聊。
```bash
node skills/feishu-message/index.js create-chat --name "Project Alpha" --users "ou_1" "ou_2" --desc "Description"
```

### 4. 列出置顶 (`list-pins`)
列出聊天中的置顶消息。
```bash
node skills/feishu-message/index.js list-pins <chat_id>
```

## 旧版脚本
独立脚本仍然可用以保持向后兼容：
- `get.js`
- `send-audio.js`
- `create_chat.js`
- `list_pins_v2.js`

## 依赖
- axios
- form-data
- music-metadata
- commander
