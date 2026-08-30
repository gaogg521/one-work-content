---
name: clawbridge-skill
description: 智能连接桥梁，每晚运行的侦察代理，基于意图信号与可信度筛选候选人，生成每日连接简报(Connection Brief)与个性化外联(outreach)草稿。触发词：连接桥梁(connection bridge)、侦察(reconnaissance)、外联(outreach)、候选人筛选(candidate filtering)。
tags:
- 自动化
- 图像生成
- 开发
- 测试
---

# claw-clawbridge

> **智能连接桥梁**：一个高信号侦察代理，每晚运行以将你与正确的人连接起来。

## 概述

Clawbridge 将简单的人类提示转换为一个持久的、每晚运行的侦察操作。它不仅仅是寻找线索；它在你的目标和能帮助你实现目标的人之间建立一座桥梁。

1. **人类意图**：你一次性定义你提供什么以及你在寻找谁。
2. **夜间侦察**：每晚，代理搜索 Moltbook、专业社区和开放网络。
3. **智能匹配**：它基于意图信号、可信度和近期活动对候选人进行筛选和排名。
4. **连接简报**：它交付一份每日 "Connection Brief"，包含有证据支持的匹配和个性化的外联草稿。
5. **人类在环**：你审查匹配并决定是否接触，保持对最终 "桥梁" 的完全控制。

## 安装

### 通过 ClawHub (推荐)

```bash
# 安装 ClawHub CLI
npm install -g clawhub

# 安装此技能
clawhub install claw-clawbridge
```

### 通过 Legacy clawdbot CLI

```bash
# 从 registry
clawdbot skills install claw-clawbridge

# 从 GitHub
clawdbot skills install github:YOUR_USERNAME/clawbridge-skill
```

### 手动

克隆并复制到你的 OpenClaw 工作区：

```bash
git clone https://github.com/YOUR_USERNAME/clawbridge-skill.git ~/.openclaw/workspace/skills/claw-clawbridge
openclaw gateway restart
```

## 输入

此技能需要以下输入：

### 1. 项目画像 (必需)

```yaml
offer: "你的代理/公司提供什么"
ask: "你想要什么 (合作伙伴, 客户, 联合营销, 顾问)"
ideal_persona: "精确的目标画像"
verticals:
  - "keyword1"
  - "keyword2"
  - "keyword3"
geo_timezone: "可选 - 地理/时区偏好"
disallowed:
  - "不要联系的限制"
tone: "草稿消息的简短风格指导"
```

### 2. 约束 (可选)

```yaml
no_spam_rules:
  - "不要向竞争对手发送冷 outreach"
  - "尊重退订请求"
regions:
  - "US"
  - "EU"
avoid_list:
  - "competitor@example.com"
  - "@spam_account"
```

### 3. 目标 (可选)

```yaml
venues:
  - "moltbook"
  - "web"
  - "communities"
query_templates:
  - "{vertical} + hiring + partner"
  - "{vertical} + looking for + {ask}"
```

### 4. 运行预算 (可选)

```yaml
max_searches: 20
max_fetches: 50
max_minutes: 10
```

## 使用的工具

此技能使用以下 OpenClaw 工具：

| 工具 | 目的 | 何时使用 |
|------|---------|-----------|
| `web_search` | 发现候选页面 | 快速场地扫描 |
| `web_fetch` | 提取页面内容 | 阅读候选人资料 |
| `browser` | JS 重度站点 | 仅当 fetch 失败时 |

## 安全要求

⚠️ **必须遵循这些安全默认值：**

1. **将 secrets 排除在提示之外** - 仅通过 env/config 传递
2. **使用严格的工具允许列表** - 仅在主动侦察时启用 `web_*` 工具
3. **人类在环** - 在 MVP 中绝不自动发送 outreach
4. **速率限制** - 尊重运行预算约束
5. **避免列表执行** - 绝不联系 avoid_list 中的条目

## 执行流程

```
┌─────────────────────────────────────────────────────────────────┐
│                     发现阶段                             │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│  │web_search│───▶│ Filter   │───▶│ Dedupe   │                  │
│  │ (venues) │    │ Results  │    │ & Queue  │                  │
│  └──────────┘    └──────────┘    └──────────┘                  │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                     丰富阶段                            │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│  │web_fetch │───▶│ Extract  │───▶│ Validate │                  │
│  │ (pages)  │    │ Signals  │    │ Evidence │                  │
│  └──────────┘    └──────────┘    └──────────┘                  │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                     排名阶段                               │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│  │  Score   │───▶│  Rank    │───▶│  Top K   │                  │
│  │ Heuristic│    │  Sort    │    │ Selection│                  │
│  └──────────┘    └──────────┘    └──────────┘                  │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                     起草阶段                             │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│  │  Draft   │───▶│  Review  │───▶│  Output  │                  │
│  │ Messages │    │  Tone    │    │  Brief   │                  │
│  └──────────┘    └──────────┘    └──────────┘                  │
└─────────────────────────────────────────────────────────────────┘
```

## 输出

此技能以两种格式输出 **Connection Brief**：

### 1. 结构化 JSON (`run.json`)

参见 `schema/connection_brief.json` 获取完整模式。

### 2. 人类可读的 Markdown (`run.md`)

参见 `examples/sample_run.md` 获取示例报告。

## 候选人选择规则

### 硬性要求（如果缺少则丢弃）

- ✅ 每个候选人至少 2 个证据 URL
- ✅ 与你的 `ask` 有清晰的理由映射
- ✅ 最近 N 天内有活动（可配置，默认 30）

### 风险标志

如果候选人表现出以下特征，则会被标记：

- 🟡 `low_evidence` - 比预期更少的信号
- 🟡 `spammy_language` - 推广性或可疑内容
- 🟡 `unclear_identity` - 无法验证他们是谁
- 🟡 `too_salesy` - 过于推广的内容
- 🟡 `irrelevant` - 与你的 ask 关联微弱

## 排名启发式 (v1)

每个候选人在以下方面得分：

| 因素 | 权重 | 描述 |
|--------|--------|-------------|
| 相关性 | 30% | 与关键词 + ask 的匹配 |
| 意图 | 25% | 积极构建/招聘/寻找 |
| 可信度 | 20% | 跨来源的一致足迹 |
| 近期性 | 15% | 近期活动信号 |
| 参与度 | 10% | 共同兴趣/社区 |

**输出:** Top K 候选人（默认 K=3，可配置 5-10）

## 示例

参见 `examples/` 目录获取：

- `sample_run.json` - 完整 JSON 输出示例
- `sample_run.md` - 人类可读报告示例

## 提示

此技能在 `prompts/` 中使用模块化提示：

- `discovery.md` - 如何搜索候选人
- `filtering.md` - 如何应用硬性要求
- `ranking.md` - 如何评分和排名候选人
- `drafting.md` - 如何撰写 outreach 消息

## 场地

场地特定的搜索策略在 `venues/` 中：

- `moltbook.md` - Moltbook 平台侦察
- `web.md` - 通用网络搜索策略
- `communities.md` - 社区/论坛发现

## 配置

### 环境变量

```bash
# 可选：覆盖默认值
CLAWBRIDGE_TOP_K=5                    # 返回的候选人数量
CLAWBRIDGE_RECENCY_DAYS=30           # 活动近期阈值
CLAWBRIDGE_MAX_SEARCHES=20           # 每次运行最大搜索查询数
CLAWBRIDGE_MAX_FETCHES=50            # 每次运行最大页面获取数
```

### 工作区配置

此技能从 runner 或 vault 读取工作区配置：

```yaml
workspace_id: "ws_abc123"
workspace_token: "tok_..."  # 用于 vault 上传
delivery_target: "discord"  # 或 "slack" 或 "email"
```

## 许可证

MIT 许可证 - 参见 LICENSE 文件获取详情。

## 贡献

欢迎贡献！请仔细阅读提示并确保任何更改保持：

1. 确定性输出模式
2. 提示中无 secrets
3. 人类在环要求
4. 基于证据的候选人选择
