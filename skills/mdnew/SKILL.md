---
name: mdnew
description: 使用 markdown.new 服务从任何 URL 获取干净、针对代理优化的 Markdown。当 web_fetch 或浏览器无法提供干净内容，或者当你需要网页的令牌高效版本以进行深度分析时使用。
tags:
- Web
---

# mdnew

使用 `markdown.new` 三级转换管道（Header Negotiation -> Workers AI -> Browser Rendering）从任何 URL 获取干净的 Markdown。

## 用法

使用目标 URL 运行脚本：

```bash
python3 scripts/mdnew.py <url>
```

## 为什么使用 mdnew?

1. **令牌效率**: 与原始 HTML 相比，内容大小减少多达 80%。
2. **干净数据**: 剥离样板、广告和导航菜单，仅保留核心内容。
3. **JS 执行**: 通过 Cloudflare Browser Rendering 回退自动处理 JS 密集型页面。
4. **代理优先**: 包含 `x-markdown-tokens` 跟踪以帮助管理上下文窗口。
