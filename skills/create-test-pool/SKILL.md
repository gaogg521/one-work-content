---
name: create-test-pool
description: 在本地测试网部署可配置参数的自定义 Uniswap 池，支持薄流动性、宽价差、精确 tick 范围等条件，用于在受控场景下测试 agent 行为。触发词：Uniswap、测试网(testnet)、流动性池(liquidity pool)、tick 范围(tick range)、本地部署(local deploy)。
model: sonnet
allowed-tools:
- mcp__uniswap__deploy_mock_pool
- mcp__uniswap__fund_test_account
- mcp__uniswap__get_pool_info
- mcp__uniswap__search_tokens
tags:
- AI
---

# Create Test Pool

## Overview

根据你指定的精确参数在本地测试网上部署自定义 Uniswap 池。这让你可以创建受控测试环境——薄流动性池、极端价格范围、特定费用层级——以测试 agent 在边缘条件下的行为。

**为什么这比手动操作好 10 倍：**

1. **无需 Solidity 脚本**：手动创建 V3 池需要调用 `createAndInitializePoolIfNecessary`、计算 `sqrtPriceX96`、计算 tick 范围、授权 token 以及调用 `mint`。本 skill 全部通过自然语言完成。
2. **Token 解析**：说 "WETH/USDC" 即可解析地址、decimals 并正确排序 token。无需查找合约地址。
3. **自动注资**：如果部署者账户 token 不足，工具会处理 whale 模拟以资助部署。
4. **价格到 tick 转换**：指定一个价格如 "2000"（USDC 计价的 WETH），工具会计算正确的 `sqrtPriceX96` 和 tick 范围。
5. **边缘情况测试**：创建仅有 $100 流动性的池来测试薄市场行为，或创建极端价格的池来测试边界条件。
6. **验证**：部署后，你可以立即使用 `get_pool_info` 查询池以确认状态。

## When to Use

当用户说出以下任何内容时激活：

- "Create a WETH/USDC pool with thin liquidity"
- "Deploy a test pool with 0.05% fee"
- "Set up a DAI/USDC pool at 1:1"
- "Create a pool with only $1000 liquidity"
- "Deploy a V2 pair for testing"
- "I need a pool with a narrow tick range"
- "Create a WBTC/WETH pool at the current price"
- "Set up a pool to test high slippage scenarios"

**请勿使用**当没有测试网正在运行时（先使用 `setup-local-testnet`），或当用户想要与现有主网池交互时（使用 `analyze-pool`）。

## Parameters

| Parameter    | Required | Default   | How to Extract                                                           |
| ------------ | -------- | --------- | ------------------------------------------------------------------------ |
| token0       | Yes      | --        | First token: "WETH", "USDC", or a 0x address                            |
| token1       | Yes      | --        | Second token: "USDC", "DAI", or a 0x address                            |
| version      | No       | v3        | "v2" or "v3"                                                             |
| fee          | No       | 3000      | Fee tier: 100 (0.01%), 500 (0.05%), 3000 (0.3%), 10000 (1%)             |
| initialPrice | No       | --        | Price of token0 in token1 terms (e.g. 2000 for ETH at $2000)            |
| liquidityUsd | No       | 1,000,000 | Dollar value of initial liquidity                                        |
| tickLower    | No       | auto      | V3 lower tick (advanced users only)                                      |
| tickUpper    | No       | auto      | V3 upper tick (advanced users only)                                      |

## Workflow

### Step 1: Verify Testnet is Running

如果工具返回 `TESTNET_NOT_RUNNING`，告诉用户：

```text
No local testnet is running. Let me set one up first.
```

然后建议使用 `setup-local-testnet` 或主动为他们设置。

### Step 2: Extract Parameters

仔细解析用户的请求：

- **Token pair**: "WETH/USDC", "ETH/DAI", "WBTC/WETH"
  - 将 "ETH" 映射为 "WETH"（Uniswap 使用 wrapped ETH）
- **Fee tier**: "0.05% fee" → 500, "0.3%" → 3000, "1%" → 10000, "0.01%" → 100
- **Price**: "at $2000" → initialPrice: 2000（对于 WETH/USDC）
- **Liquidity**: "thin liquidity" → liquidityUsd: 1000, "$10M" → liquidityUsd: 10000000
- **Version**: "V2 pair" → version: "v2", 默认是 "v3"

**常见流动性描述：**
- "thin" / "low" / "shallow" → $1,000 - $10,000
- "moderate" / "normal" → $100,000 - $1,000,000
- "deep" / "high" → $10,000,000+

### Step 3: Fund Deployer If Needed

如果池需要部署者可能没有持有的 token，首先调用 `mcp__uniswap__fund_test_account` 以确保部署者（account #1: `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266`）拥有足够的 token。

### Step 4: Deploy the Pool

使用提取的参数调用 `mcp__uniswap__deploy_mock_pool`。

### Step 5: Verify and Present

展示部署完成的池的完整详情：

```text
Test Pool Deployed

  Pool:       WETH/USDC (V3, 0.05% fee)
  Address:    0xNEW...
  Price:      1 WETH = 2,000 USDC
  Liquidity:  ~$1,000,000
  Tick Range: -204714 to -199514 (±50% around current price)

  Token0: USDC  0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48  (6 decimals)
  Token1: WETH  0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2  (18 decimals)

  Test Scenarios This Pool Enables:
  - Swap testing: "Get a quote for 1 WETH → USDC"
  - LP testing: "Add liquidity to the WETH/USDC pool"
  - Price impact: "What's the price impact of swapping 100 WETH?"
  - Time-dependent: "Advance 7 days and check fee accumulation"
```

### Step 6: Suggest Follow-ups

```text
  Next Steps:
  - Query pool state: "Get info on pool 0xNEW..."
  - Test a swap against this pool
  - Create another pool with different parameters
  - Advance time to test fee accumulation: "Time travel 7 days"
```

## Important Notes

- **Tokens are automatically sorted.** Uniswap 要求 token0 < token1（按地址）。工具会自动处理。
- **V3 pools need initialization.** 工具调用 `createAndInitializePoolIfNecessary` 来设置初始价格。
- **Default tick range is ±50%.** 如果未指定 tick 范围，流动性将分布在初始价格周围的宽范围内。
- **Deployer is Anvil account #1.** 使用第一个 Anvil 默认账户进行部署。
- **Pool may already exist on fork.** 如果你 fork Ethereum 并尝试创建 WETH/USDC 0.05% 池，它已经存在。工具会向现有池添加流动性。
- **V2 pools always have 0.3% fee.** V2 池会忽略 fee 参数。

## Error Handling

| Error                          | User-Facing Message                                                    | Suggested Action                                    |
| ------------------------------ | ---------------------------------------------------------------------- | --------------------------------------------------- |
| `TESTNET_NOT_RUNNING`          | "No local testnet is running."                                         | Run setup-local-testnet first                       |
| `TESTNET_TOKEN_NOT_FOUND`      | "Cannot resolve token X."                                              | Use a well-known symbol or provide the 0x address   |
| `TESTNET_CONTRACT_NOT_FOUND`   | "NonfungiblePositionManager not found on this chain."                   | Fork Ethereum mainnet which has all V3 contracts    |
| `TESTNET_DEPLOY_POOL_FAILED`   | "Failed to deploy pool: {reason}"                                      | Check token balances, fund deployer if needed       |
