---
name: brave-images
description: 使用 Brave Search API 搜索图片。当需要查找任何主题的图片、照片或视觉内容时使用，需配置 BRAVE_API_KEY 环境变量。
tags:
- API
- Key
- 搜索
---

# Brave 图片搜索

通过 Brave Search API 搜索图片。

## 用法

```bash
curl -s "https://api.search.brave.com/res/v1/images/search?q=QUERY&count=COUNT" \
  -H "X-Subscription-Token: $BRAVE_API_KEY"
```

## 参数

| 参数 | 必填 | 说明 |
|-------|----------|-------------|
| `q` | 是 | 搜索查询 (URL-encoded) |
| `count` | 否 | 结果数量 (1-100, 默认 20) |
| `country` | 否 | 2 字母代码 (US, DE, IL) 用于区域偏好 |
| `search_lang` | 否 | 语言代码 (en, de, he) |
| `safesearch` | 否 | off, moderate, strict (默认: moderate) |

## 响应解析

每个结果中的关键字段：
- `results[].title` —— 图片标题
- `results[].properties.url` —— 完整图片 URL
- `results[].thumbnail.src` —— 缩略图 URL  
- `results[].source` —— 来源网站
- `results[].properties.width/height` —— 尺寸

## 示例

搜索以色列的 "sunset beach" 图片：
```bash
curl -s "https://api.search.brave.com/res/v1/images/search?q=sunset%20beach&count=5&country=IL" \
  -H "X-Subscription-Token: $BRAVE_API_KEY"
```

然后从 JSON 响应中提取：
- 缩略图: `.results[0].thumbnail.src`
- 完整图片: `.results[0].properties.url`

## 交付结果

当展示图片搜索结果时：
1. 直接将图片发送给用户（不要只列出 URL）
2. 使用 `results[].properties.url` 获取完整图片或 `results[].thumbnail.src` 获取缩略图
3. 包含图片标题作为说明
4. 如果存在比展示更多的结果，告诉用户（例如，"找到 20 张图片，展示 3 张 —— 想要更多？"）

示例流程：
```
用户: "find me pictures of sunsets"
→ 使用 count=10 搜索
→ 发送 3-5 张图片并附带说明
→ "找到 10 张日落图片，展示 5 张。想要看更多？"
```

## 注意

- URL-encode 查询字符串（空格 → `%20`）
- API key 来自环境变量: `$BRAVE_API_KEY`
- 尊重每个订阅层级的速率限制
