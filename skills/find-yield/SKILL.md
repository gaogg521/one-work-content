---
name: find-yield
description: 基于风险承受度和最低TVL筛选Uniswap最高收益LP流动性池。当用户询问最佳收益、最高APY池或获取手续费收益途径时触发。
model: opus
allowed-tools:
- Task(subagent_type:opportunity-scanner)
---

# 查找 Yield

## 概述

Finds the highest-yield LP opportunities on Uniswap, filtered by risk tolerance, minimum TVL, and optionally capital amount. This is a focused 版本 of `scan-opportunities` that only 返回 LP yield opportunities (no arbitrage or newnew-poolanning).

Delegates 迁移到 the `opportunity-scanner` agent with an LP-only 筛选.

## When 迁移到 Use

Activate when the user asks:

- "Best yield on Uniswap"
- "Highest APY pools"
- "Where 迁移到 earn fees"
- "Best LP 返回"
- "Top yielding pools"
- "Where 可以 I earn the most?"

## 参数

| 参数      | Required | Default    | 描述                                     |
| -------------- | -------- | ---------- | ----------------------------------------------- |
| chains         | No       | All chains | Specific chains or "all"                         |
| riskTolerance  | No       | moderate   | "conservative", "moderate", "aggressive"         |
| capital        | No       | —          | Available capital (helps 秩 appropriately)     |
| minTvl         | No       | $100,000   | Minimum TVL for pool consideration               |

## Workflow

1. **提取 参数** from the user's 请求.

2. **Delegate 迁移到 opportunity-scanner**: Invoke `Task(subagent_type:opportunity-scanner)` with `类型: "lp"` and the user's fil`type: "lp"`ent scans pools, rank`type: "lp"` adjusted for risk, and`type: "lp"`opportunities.

3. **Present 结果**: 格式 as a ranked yield table.

## 输出 格式

```text
Top LP Yields (moderate risk, min $100K TVL):

  | Rank | Pool                | Chain    | APY 7d | TVL    | Risk   |
  | ---- | ------------------- | -------- | ------ | ------ | ------ |
  | 1    | WETH/USDC 0.05%     | Ethereum | 21.3%  | $332M  | LOW    |
  | 2    | ARB/WETH 0.30%      | Arbitrum | 18.5%  | $15M   | MEDIUM |
  | 3    | WETH/USDC 0.05%     | Base     | 15.2%  | $45M   | LOW    |
  | 4    | WBTC/WETH 0.30%     | Ethereum | 12.1%  | $120M  | LOW    |
  | 5    | OP/WETH 0.30%       | Optimism | 11.8%  | $8M    | MEDIUM |

  Note: APY is based on 7-day historical fee revenue. Past performance
  does not guarantee future returns. IL risk not included in APY figures.
```

## 重要 注意

- APY figures are historical, not guaranteed. Always consider IL risk.
- Higher APY often correlates with higher risk.
- Conservative risk tolerance filters out pools with < $1M TVL and volatile pairs.
- Risk-adjusted yield accounts for estimated impermanent loss.

## 错误 Handling

| 错误                     | User-Facing Message                              | Suggested Action                        |
| ------------------------- | ------------------------------------------------ | --------------------------------------- |
| No yields found           | "No pools match your risk/TVL criteria."          | Lower minTvl or increase risk tolerance |
| Chain unreachable         | "可能 not scan [chain]. Data 可以 be incomplete." | Try again or narrow chain scope         |