---
name: strykr-qa-bot
description: 面向 Strykr 交易平台的 AI 驱动质量保证(QA)。针对加密、股票、新闻、AI 聊天的预构建测试。支持 CI/CD。适用于 Cursor、Claude、ChatGPT、Copilot。支持氛围编码(vibe-coding)。
version: 0.1.2
author: NextFrontierBuilds
keywords:
- strykr
- prism
- qa
- testing
- automation
- web-qa-bot
- clawdbot
- moltbot
- ai
- ai-agent
- vibe-coding
- cursor
- claude
- chatgpt
- copilot
- github-copilot
- crypto
- trading
- fintech
- openclaw
- ai-tools
- developer-tools
- devtools
- typescript
- llm
tags:
- AI
- ArgoCD
- CI/CD
---

# strykr-qa-bot

用于测试 Strykr (https://app.strykr.ai) 的 QA 自动化技能。

## 功能

Strykr AI 金融仪表板的自动化测试：
- 所有页面的预构建测试套件
- Signal card 验证
- AI 响应质量检查
- PRISM API 健康监控
- 已知问题追踪

## 何时使用

- 部署后测试 Strykr
- 回归测试
- 监控站点健康
- 验证新功能

## 用法

### 运行所有测试
```bash
cd /path/to/strykr-qa-bot
npm test
```

### 运行特定套件
```bash
npm run test:homepage
npm run test:crypto
npm run test:stocks
npm run test:news
npm run test:events
npm run test:ai-chat
```

### 快速 Smoke 测试
```bash
npm run smoke
```

### 程序化用法
```typescript
import { StrykrQABot } from 'strykr-qa-bot';

const qa = new StrykrQABot({
  baseUrl: 'https://app.strykr.ai'
});

// 运行所有套件
const results = await qa.runAll();

// 检查特定断言
await qa.expectSignalCard({ hasPrice: true, hasChart: true });
await qa.expectAIResponse({ minLength: 200 });

// 健康检查 API
const health = await qa.checkPrismEndpoints();

// 生成报告
const report = qa.generateReport();
```

## 测试套件

| 套件 | 测试 | 备注 |
|-------|-------|-------|
| homepage | 导航、widgets、状态 | 入口点 |
| crypto-signals | 筛选器、cards、actions | 有已知 modal 问题 |
| stock-signals | Asset 筛选器、actions | Stocks/ETFs/Forex |
| news | 路由、分类 | 已知直接 URL 问题 |
| events | 影响筛选器、时间 | 已知直接 URL 问题 |
| ai-chat | 输入、响应 | 质量验证 |

## 追踪的已知问题

1. **details-modal-empty** (High) - Modal 打开但内容为空
2. **direct-url-blank-news** (Medium) - /news 直接导航时空白
3. **direct-url-blank-events** (Medium) - /economic-events 空白
4. **events-widget-race-condition** (Low) - 间歇性 widget 加载

## 配置

编辑 `strykr-qa.yaml`：
```yaml
baseUrl: https://app.strykr.ai
browser:
  headless: false
  timeout: 30000
```

## 依赖

- [web-qa-bot](https://github.com/NextFrontierBuilds/web-qa-bot) (peer dependency)

## 输出

测试结果包含：
- Pass/Fail/Known-issue 状态
- 每步截图
- Console 错误捕获
- 计时指标
- Markdown 报告

## 作者

Next Frontier (@NextXFrontier)

## 链接

- [GitHub](https://github.com/NextFrontierBuilds/strykr-qa-bot)
- [Strykr](https://app.strykr.ai)
