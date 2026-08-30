---
name: feishu-calendar
description: 管理飞书（Lark）日历，支持列出日历、检查日程、同步事件和创建共享日历
tags:
- 飞书
---

# feishu-calendar

管理飞书 (Lark) 日历。使用此技能列出日历、检查日程和同步事件。

## 用法

### 列出日历
检查可用的日历及其 ID。
```bash
node skills/feishu-calendar/list_test.js
```

### 搜索日历
按名称/摘要查找日历。
```bash
node skills/feishu-calendar/search_cal.js
```

### 检查主日历
专门检查主日历的状态。
```bash
node skills/feishu-calendar/check_master.js
```

### 同步例行程序
运行日历同步例行程序（将事件同步到本地状态/内存）。
```bash
node skills/feishu-calendar/sync_routine.js
```

## 设置
需要 `.env` 中的 `FEISHU_APP_ID` 和 `FEISHU_APP_SECRET`。

## 标准协议：任务标记
**触发器**：用户说 "Mark this task" 或 "Remind me to..."。
**操作**：
1. **分析**：提取日期/时间（例如，"Feb 4th" -> YYYY-MM-04）。
2. **执行**：运行 `create.js`，`--attendees` 设置为请求者的 ID。
3. **格式**：
   ```bash
   node skills/feishu-calendar/create.js --summary "Task: <Title>" --desc "<Context>" --start "<ISO>" --end "<ISO+1h>" --attendees "<User_ID>"
   ```

### 设置共享日历
为项目创建共享日历并添加成员。
```bash
node skills/feishu-calendar/setup_shared.js --name "Project Name" --desc "Description" --members "ou_1,ou_2" --role "writer"
```
