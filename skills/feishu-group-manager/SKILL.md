---
name: feishu-group-manager
description: 管理飞书群聊设置，支持切换忙碌状态、更新群名/描述和权限配置
tags:
- 飞书
---

# Feishu Group Manager

管理飞书群聊（设置、名称、元数据）。

## 工具

### 切换忙碌状态
在群名中添加前缀（例如 `[⏳]`）以表示机器人正在处理长任务。

```bash
node skills/feishu-group-manager/toggle_busy.js --chat-id <chat_id> --mode <busy|idle>
```

### 更新设置
更新群名、描述（公告区域）和权限。

```bash
node skills/feishu-group-manager/update_settings.js --chat-id <chat_id> [options]
```

**选项：**
- `-n, --name <text>`: 新群名
- `-d, --description <text>`: 新群描述
- `--edit-permission <all_members|only_owner>`: 谁可以编辑群信息
- `--at-all-permission <all_members|only_owner>`: 谁可以 @All
- `--invite-permission <all_members|only_owner>`: 谁可以邀请其他人

## 使用协议
参见 `MEMORY.md` -> "Busy Status Protocol"。
- 触发器：一对一控制组中的长时运行任务（>30s）。
