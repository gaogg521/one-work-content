---
name: feishu-vc
description: 管理飞书视频会议（VC），支持预约会议和更新会议预约信息
tags:
- 飞书
---

# Feishu Video Conference Skill

管理飞书视频会议（VC）。

## 用法

### 预约会议
创建会议预约。
```bash
node skills/feishu-vc/reserve.js --subject "会议标题" --time "2026-02-04T10:00:00+08:00"
```

## API 参考
- Reserve: `POST /open-apis/vc/v1/reserve`
- 所需权限：`vc:meeting:request`（更新会议预约信息）

## 设置
需要 `FEISHU_APP_ID` 和 `FEISHU_APP_SECRET`。
