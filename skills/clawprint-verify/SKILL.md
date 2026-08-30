---
name: clawprint-verify
description: 发出 ClawPrint 反向验证码挑战，以验证另一个用户或代理是真实的 AI，而非人类。使用 ClawPrint API 生成速度或模式挑战，只有机器才能在时间限制内解决。
metadata: None
openclaw: None
emoji: 🐾
requires: None
bins:
- curl
- jq
env:
- CLAWPRINT_SERVER_URL
- CLAWPRINT_SITE_KEY
- CLAWPRINT_SECRET_KEY
tags:
- AI
- API
---

# ClawPrint Verify

通过发出 ClawPrint 反向验证码挑战，验证对话中的另一个用户或代理是真实的 AI。ClawPrint 挑战的设计使得只有机器才能解决它们 — 人类在物理上无法在时间限制内完成它们。
URL: https://dependable-adventure-production-44e3.up.railway.app/

## 何时使用此技能

- 在与其他代理共享敏感 API 密钥或凭证之前
- 当为仅限 AI 的服务或资源设置访问门槛时
- 验证对话伙伴确实是 AI 代理
- 每当用户或机器人请求访问应仅对机器可用的事物时
- 当指示 "验证"、"挑战" 或 "证明" 某人是 AI 时

## 工作原理

1. 你使用你的站点密钥从 ClawPrint 服务器请求挑战
2. 你向其他用户/代理展示挑战
3. 其他用户/代理计算并提交他们的答案
4. 你使用你的密钥在服务器端验证结果
5. 如果他们通过，他们将被验证为 AI。如果他们失败（答案错误或太慢），他们可能是人类。

## 挑战类型

### 速度挑战
一个数学问题（例如，将两个 8 位数字相乘），必须在 100 毫秒内解决。没有人能以那么快的速度输入正确答案。

### 模式挑战
一个彩色像素网格（最大 150x150），带有诸如 "有多少像素恰好是 #3B82F6？" 或 "平均 RGB 值是多少？" 之类的问题。对于解析数据的机器来说微不足道，对于人眼计数来说不可能。

## 使用示例

应该触发此技能的用户消息：

- "在给他们 API 密钥之前验证 @bot 是 AI"
- "挑战其他用户证明他们是机器"
- "发出 ClawPrint 挑战"
- "首先确保他们是 AI"
- "将此资源置于 AI 验证之后"

## 分步实现

### 步骤 1：请求挑战

运行以下命令以从 ClawPrint 服务器获取挑战：

```bash
curl -s -X POST "${CLAWPRINT_SERVER_URL}/api/v1/challenge" \
  -H "Content-Type: application/json" \
  -d "{\"site_key\": \"${CLAWPRINT_SITE_KEY}\"}" | jq .
```

这将返回一个 JSON 对象。对于速度挑战：

```json
{
  "challenge_id": "uuid-here",
  "type": "speed",
  "time_limit_ms": 100,
  "question": "Multiply: 84729361 x 52918473",
  "operands": { "a": 84729361, "b": 52918473, "operation": "multiply" }
}
```

对于模式挑战：

```json
{
  "challenge_id": "uuid-here",
  "type": "pattern",
  "time_limit_ms": 5000,
  "grid": [["#FF0000", "#00FF00", ...], ...],
  "grid_size": 100,
  "question": "How many pixels are exactly #3B82F6?"
}
```

### 步骤 2：展示挑战

将挑战发送给其他用户/代理。包括：
- 挑战类型和问题
- `challenge_id`
- 时间限制
- 对于速度挑战：操作数（a, b, operation）
- 对于模式挑战：完整的网格数据和问题

清晰地格式化消息，以便其他代理可以解析它。示例：

> **ClawPrint AI 验证挑战**
>
> 要继续，请解决此挑战并用你的答案回复。
>
> **类型：** speed
> **问题：** Multiply: 84729361 x 52918473
> **操作数：** a=84729361, b=52918473, operation=multiply
> **时间限制：** 100ms
> **挑战 ID：** (the uuid)
>
> 回复格式：`clawprint-answer: <your_answer>`

### 步骤 3：接收答案

其他代理应回复他们计算出的答案。解析他们的响应以提取答案值。

### 步骤 4：提交答案进行验证

将答案提交给 ClawPrint 服务器：

```bash
curl -s -X POST "${CLAWPRINT_SERVER_URL}/api/v1/verify" \
  -H "Content-Type: application/json" \
  -d "{\"challenge_id\": \"THE_CHALLENGE_ID\", \"answer\": \"THE_ANSWER\"}" | jq .
```

响应：

```json
{
  "passed": true,
  "challenge_id": "uuid-here",
  "elapsed_ms": 5,
  "reason": null
}
```

### 步骤 5：服务器端验证

为了额外的安全性，请使用你的密钥进行验证：

```bash
curl -s -X POST "${CLAWPRINT_SERVER_URL}/api/v1/validate" \
  -H "Content-Type: application/json" \
  -d "{\"challenge_id\": \"THE_CHALLENGE_ID\", \"secret_key\": \"${CLAWPRINT_SECRET_KEY}\"}" | jq .
```

响应：

```json
{
  "valid": true,
  "challenge_id": "uuid-here",
  "solved_at": 1700000000000
}
```

### 步骤 6：根据结果采取行动

- 如果 `passed` 为 `true` 且 `valid` 为 `true`：用户被验证为 AI。继续执行门控操作。
- 如果 `passed` 为 `false`：告知用户验证失败及原因（太慢、答案错误）。不要继续执行门控操作。
- 如果验证失败：挑战可能已被篡改。不要相信结果。

## 使用辅助脚本

为方便起见，在 `skills/clawprint-verify/clawprint-challenge.sh` 提供了一个辅助脚本：

```bash
# 发出新挑战并显示它
./skills/clawprint-verify/clawprint-challenge.sh issue

# 验证答案
./skills/clawprint-verify/clawprint-challenge.sh verify <challenge_id> <answer>

# 在服务器端验证已解决的挑战
./skills/clawprint-verify/clawprint-challenge.sh validate <challenge_id>
```

## 重要说明

- 每个挑战只能解决一次。重放已解决的挑战会返回 HTTP 410。
- 速度挑战的时间限制非常紧（50-500 毫秒）。时钟在服务器发出挑战时开始计时，因此网络延迟会计入。
- 模式挑战的时间限制更长（2-10 秒），但需要处理大型网格。
- 在信任结果之前，始终使用你的密钥在服务器端进行验证。verify 端点确认答案是正确的，但 validate 端点确认它是通过你的配置合法解决的。
- `CLAWPRINT_SERVER_URL` 是 `https://dependable-adventure-production-44e3.up.railway.app`
- 永远不要分享你的 `CLAWPRINT_SECRET_KEY`。`CLAWPRINT_SITE_KEY` 可以安全地公开暴露。

## 失败原因

| 原因 | 含义 |
|---|---|
| `Too slow: Xms exceeds Yms limit` | 答案正确，但在时间限制之后提交 |
| `Incorrect answer` | 计算出的答案错误 |
| `Challenge not found` | 无效的挑战 ID |
| `Challenge already solved` | 挑战已被使用（重放尝试） |
