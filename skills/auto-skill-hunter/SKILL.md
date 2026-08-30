---
name: auto-skill-hunter
description: 自动检索并安装ClawHub高价值Skill。当用户提出新问题、近期对话存在未解决问题，或需基于用户画像/个性特征主动扩展Skill覆盖范围时触发。支持从近期聊天和任务记忆中提取关键词，通过多维度评分筛选候选Skill并自动安装。
tags:
- meta
- evolution
- learning
- proactive
---

# Auto Skill Hunter

只专注于 **Skill**（不处理 Gene），支持自动检索与自动安装。

## 用法

```bash
node skills/skill-hunter/src/hunt.js
```

### 常用参数

```bash
# 1) 全自动巡逻（默认）
node skills/skill-hunter/src/hunt.js --auto

# 2) 指定一个“当前无法解决的问题”，触发定向找技能
node skills/skill-hunter/src/hunt.js --query "无法稳定抓取网页并总结关键信息"

# 3) 只看候选，不真正安装（调试推荐）
node skills/skill-hunter/src/hunt.js --dry-run
```

## 功能

1. 从近期聊天和任务记忆中抽取“待解决问题”与关键词。  
2. 在 ClawHub 上做趋势 + 关键词搜索。  
3. 用多维价值评分筛选候选 skill：  
   - 问题匹配度（是否能解决近期问题）  
   - 用户与 代理 画像匹配度（`USER.md` + personality）  
   - 互补性（与现有技能重复度低）  
   - 基础质量信号（stars/downloads 等）  
4. 自动安装高分技能，并确保生成可运行入口（`index.js` 自检）。  
5. 生成中文巡逻报告。  

## 安装 政策

- 默认每轮最多安装 2 个（可通过 `--max-install` 或环境变量覆盖）。  
- 已存在同名 skill 会跳过。  
- 若远程克隆失败，会回退到“可运行模板安装”，保证可执行。