---
name: portfolio-watcher
description: 监控股票/加密货币持仓，获取价格预警，跟踪投资组合表现
author: clawd-team
version: 1.0.0
triggers:
- check portfolio
- stock price
- crypto price
- set price alert
- portfolio performance
tags:
- 金融
---

# Portfolio Watcher

通过自然对话监控你的投资。实时价格、预警和投资组合表现跟踪。

## 它能做什么

跟踪你的股票和加密货币持仓，获取实时价格，在达到目标时发送预警，并计算投资组合表现。无需经纪商连接——只需告诉 Clawd 你拥有什么。

## 用法

**添加持仓：**
```
"I own 50 shares of AAPL at $150"
"Add 0.5 BTC bought at $40,000"
"Track NVDA, bought 20 shares at $280"
```

**检查价格：**
```
"What's TSLA at?"
"Bitcoin price"
"Check all my stocks"
```

**设置预警：**
```
"Alert me if AAPL hits $200"
"Notify when ETH drops below $2000"
"Remove MSFT alert"
```

**投资组合概览：**
```
"How's my portfolio doing?"
"Total gains/losses"
"Best and worst performers"
```

## 支持的资产

- US stocks (NYSE, NASDAQ)
- Major cryptocurrencies
- ETFs
- International stocks (limited)

## 提示

- 包含购买价格以实现准确的盈亏跟踪
- 说 "update [ticker] to [shares] at [price]" 以修改持仓
- 询问 "portfolio allocation" 以获取饼图细分
- 价格每隔几分钟更新一次（非实时流式传输）
- 仅供参考——不是财务建议
