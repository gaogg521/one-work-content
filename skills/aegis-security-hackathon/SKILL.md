---
name: aegis-security-hackathon
version: 1.0.0
description: 面向 AI agent 的区块链安全扫描器（testnet）。通过 x402 协议使用 Base Sepolia USDC 支付。支持 token 风险检测、地址信誉检查与交易模拟。触发词：区块链安全(blockchain security)、token 扫描(token scan)、交易模拟(transaction simulation)、x402 支付(x402 payment)、Base Sepolia。
homepage: https://hackathon.aegis402.xyz
metadata: None
api_base: https://hackathon.aegis402.xyz/v1
category: blockchain-security
emoji: 🛡️
network: testnet
tags:
- AI
- 安全
---

# Aegis402 Shield Protocol (Hackathon/Testnet)

面向AI agent的区块链安全API。**Testnet版本** - 使用Base Sepolia USDC支付。

> ⚠️ 这是hackathon/testnet部署版本。如需生产环境使用，请访问 [aegis-security](https://aegis402.xyz/skill.md)。

## Skill文件

| 文件 | URL |
|------|-----|
| **SKILL.md** (本文件) | `https://hackathon.aegis402.xyz/skill.md` |
| **package.json** (metadata) | `https://hackathon.aegis402.xyz/skill.json` |

**Base URL:** `https://hackathon.aegis402.xyz/v1`

## 快速开始

```bash
npm install @x402/fetch @x402/evm
```

```typescript
import { x402Client, wrapFetchWithPayment } from '@x402/fetch';
import { ExactEvmScheme } from '@x402/evm/exact/client';

const client = new x402Client()
  .register('eip155:*', new ExactEvmScheme(yourEvmWallet));

const fetch402 = wrapFetchWithPayment(fetch, client);

// 在Base Sepolia上进行支付（testnet USDC）
const res = await fetch402('https://hackathon.aegis402.xyz/v1/check-token/0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48?chain_id=1');
const data = await res.json();
```

**要求:** Base Sepolia上的Testnet USDC（chain ID 84532）

**获取testnet USDC:** [Base Sepolia Faucet](https://www.coinbase.com/faucets/base-ethereum-sepolia-faucet)

---

## 定价 (Testnet USDC)

| Endpoint | 价格 | 使用场景 |
|----------|-------|----------|
| `POST /simulate-tx` | $0.05 | Transaction simulation, DeFi安全 |
| `GET /check-token/:address` | $0.01 | Token honeypot检测 |
| `GET /check-address/:address` | $0.005 | Address信誉检查 |

---

## Endpoints

### Check Token ($0.01)

扫描任何token是否存在honeypot、诈骗和风险。

```bash
curl "https://hackathon.aegis402.xyz/v1/check-token/0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48?chain_id=1"
```

**响应:**
```json
{
  "address": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "isHoneypot": false,
  "trustScore": 95,
  "risks": [],
  "_meta": { "requestId": "uuid", "duration": 320 }
}
```

### Check Address ($0.005)

验证address是否被标记为phishing或poisoning。

```bash
curl "https://hackathon.aegis402.xyz/v1/check-address/0x742d35Cc6634C0532925a3b844Bc454e4438f44e"
```

**响应:**
```json
{
  "address": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
  "isPoisoned": false,
  "reputation": "NEUTRAL",
  "tags": ["wallet", "established"],
  "_meta": { "requestId": "uuid", "duration": 180 }
}
```

### Simulate Transaction ($0.05)

在签名前预测balance变化并检测威胁。

```bash
curl -X POST "https://hackathon.aegis402.xyz/v1/simulate-tx" \
  -H "Content-Type: application/json" \
  -d '{
    "from": "0xYourWallet...",
    "to": "0xContract...",
    "value": "1000000000000000000",
    "data": "0x...",
    "chain_id": 8453
  }'
```

**响应:**
```json
{
  "isSafe": true,
  "riskLevel": "LOW",
  "simulation": {
    "balanceChanges": [
      { "asset": "USDC", "amount": "-100.00", "address": "0x..." }
    ]
  },
  "warnings": [],
  "_meta": { "requestId": "uuid", "duration": 450 }
}
```

---

## x402支付流程 (Testnet)

1. Agent调用任意付费endpoint
2. 收到 `402 Payment Required` 以及Base Sepolia支付说明
3. 在Base Sepolia上支付testnet USDC（chain ID: 84532）
4. 使用支付证明header重试请求
5. 获取安全扫描结果

**网络:** Base Sepolia (eip155:84532)
**货币:** Testnet USDC

---

## AI Agent使用场景

### 在Swap Token前
```typescript
const tokenCheck = await fetch402(`https://hackathon.aegis402.xyz/v1/check-token/${tokenAddress}?chain_id=8453`);
const { isHoneypot, trustScore } = await tokenCheck.json();

if (isHoneypot || trustScore < 50) {
  console.log('⚠️ 检测到高风险token！');
}
```

### 在签名Transaction前
```typescript
const simulation = await fetch402('https://hackathon.aegis402.xyz/v1/simulate-tx', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ from, to, value, data, chain_id: 8453 })
});

const { isSafe, riskLevel, warnings } = await simulation.json();

if (!isSafe || riskLevel === 'CRITICAL') {
  console.log('🚨 危险transaction！', warnings);
}
```

---

## 风险等级

| 等级 | 含义 |
|-------|---------|
| `SAFE` | 未检测到问题 |
| `LOW` | 轻微担忧，总体安全 |
| `MEDIUM` | 存在一定风险，谨慎操作 |
| `HIGH` | 检测到重大风险 |
| `CRITICAL` | 请勿继续操作 |

---

## 支持的链 (用于扫描)

| 链 | ID | check-token | check-address | simulate-tx |
|-------|-----|-------------|---------------|-------------|
| Ethereum | 1 | ✅ | ✅ | ✅ |
| Base | 8453 | ✅ | ✅ | ✅ |
| Polygon | 137 | ✅ | ✅ | ✅ |
| Arbitrum | 42161 | ✅ | ✅ | ✅ |
| Optimism | 10 | ✅ | ✅ | ✅ |
| BSC | 56 | ✅ | ✅ | ✅ |

---

## Health Check (免费)

```bash
curl https://hackathon.aegis402.xyz/health
```

---

## 链接

- **Hackathon API**: https://hackathon.aegis402.xyz
- **Production API**: https://aegis402.xyz
- **GitHub**: https://github.com/SwiftAdviser/aegis-402-shield-protocol
- **x402 Protocol**: https://docs.x402.org

---

🛡️ 为Agentic Economy构建。由x402 Protocol提供支持。
