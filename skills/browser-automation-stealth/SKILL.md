---
name: browser-automation-stealth
description: 反机器人规避 Playwright 包装器，提供隐身模式、代理(proxy)轮换、验证码(captcha)处理与指纹(fingerprint)随机化。触发词：浏览器自动化(browser automation)、反检测(anti-detection)、Playwright、网页抓取(scraping)、代理轮换(proxy rotation)。
tags:
- Nginx
- 自动化
---

# 浏览器自动化隐身

**版本:** 1.0.0  
**作者:** Midas Skills  
**许可证:** MIT

## 描述
反机器人规避 Playwright 包装器。隐身模式、代理轮换、验证码处理、指纹随机化。

## 价值主张
反机器人规避 Playwright 包装器。规避检测、管理 cookies、轮换 headers、处理验证码。静默、无头、不可检测。

## 类别
browser-automation

## 标签
stealth, anti-detection, playwright, scraping, automation

## 技能类型
automation

## 定价
- **免费:** $0
- **专业版:** $49.99

## 主要功能
- ✅ 带隐身默认设置的 Playwright 包装器
- ✅ 反检测机制（指纹随机化）
- ✅ Header 轮换（100+ user-agents）
- ✅ 代理支持（SOCKS5, HTTP）
- ✅ Cookie jar 管理
- ✅ 验证码绕过（集成就绪）
- ✅ 速率限制感知
- ✅ 截图/PDF 生成
- ✅ 表单自动化
- ✅ Cookie/session 持久化

## 使用场景
- 大规模网页抓取（未被检测）
- 受保护站点的自动化测试
- 市场研究的数据收集
- 竞争情报收集
- 自动化表单提交（合规）
- 无检测的截图自动化

## 安装
```bash
npm install browser-automation-stealth
# 或
pip install browser-automation-stealth
```

## 快速开始
```javascript
const { StealthBrowser } = require('browser-automation-stealth');

const browser = new StealthBrowser({
  headless: true,
  stealth: 'aggressive'  // 规避级别
});

const page = await browser.newPage();
await page.goto('https://example.com');
await page.screenshot({ path: 'example.png' });
await browser.close();
```

## 仓库
https://github.com/midas-skills/browser-automation-stealth

## 支持
📧 support@midas-skills.com  
🔗 文档: https://docs.midas-skills.com/browser-automation-stealth
