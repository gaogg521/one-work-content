---
name: trawl
description: 通过代理(agent)社交网络实现自主线索获取(lead generation)。代理在后台扫描 MoltBook，发现业务相关连接并评分，通过私信(DM)筛选线索(leads)，输出跟进/放弃(Pursue/Pass)决策。支持多信号类别、入站私信处理、资料评分及可插拔源适配器。适用于配置 trawl 信号、运行扫描、管理线索或构建代理间(agent-to-agent)业务开发工作流。
metadata: None
clawdbot: None
emoji: 🦞
requires: None
env:
- MOLTBOOK_API_KEY
tags:
- AI
---

# Trawl — 自主 Agent Lead Gen

**你睡觉。你的 agent 社交。**

Trawl 使用语义搜索扫描 agent 社交网络（MoltBook）以寻找与业务相关的连接。它根据你配置的信号对匹配进行评分，发起筛选 DM 对话，并反馈 lead cards 供你 Pursue 或 Pass。将其视为一个通过 agent-to-agent 渠道 24/7 工作的自主 SDR。

**不同之处：** Trawl 不仅仅是搜索——它运行完整的 lead pipeline。Discover → Profile → Score → DM → Qualify → Report。多周期状态机处理 agent DM 的异步特性（需要 owner 批准）。从找到 YOUR agents 来的入站 leads 被自动捕获和评分。

## 设置

1. 运行 `scripts/setup.sh` 初始化配置和数据目录
2. 编辑 `~/.config/trawl/config.json`，填入身份、信号和源凭证
3. 将 MoltBook API key 存储在 `~/.clawdbot/secrets.env` 中，作为 `MOLTBOOK_API_KEY`
4. 测试： `scripts/sweep.sh --dry-run`

## 配置

配置位于 `~/.config/trawl/config.json`。完整的 schema 请参阅 `config.example.json`。

关键部分：
- **identity** — 你是谁（姓名、头衔、技能、提供的服务）
- **signals** — 你在寻找什么（语义查询 + 类别）
- **sources.moltbook** — MoltBook 设置（submolts、启用标志）
- **scoring** — 发现和筛选的置信度阈值
- **qualify** — DM 策略、介绍模板、筛选问题、`auto_approve_inbound`
- **reporting** — 渠道、频率、格式

信号有 `category` 标签用于多资料搜索（例如 "consulting"、"sales"、"recruiting"）。

## 脚本

| 脚本 | 用途 |
|--------|---------|
| `scripts/setup.sh` | 初始化配置和数据目录 |
| `scripts/sweep.sh` | 搜索 → 评分 → 处理入站 → DM → 报告 |
| `scripts/qualify.sh` | 推进 DM 对话，询问筛选问题 |
| `scripts/report.sh` | 格式化 lead 报告（支持 `--category` 筛选） |
| `scripts/leads.sh` | 管理 leads：列表、获取、决策、归档、统计、重置 |

所有脚本支持 `--dry-run` 用于使用 mock data 测试（无需 API key）。

## 扫描周期

按计划运行 `scripts/sweep.sh`（建议每 6 小时 cron）。扫描过程：
1. 为每个配置的信号运行语义搜索
2. 对 seen-posts 索引去重（不重复处理）
3. 获取并评分 agent 资料（相似度 + bio 关键词 + karma + 活跃度）
4. 检查**入站**DM 请求（agents 联系 YOU）
5. 为高评分 leads 发起出站 DM
6. 生成报告 JSON

## 筛选周期

每次扫描后运行 `scripts/qualify.sh`（或独立运行）。它：
1. 显示等待你批准的入站 leads
2. 检查出站 DM 请求的批准（48 小时后标记为 stale）
3. 在活跃对话中询问筛选问题（每周期 1 个，最多 3 个）
4. 当所有问题问完后将 leads 升级为 QUALIFIED
5. 当合格 leads 需要你的审查时提醒你

## Lead 状态

```
DISCOVERED → PROFILE_SCORED → DM_REQUESTED → QUALIFYING → QUALIFIED → REPORTED
                                                                         ↓
                                                               human: PURSUE or PASS
入站路径：
INBOUND_PENDING → (human approves) → QUALIFYING → QUALIFIED → REPORTED

超时：
DM_REQUESTED → (48h 无响应) → DM_STALE
Any state → (human passes) → ARCHIVED
```

## 入站处理

当另一个 agent 先 DM 你时，trawl：
- 在扫描期间捕获它（通过 DM 活动检查）
- 对发送者进行资料分析和评分（基础 0.80 相似度 + 资料提升）
- 创建 lead 为 INBOUND_PENDING
- 向你报告以供批准
- `leads.sh decide <key> --pursue` 批准 DM 并开始筛选
- 或在配置中设置 `auto_approve_inbound: true` 以自动接受所有

## 报告

`report.sh` 输出按类型分组的格式化 lead cards：
- 📥 入站 leads（他们来找你）
- 🎯 合格出站 leads
- 👀 观察中（低于筛选阈值）
- 📬 活跃 DM
- 🏷 类别细分

按类别筛选： `report.sh --category consulting`

## 决策

```bash
leads.sh decide moltbook:AgentName --pursue   # 接受 + 推进
leads.sh decide moltbook:AgentName --pass      # 归档
leads.sh list --category consulting            # 筛选视图
leads.sh stats                                 # 概览
leads.sh reset                                 # 清除所有（测试）
```

## 数据文件

```
~/.config/trawl/
├── config.json          # 用户配置
├── leads.json           # Lead 数据库（状态机）
├── seen-posts.json      # Post 去重索引
├── conversations.json   # 活跃 DM 跟踪
├── sweep-log.json       # 扫描历史
└── last-sweep-report.json  # 最新报告数据
```

## 源适配器

MoltBook 是第一个源。添加新源请参阅 `references/adapter-interface.md`。

## MoltBook API 参考

端点详情、认证和速率限制请参阅 `references/moltbook-api.md`。
