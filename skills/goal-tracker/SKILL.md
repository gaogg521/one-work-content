---
name: goal-tracker
description: 目标追踪(goal tracking)技能。使用里程碑(milestone)、每日日志(daily log)和问责制(accountability)追踪长期目标，支持生成 HTML 仪表板(dashboard)。触发词：目标追踪(goal tracking)、里程碑(milestone)、每日日志(daily log)、仪表板(dashboard)
tags:
- Grafana
---

# 目标追踪技能

使用里程碑、每日日志和问责制追踪长期目标。

## 位置
`/root/clawd/goal-tracker/`

## 命令

```bash
# 显示状态
/root/clawd/tracker status

# 记录每日活动
/root/clawd/tracker log --trained --business --wins "Description"

# 更新 MRR
/root/clawd/tracker mrr 5000

# 标记里程碑完成
/root/clawd/tracker milestone ironman "5km run"
/root/clawd/tracker milestone mrr_100k "first client"

# 每周总结
/root/clawd/tracker week

# 生成 HTML 仪表板
/root/clawd/goal-tracker/generate-dashboard
```

## 数据文件
- `data/goals.json` - 目标定义和里程碑
- `data/daily-log.json` - 每日签到
- `index.html` - 生成的可视化仪表板

## 与 Alto 集成

### 晚间签到期间
问: "Did you train today? Work on business?"
然后记录: `tracker log --trained --business` (或 `--no-trained` 等)

### 当胜利发生时
记录它们: `tracker log --wins "Landed new client"`

### 当 MRR 变化时
更新: `tracker mrr 5000`

### 当里程碑完成时
标记: `tracker milestone ironman "5km"`

## 追踪的目标

### 铁人三项 (by 2030)
里程碑:
1. Run 5km without stopping
2. Complete sprint triathlon
3. Complete Olympic distance
4. Complete Half Ironman (70.3)
5. Complete Full Ironman

### $100k MRR (by 2030)
里程碑:
1. First paying client
2. $5k MRR
3. $10k MRR
4. $25k MRR
5. $50k MRR
6. $100k MRR

## 每周节奏
- Monday: Week planning (set priorities)
- Daily: Evening check-in (log training + business)
- Friday: Week review (score + adjust)

## 需要注意的模式
- 3+ days without training = gentle nudge
- Week score < 50% = check in on blockers
- Milestone dates approaching = increase urgency
