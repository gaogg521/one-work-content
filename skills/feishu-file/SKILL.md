---
name: feishu-file
description: 通过飞书 API 管理文件上传和下载，支持发送文件、仅上传获取 file_key 和下载消息中的文件
tags:
- API
- Key
- 飞书
---

# Feishu File Skill

通过飞书 API 管理文件上传和下载。

## 前置条件

- 先安装 `feishu-common`。
- 此技能依赖 `../feishu-common/index.js` 获取 token 和 API 认证。

## 命令

### 发送文件
上传本地文件并发送到聊天或用户。
```bash
node skills/feishu-file/send.js --target <chat_id_or_user_id> --file <local_path>
```

### 仅上传
上传文件并获取其 `file_key`（用于卡片或富文本）。
```bash
node skills/feishu-file/upload.js --file <local_path>
```

### 下载文件
从消息中下载文件资源。
```bash
node skills/feishu-file/download.js --message-id <msg_id> --file-key <file_key> --output <local_path>
```
**注意：** Bot 必须有权访问该消息（在聊天中）。对于他人发送的文件，需要 `im:resource:read` 权限。
