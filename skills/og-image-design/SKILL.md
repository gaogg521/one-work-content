---
name: og-image-design
description: 设计 Open Graph(OG) 和社交分享图像，涵盖平台规格、文本排版和品牌设计。支持 OG 元标签、Twitter 卡片、LinkedIn 预览和动态生成，适用于社交分享图、博客缩略图和链接预览。
allowed-tools: Bash(infsh *)
tags:
- 架构
---

# OG Image Design

通过 [inference.sh](https://inference.sh) CLI 创建社交分享图像（Open Graph）。

## 快速开始

```bash
curl -fsSL https://cli.inference.sh | sh && infsh login

# 使用 HTML-to-image 生成 OG 图像
infsh app run infsh/html-to-image --input '{
  "html": "<div style=\"width:1200px;height:630px;background:linear-gradient(135deg,#1a1a2e,#16213e);display:flex;align-items:center;padding:60px;font-family:system-ui;color:white\"><div><h1 style=\"font-size:56px;margin:0;line-height:1.2\">How We Reduced Build Times by 80%</h1><p style=\"font-size:24px;opacity:0.8;margin-top:20px\">engineering.yourcompany.com</p></div></div>"
}'
```

## 平台规格

| 平台 | 尺寸 | 宽高比 | 文件大小 | 格式 |
|----------|-----------|--------------|-----------|--------|
| **Facebook** | 1200 x 630 px | 1.91:1 | < 8 MB | JPG, PNG |
| **Twitter/X (summary_large_image)** | 1200 x 628 px | 1.91:1 | < 5 MB | JPG, PNG, WEBP, GIF |
| **Twitter/X (summary)** | 800 x 418 px | 1.91:1 | < 5 MB | JPG, PNG |
| **LinkedIn** | 1200 x 627 px | 1.91:1 | < 5 MB | JPG, PNG |
| **Discord** | 1200 x 630 px | 1.91:1 | < 8 MB | JPG, PNG |
| **Slack** | 1200 x 630 px | 1.91:1 | — | JPG, PNG |
| **iMessage** | 1200 x 630 px | 1.91:1 | — | JPG, PNG |

**通用安全选择：1200 x 630 px，PNG 或 JPG，低于 5 MB。**

## 黄金布局

```
┌──────────────────────────────────────────────────┐
│                                                  │
│  ┌─────────────────────────────────┐  ┌───────┐  │
│  │                                 │  │       │  │
│  │  Title Text (max 60 chars)      │  │ Logo/ │  │
│  │  ───────────────────            │  │ Visual│  │
│  │  Subtitle (max 100 chars)       │  │       │  │
│  │                                 │  │       │  │
│  │  author / site name             │  └───────┘  │
│  └─────────────────────────────────┘             │
│                                                  │
└──────────────────────────────────────────────────┘
  1200 x 630 px
```

## 设计规则

### 文本

| 规则 | 值 |
|------|-------|
| 标题字体大小 | 48-64px |
| 副标题字体大小 | 20-28px |
| 最大标题长度 | 60 个字符（在某些平台上会被截断） |
| 最大副标题长度 | 100 个字符 |
| 行高 | 标题 1.2-1.3 |
| 字重 | 标题粗体/黑体，副标题常规 |
| 文本对比度 | 至少 WCAG AA（4.5:1 比例） |

### 安全区域

```
┌──────────────────────────────────────────────────┐
│  ┌──────────────────────────────────────────────┐│
│  │ 距所有边缘 40px 内边距                        ││
│  │                                              ││
│  │  内容位于此处                                ││
│  │                                              ││
│  │                                              ││
│  └──────────────────────────────────────────────┘│
└──────────────────────────────────────────────────┘
```

- 距所有边缘至少 40px 内边距
- 某些平台会裁剪边缘或添加圆角
- 切勿将关键文本放在外部 5% 区域

### 颜色

| 背景类型 | 何时使用 |
|----------------|-------------|
| 纯色品牌色 | 一致的系列，企业 |
| 渐变 | 现代，引人注目 |
| 带叠加层的照片 | 博客文章，编辑 |
| 深色背景 | 更好的对比度，在信息流中脱颖而出 |

**深色背景在社交流中的表现优于浅色** — 大多数信息流具有白色/浅色背景，因此深色 OG 图像会弹出。

## 按内容类型的模板

### 博客文章

```bash
infsh app run infsh/html-to-image --input '{
  "html": "<div style=\"width:1200px;height:630px;background:linear-gradient(135deg,#667eea,#764ba2);display:flex;align-items:center;padding:60px;font-family:system-ui,sans-serif;color:white\"><div style=\"flex:1\"><p style=\"font-size:18px;text-transform:uppercase;letter-spacing:2px;opacity:0.8;margin:0\">Engineering Blog</p><h1 style=\"font-size:52px;margin:16px 0 0;line-height:1.2;font-weight:800\">How We Reduced Build Times by 80%</h1><p style=\"font-size:22px;opacity:0.9;margin-top:16px\">A deep dive into our CI/CD optimization</p></div></div>"
}'
```

### 产品/发布声明

```bash
infsh app run infsh/html-to-image --input '{
  "html": "<div style=\"width:1200px;height:630px;background:#0f0f0f;display:flex;align-items:center;justify-content:center;font-family:system-ui;color:white;text-align:center\"><div><p style=\"font-size:20px;color:#22c55e;text-transform:uppercase;letter-spacing:3px\">Now Available</p><h1 style=\"font-size:64px;margin:12px 0;font-weight:900\">DataFlow 2.0</h1><p style=\"font-size:24px;opacity:0.7\">Automated reports. Zero configuration.</p></div></div>"
}'
```

### 教程/How-To

```bash
infsh app run infsh/html-to-image --input '{
  "html": "<div style=\"width:1200px;height:630px;background:linear-gradient(to right,#1a1a2e,#16213e);display:flex;align-items:center;padding:60px;font-family:system-ui;color:white\"><div><div style=\"display:inline-block;background:#e74c3c;color:white;padding:8px 16px;border-radius:4px;font-size:16px;font-weight:bold;margin-bottom:16px\">TUTORIAL</div><h1 style=\"font-size:48px;margin:0;line-height:1.2\">Build a REST API in 10 Minutes with Node.js</h1><p style=\"font-size:20px;opacity:0.7;margin-top:16px\">Step-by-step guide with code examples</p></div></div>"
}'
```

### AI 生成的视觉 OG

```bash
# 当你想要一个引人注目的视觉而不是基于文本的
infsh app run falai/flux-dev-lora --input '{
  "prompt": "clean professional social sharing card, dark gradient background, abstract geometric shapes, modern tech aesthetic, minimal, no text, 1200x630 equivalent aspect ratio",
  "width": 1200,
  "height": 630
}'
```

## OG 元标签参考

```html
<!-- 必需 (Facebook, LinkedIn, Discord, Slack) -->
<meta property="og:title" content="Title here (60 chars max)" />
<meta property="og:description" content="Description (155 chars max)" />
<meta property="og:image" content="https://yoursite.com/og-image.png" />
<meta property="og:url" content="https://yoursite.com/page" />
<meta property="og:type" content="article" />

<!-- Twitter/X 特定 -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="Title here" />
<meta name="twitter:description" content="Description" />
<meta name="twitter:image" content="https://yoursite.com/og-image.png" />

<!-- 图像尺寸（可选但推荐） -->
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
```

### Twitter 卡片类型

| 卡片类型 | 图像大小 | 何时使用 |
|-----------|-----------|----------|
| `summary` | 800 x 418 (小缩略图) | 短更新，链接 |
| `summary_large_image` | 1200 x 628 (全宽) | 博客文章，文章，声明 |

**始终使用 `summary_large_image`** 除非你有特定原因不这样做 — 大图像获得的点击次数明显更多。

## 一致性系统

对于具有许多页面的博客或站点，创建一个模板系统：

| 元素 | 保持一致 | 变化 |
|---------|----------------|------|
| 背景样式 | 相同的渐变或品牌色 | — |
| 字体系列 | 相同的字体 | — |
| 布局 | 相同的位置 | — |
| Logo/品牌 | 相同的位置（角落） | — |
| 类别徽章 | 相同的样式 | 每个类别的颜色 |
| 标题文本 | 相同的大小/字重 | 内容变化 |

## 测试 OG 图像

| 工具 | URL |
|------|-----|
| Facebook Debugger | developers.facebook.com/tools/debug/ |
| Twitter Card Validator | cards-dev.twitter.com/validator |
| LinkedIn Post Inspector | linkedin.com/post-inspector/ |
| OpenGraph.xyz | opengraph.xyz |

```bash
# 研究 OG 调试工具
infsh app run tavily/search-assistant --input '{
  "query": "open graph image debugger preview tool test og:image"
}'
```

## 常见错误

| 错误 | 问题 | 修复 |
|---------|---------|-----|
| 根本没有 OG 图像 | 平台显示随机页面元素或什么都没有 | 始终设置 og:image |
| 文本太小 | 在移动预览上不可读 | 标题在 1200px 宽度下至少 48px |
| 浅色背景 | 在白色/浅色信息流中丢失 | 使用深色或饱和背景 |
| 文本太多 | 杂乱，压倒性 | 最多：标题 + 副标题 + 品牌 |
| 图像太大 (>5MB) | 某些平台不会加载它 | 理想情况下优化到低于 1MB |
| 没有安全区域内边距 | 文本在某些平台上被裁剪 | 距所有边缘 40px 内边距 |
| 每个平台不同的图像 | 不一致的分享体验 | 对所有内容使用 1200x630 |
| HTTP 图像 URL | 许多平台需要 HTTPS | 始终通过 HTTPS 提供 OG 图像 |
| 相对图像路径 | 分享时不会解析 | 使用完整的绝对 URL |

## 相关技能

```bash
npx skills add inferencesh/skills@ai-image-generation
npx skills add inferencesh/skills@landing-page-design
npx skills add inferencesh/skills@prompt-engineering
```

浏览所有应用：`infsh app list`
