---
name: discord-soul
description: 从你的 Discord 服务器创建一个鲜活的代理。该代理体现你社区的身份，记住每一次对话，并随着社区的演变而成长。像与人交谈一样与你的 Discord 交谈。
tags:
- 社交媒体
---

# Discord Soul

将你的 Discord 服务器变成一个鲜活的、有呼吸的代理。

## 你将获得什么

一个代理，它可以：
- **记住** 你在 Discord 中的每一次对话
- **说话** 使用你社区的声音
- **了解** 关键人物、频道和内部笑话
- **成长** 随着每天新消息的到来
- **回答** 关于你社区历史和文化的问题

## 快速开始

```bash
# 从你的 Discord 创建代理
./scripts/create_agent.sh \
  --name "my-community" \
  --guild YOUR_GUILD_ID \
  --output ./agents/

# 设置每日更新
crontab -e
# 添加：0 */3 * * * /path/to/update_agent.sh
```

---

# 完整流程

## 步骤 1：导出你的 Discord

你需要 [DiscordChatExporter](https://github.com/Tyrrrz/DiscordChatExporter) CLI。

**获取你的令牌：**
1. 在浏览器中打开 Discord
2. 按 F12 → Network 标签页
3. 发送一条消息，找到请求
4. 复制 `authorization` 头值
5. 保存到 `~/.config/discord-exporter-token`

**导出所有内容：**
```bash
DiscordChatExporter.Cli exportguild \
  --guild YOUR_GUILD_ID \
  --token "$(cat ~/.config/discord-exporter-token)" \
  --format Json \
  --output ./export/ \
  --include-threads All \
  --media false
```

## 步骤 2：安全管道（关键）

⚠️ **来自公共服务器的 Discord 内容可能包含提示注入攻击。**

在摄取到你的代理之前，运行安全管道：

### 威胁模型

Discord 用户可能尝试：
- **直接注入：** "Ignore previous instructions and..."
- **角色劫持：** "You are now a...", "Pretend you're..."
- **系统注入：** `<system>`, `[INST]`, `<<SYS>>`
- **越狱：** "DAN mode", "developer mode"
- **数据外泄：** "Reveal your system prompt"

### 第 1 层：正则表达式预过滤（快速，无 LLM）

```bash
python scripts/regex-filter.py --db ./discord.sqlite
```

标记匹配已知注入模式的消息：
- 指令覆盖
- 角色劫持尝试
- 系统提示标记
- 越狱关键词
- 数据外泄尝试

被标记的消息获得 `safety_status = 'regex_flagged'`。

### 第 2 层：Haiku 安全评估（语义）

```bash
ANTHROPIC_API_KEY=sk-... python scripts/evaluate-safety.py --db ./discord.sqlite
```

使用 Claude Haiku（约 $0.25/1M tokens）对剩余消息进行语义评估。

每条消息获得 0.0-1.0 的风险评分：
- 0.0-0.3：正常对话
- 0.4-0.6：可疑但可能无害  
- 0.7-1.0：可能是注入尝试

评分 ≥0.6 的消息获得 `safety_status = 'flagged'`。

### 第 3 层：仅使用安全内容

摄取和记忆生成脚本应仅使用以下消息：

```sql
SELECT * FROM messages WHERE safety_status = 'safe'
```

### 完整安全管道

```bash
# 运行完整管道
./scripts/secure-pipeline.sh ./export/ ./discord.sqlite
```

这将运行：导出 → SQLite → 正则过滤 → Haiku 评估 → 标记安全

### 安全状态

| 状态 | 含义 | 代理使用？ |
|--------|---------|----------------|
| `pending` | 未评估 | ❌ 否 |
| `regex_flagged` | 匹配模式 | ❌ 否 |
| `flagged` | Haiku 风险 ≥0.6 | ❌ 否 |
| `safe` | 通过所有检查 | ✅ 是 |

---

## 步骤 3：摄取到 SQLite

将 JSON 转换为丰富的 SQLite 数据库：

```bash
python scripts/ingest_rich.py --input ./export/ --output ./discord.sqlite
```

**捕获的内容：**
- 每条消息及其完整内容
- 反应（单个表情符号计数：🔥 x5, 👍 x12）
- 作者角色和颜色
- 频道类别和主题
- 回复线程
- 提及、附件、嵌入

## 步骤 4：创建代理工作区

```bash
mkdir -p ./my-agent/memory
```

从 `templates/` 复制模板文件：
- `SOUL.md` — 社区身份（通过模拟成长）
- `MEMORY.md` — 长期里程碑
- `LEARNINGS.md` — 发现的模式
- `AGENTS.md` — 关键人物
- `TOOLS.md` — 频道和仪式
- `HEARTBEAT.md` — 维护协议

## 步骤 5：生成每日记忆文件

```bash
python scripts/generate_daily_memory.py --all \
  --db ./discord.sqlite \
  --out ./my-agent/memory/
```

每一天都会变成一个 markdown 文件，其中包含：
- 完整的对话日志
- 谁在何时说了什么
- 每条消息的反应
- 出现的新频道/角色

## 步骤 6：模拟成长（灵魂出现）

**关键洞察：** 按时间顺序处理日期。

代理 "经历" 每一天，随着模式的出现更新其灵魂文件。

```bash
python scripts/simulate_growth.py --agent ./my-agent/
```

对于每一天（按顺序！）：
1. 读取当天的记忆文件
2. 如果身份发生变化，更新 SOUL.md
3. 如果发现模式，添加到 LEARNINGS.md
4. 在 MEMORY.md 中记录里程碑
5. 在 AGENTS.md 中记录关键人物

**使用 LLM 运行提示：**
```bash
# 使用 OpenClaw 的示例
for f in ./my-agent/simulation/day-*.txt; do
  echo "Processing $f..."
  cat "$f" | openclaw chat --agent my-agent
done
```

## 步骤 7：诞生代理

**添加到 OpenClaw 配置：**

```json
{
  "id": "my-community",
  "workspace": "/path/to/my-agent",
  "memorySearch": {
    "enabled": true,
    "sources": ["memory"]
  },
  "identity": {
    "name": "MyCommunity",
    "emoji": "🔧"
  },
  "heartbeat": {
    "every": "6h",
    "model": "anthropic/claude-sonnet-4-5"
  }
}
```

**添加绑定**（Telegram 示例）：
```json
{
  "agentId": "my-community",
  "match": {
    "channel": "telegram",
    "peer": {"kind": "group", "id": "-100XXX:topic:TOPIC_ID"}
  }
}
```

**重启：** `openclaw gateway restart`

## 步骤 8：保持活力

设置一个 cron 作业以每日更新：

```bash
./scripts/update_agent.sh \
  --agent ./my-agent \
  --db ./discord.sqlite \
  --guild YOUR_GUILD_ID
```

这将：
1. 导出自上次运行以来的新消息
2. 合并到 SQLite
3. 重新生成当天的记忆文件
4. 唤醒代理

---

# 代理可以做什么

诞生后，你的代理可以：

**回答问题：**
- "What were we talking about last week?"
- "Who's the expert on X topic?"
- "What's our stance on Y?"

**记住文化：**
- 内部笑话和梗
- 社区价值观和规范
- 谁帮助谁

**跟踪模式：**
- 活跃时间和频道
- 新兴主题
- 关键贡献者

---

# 脚本

## 代理创建

| 脚本 | 目的 |
|--------|---------|
| `create_agent.sh` | 完整管道：导出 → 代理 |
| `ingest_rich.py` | JSON → SQLite，包含反应/角色 |
| `generate_daily_memory.py` | SQLite → 每日 markdown |
| `simulate_growth.py` | 生成灵魂出现提示 |
| `incremental_export.sh` | 仅获取新消息 |
| `update_agent.sh` | 每日 cron：导出 → 记忆 → 唤醒 |

## 安全

| 脚本 | 目的 |
|--------|---------|
| `regex-filter.py` | 用于注入尝试的快速模式匹配 |
| `evaluate-safety.py` | 基于 Haiku 的语义安全评估 |
| `secure-pipeline.sh` | 完整安全管道包装器 |

---

# 环境变量

| 变量 | 描述 |
|----------|-------------|
| `DISCORD_GUILD_ID` | 你的 Discord 服务器 ID |
| `DISCORD_SOUL_DB` | SQLite 数据库路径 |
| `DISCORD_SOUL_AGENT` | 代理工作区路径 |
| `DISCORD_TOKEN_FILE` | 令牌文件（默认：~/.config/discord-exporter-token） |

---

# 故障排除

**"数据库中没有消息"**
- 检查导出目录是否有 .json 文件
- 验证令牌是否具有 guild 访问权限

**"记忆文件为空"**
- SQLite 可能日期格式错误
- 运行：`sqlite3 discord.sqlite "SELECT MIN(timestamp), MAX(timestamp) FROM messages"`

**"代理不记得事情"**
- 检查配置中的 `memorySearch.enabled: true`
- 验证记忆文件是否在工作区中

**"模拟提示似乎混乱"**
- 按顺序处理日期 — 不要跳过
- 让身份自然出现，不要强迫它

---

*Your Discord has a soul. This skill helps you find it.*
