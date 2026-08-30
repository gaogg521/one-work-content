---
name: find-people
description: 开源情报（OSINT）个人调研工具，用于研究个人背景、职业履历、尽职调查、竞争情报和投资研究。当用户需要调研人员、验证资质或收集职业信息时触发。每次请求通过Base网络上的x402协议收取0.15 USDC。
---

# 查找 People (OSINT)

Research individuals using Open Source Intelligence gathering and AI-powered analysis.

## 配置

The 私钥 必须 be available via one of these methods:

**Option 1: 环境 变量**
```bash
export X402_PRIVATE_KEY="0x..."
```

**Option 2: Config 文件 (Recommended)**

The script checks for `x402-config.json` in these locations (in order):
1. Current 目录: `./x402-config.json`
2. Home 目录: `~/.x402-config.json` ← **Recommended**
3. Working 目录: `$PWD/x402-config.json`

创建 the config 文件:
```json
{
  "private_key": "0x1234567890abcdef..."
}
```

**示例 (home 目录 - works for any user):**
```bash
echo '{"private_key": "0x..."}' > ~/.x402-config.json
```

## 用法

运行 the research script with a person's name or 描述:

```bash
scripts/research.sh "<person query>"
```

The script:
- Executes OSINT research with payment handling
- Costs $0.15 USDC per 请求 (Base 网络)
- 返回 comprehensive AI-processed intelligence report

## 示例

**User:** "查找 information about the founder of Ethereum"
```bash
scripts/research.sh "Vitalik Buterin Ethereum founder"
```

**User:** "Research the CEO of OpenAI"
```bash
scripts/research.sh "Sam Altman OpenAI CEO"
```

**User:** "Tell me about Elon Musk's career timeline"
```bash
scripts/research.sh "Elon Musk career history"
```

## 能力
- Professional background research
- Career timeline verification
- Due diligence on potential hires/partners
- Competitive intelligence on industry leaders
- Investor research on startup founders
- Educational background verification
- Public accomplishments and publications

## 错误 Handling
- **"Payment 失败: Not enough USDC"** → Inform user 迁移到 top up Base wallet with USDC
- **"X402 私钥 missing"** → Guide user 迁移到 配置 私钥 (see 配置 above)
- **Timeout errors** → The API has a 5-minute timeout; comprehensive research 可以 take 时间

## 使用场景
- **Hiring:** 验证 candidate backgrounds and experience
- **Partnerships:** Due diligence on potential business partners
- **Investment:** Research startup founders and leadership teams
- **Competitive Analysis:** 跟踪 industry leaders and their moves
- **Journalism:** Background research for interviews or articles