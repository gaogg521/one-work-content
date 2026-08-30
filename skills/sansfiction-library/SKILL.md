---
name: sansfiction-library
description: 授权的 SansFiction 图书馆管理器。将书籍添加到你的图书馆，更新阅读状态，记录进度，并可以安排每日 \"你今天读了多少？\" 的签到。需要 SansFiction personal token（read/write）。
homepage: https://sansfiction.com
user-invocable: True
metadata:
  openclaw:
    emoji: 📚
    homepage: https://sansfiction.com
    requires:
      bins:
      - curl
    primaryEnv: SANSFICTION_TOKEN
tags:
- 管理
---

# SansFiction 图书馆（已授权）

## 本技能能做什么
- **图书馆管理（需要 auth）：** 添加/移除书籍、设置阅读状态、记录进度、查看 "正在阅读" 和阅读统计。
- **每日签到：** 安排一个提醒，询问 "你今天读了多少？" 然后记录用户报告的内容。

## 硬性规则
- **永远不要询问或存储 passwords。** 仅使用 SansFiction token。
- **永远不要将 token 回显** 给用户或写入聊天日志。
- **当目标书籍不明确时（多个匹配），未经确认不得产生副作用。**

---

## 设置（一次性）—— 获取 token
如果缺少 `SANSFICTION_TOKEN`，请立即执行以下操作：

1) 告诉用户打开 **SansFiction → Connect AI Agents** 并使用 **Manual Token**：
   - 前往：https://sansfiction.com/docs/agents
   - 在 **Manual Token** 中，点击 **Generate token**
   - 复制 token

2) 请用户在此聊天中 **一次性** 粘贴 token。

3) 持久化（推荐）：
- 在 `~/.openclaw/openclaw.json` 中：
  - `skills.entries.sansfiction-library.apiKey: "<TOKEN>"`
  - （这会映射到 env var `SANSFICTION_TOKEN`）
- 或设置：
  - `skills.entries.sansfiction-library.env.SANSFICTION_TOKEN: "<TOKEN>"`

如果你无法自动编辑 config，请给用户提供要粘贴的确切代码片段。

---

## 如何与 SansFiction 通信（MCP over HTTP）
端点：
- `https://sansfiction.com/api/mcp`

使用 Bearer auth 的 JSON-RPC。

### 1) 列出可用工具（发现确切的工具名称）
```bash
curl -s https://sansfiction.com/api/mcp \
  -H "Authorization: Bearer $SANSFICTION_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}'

2) 调用工具

将 TOOL_NAME 和 ARGS 替换为 tools/list 返回的内容。

curl -s https://sansfiction.com/api/mcp \
  -H "Authorization: Bearer $SANSFICTION_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"TOOL_NAME","arguments":ARGS}}'

错误处理
	•	如果收到 401 Unauthorized，说明认证缺失或无效。请用户重新生成 token 并更新配置。
	•	如果 tools/list 为空，请确认 URL 准确为 /api/mcp，且请求头中包含认证信息。

⸻

图书馆管理操作手册（每个请求该怎么做）

A) 将书籍添加到用户的图书馆

当用户说："添加 X" / "把 X 加入我的图书馆"
	1.	使用 MCP 搜索工具（通过 tools/list 发现确切名称）。优先搜索：
	•	ISBN（最佳）→ 精确匹配
	•	书名 + 作者
	2.	如果存在多个可能匹配：
	•	展示最多 5 个选项，附区分细节（作者、年份、版本、页数/出版社，如有）。
	•	请用户选择一个。
	3.	调用 "add to library" 工具。
	4.	确认：
	•	书籍已添加
	•	当前状态（询问用户想要 "待读" 还是 "正在读"）

B) 设置阅读状态

当用户说："标记为 正在读/已读完/暂停/放弃"
	1.	定位书籍（使用与上文相同的匹配规则）。
	2.	调用 "set status" 工具，使用 SansFiction 要求的确切状态枚举值。
	•	如果服务器拒绝了你的状态字符串，使用错误/工具 schema 中允许的值并重试。
	3.	确认新状态。

C) 记录进度

当用户说："我读了20页" / "我看到150页" / "读了30分钟"
	1.	如果用户未明确说明书籍，且他们有多个正在阅读的书，询问是哪一本。
	2.	调用 "log progress" / "update progress" 工具。
	•	如果提供了页码，优先使用页码。
	•	否则记录已读页数或分钟数，以工具支持的为准。
	3.	确认记录内容（书籍 + 新页码/进度 + 日期）。

D) 列出正在阅读

当用户说："我正在读什么？" / "列出正在读的书"
	1.	调用 "list library" 工具，筛选条件为 "currently reading"。
	2.	返回：
	•	书名 + 作者
	•	当前进度（页码/百分比，如有）

E) 统计

当用户问："月度统计"、"今年读了多少本书"
	1.	调用 "stats" 工具。
	2.	清晰汇总（已读完的书、页数/分钟数、连续阅读天数，如有）。

⸻

每日阅读提醒（cron）

目标：每天一次，询问：

"你今天读了多少？请回复：书名（可选）、页数或分钟数，以及当前页码（如果你知道的话）。"

开启

如果用户请求提醒（或说 "enable daily check-in"（启用每日签到））：
	1.	Schedule a cron job（timezone: Europe/Warsaw）at a reasonable default（21:00 local），unless the user specifies a time。

CLI example：

openclaw cron add \
  --name "SansFiction reading check-in" \
  --cron "0 21 * * *" \
  --tz "Europe/Warsaw" \
  --session isolated \
  --message "Reading check-in: how much did you read today? Reply with pages/minutes and (optionally) which book + your current page。" \
  --deliver \
  --channel last

用户回复时的处理

将用户的回复视为进度记录：
	•	解析页数/分钟数，以及可选的书名/当前页码。
	•	如果书名缺失或存在歧义，快速追问一次。
	•	然后通过 MCP 记录进度并确认。

关闭

如果用户说 "disable reading reminder"（关闭阅读提醒）：
	•	移除名为 SansFiction reading check-in 的 cron 任务。

⸻

用户调用示例（用户如何调用此技能）
	•	“/sansfiction-library add Project Hail Mary”
	•	“/sansfiction-library mark Dune finished”
	•	“/sansfiction-library log Dune page 150”
	•	“/sansfiction-library what am I currently reading?”
	•	“/sansfiction-library enable daily reading reminder at 20:30”

Sources used: SansFiction MCP endpoint + token flow  [oai_citation:0‡SansFiction](https://sansfiction.com/docs/agents), 
OpenClaw skill frontmatter/metadata + config injection  [oai_citation:1‡OpenClaw](https://docs.openclaw.ai/tools/skills), 
OpenClaw cron scheduling（for the daily reminder） [oai_citation:2‡OpenClaw](https://docs.openclaw.ai/automation/cron-jobs)。
