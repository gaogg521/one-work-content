---
name: tavily-search-pro
slug: tavily-search-pro
description: Tavily AI 搜索平台，包含 5 种模式：Search（网页/新闻/财经搜索）、Extract（URL 内容提取）、 Crawl（网站爬取）、Map（站点地图发现）和 Research（带引用的深度研究）。 用于：带 LLM 回答的网页搜索、内容提取、站点爬取、深度研究。
version: 1.0.0
author: Leo 🦁
tags:
- search
- tavily
- web
- news
- finance
- extract
- crawl
- research
- api
metadata:
  clawdbot:
    emoji: 🔎
    requires:
      env:
      - TAVILY_API_KEY
    primaryEnv: TAVILY_API_KEY
    install:
    - id: pip
      kind: pip
      package: tavily-python
      label: 安装依赖 (pip)
allowed-tools:
- exec
---

# Tavily Search 🔎

AI 驱动的网页搜索平台，包含 5 种模式：Search、Extract、Crawl、Map 和 Research。

## 要求

- `TAVILY_API_KEY` 环境变量

## 配置

| 环境变量 | 默认值 | 描述 |
|---|---|---|
| `TAVILY_API_KEY` | — | **必需。** Tavily API key |

在 OpenClaw 配置中设置：
```json
{
  "env": {
    "TAVILY_API_KEY": "tvly-..."
  }
}
```

## 脚本位置

```bash
python3 skills/tavily/lib/tavily_search.py <command> "query" [options]
```

---

## 命令

### search — 网页搜索（默认）

通用网页搜索，可选 LLM 合成的回答。

```bash
python3 lib/tavily_search.py search "query" [options]
```

**示例：**
```bash
# 基础搜索
python3 lib/tavily_search.py search "latest AI news"

# 带 LLM 回答
python3 lib/tavily_search.py search "what is quantum computing" --answer

# 高级深度（更好的结果，2 积分）
python3 lib/tavily_search.py search "climate change solutions" --depth advanced

# 按时间过滤
python3 lib/tavily_search.py search "OpenAI announcements" --time week

# 按域名过滤
python3 lib/tavily_search.py search "machine learning" --include-domains arxiv.org,nature.com

# 国家/地区加权
python3 lib/tavily_search.py search "tech startups" --country US

# 带原始内容和图片
python3 lib/tavily_search.py search "solar energy" --raw --images -n 10

# JSON 输出
python3 lib/tavily_search.py search "bitcoin price" --json
```

**输出格式（文本）：**
```
Answer: <如果带 --answer 则为 LLM 合成的回答>

Results:
  1. Result Title
     https://example.com/article
     Content snippet from the page...

  2. Another Result
     https://example.com/other
     Another snippet...
```

---

### news — 新闻搜索

针对新闻文章优化的搜索。设置 `topic=news`。

```bash
python3 lib/tavily_search.py news "query" [options]
```

**示例：**
```bash
python3 lib/tavily_search.py news "AI regulation"
python3 lib/tavily_search.py news "Israel tech" --time day --answer
python3 lib/tavily_search.py news "stock market" --time week -n 10
```

---

### finance — 财经搜索

针对财经数据和新闻优化的搜索。设置 `topic=finance`。

```bash
python3 lib/tavily_search.py finance "query" [options]
```

**示例：**
```bash
python3 lib/tavily_search.py finance "NVIDIA stock analysis"
python3 lib/tavily_search.py finance "cryptocurrency market trends" --time month
python3 lib/tavily_search.py finance "S&P 500 forecast 2026" --answer
```

---

### extract — 从 URL 提取内容

从一个或多个 URL 提取可读内容。

```bash
python3 lib/tavily_search.py extract URL [URL...] [options]
```

**参数：**
- `urls`: 要提取的一个或多个 URL（位置参数）
- `--depth basic|advanced`: 提取深度
- `--format markdown|text`: 输出格式（默认：markdown）
- `--query "text"`: 按查询相关性对提取的块重新排序

**示例：**
```bash
# 提取单个 URL
python3 lib/tavily_search.py extract "https://example.com/article"

# 提取多个 URL
python3 lib/tavily_search.py extract "https://url1.com" "https://url2.com"

# 高级提取并带相关性重排序
python3 lib/tavily_search.py extract "https://arxiv.org/paper" --depth advanced --query "transformer architecture"

# 文本格式输出
python3 lib/tavily_search.py extract "https://example.com" --format text
```

**输出格式：**
```
URL: https://example.com/article
─────────────────────────────────
<以 markdown/text 提取的内容>

URL: https://another.com/page
─────────────────────────────────
<提取的内容>
```

---

### crawl — 爬取网站

从根 URL 开始爬取网站，跟随链接。

```bash
python3 lib/tavily_search.py crawl URL [options]
```

**参数：**
- `url`: 开始爬取的根 URL
- `--depth basic|advanced`: 爬取深度
- `--max-depth N`: 最大跟随链接深度（默认：2）
- `--max-breadth N`: 每深度级别最大页面数（默认：10）
- `--limit N`: 最大总页面数（默认：10）
- `--instructions "text"`: 自然语言爬取指令
- `--select-paths p1,p2`: 仅爬取这些路径模式
- `--exclude-paths p1,p2`: 跳过这些路径模式
- `--format markdown|text`: 输出格式

**示例：**
```bash
# 基础爬取
python3 lib/tavily_search.py crawl "https://docs.example.com"

# 带指令的聚焦爬取
python3 lib/tavily_search.py crawl "https://docs.python.org" --instructions "Find all asyncio documentation" --limit 20

# 仅爬取特定路径
python3 lib/tavily_search.py crawl "https://example.com" --select-paths "/blog,/docs" --max-depth 3
```

**输出格式：**
```
Crawled 5 pages from https://docs.example.com

Page 1: https://docs.example.com/intro
─────────────────────────────────
<Content>

Page 2: https://docs.example.com/guide
─────────────────────────────────
<Content>
```

---

### map — 站点地图发现

发现网站上的所有 URL（站点地图）。

```bash
python3 lib/tavily_search.py map URL [options]
```

**参数：**
- `url`: 要映射的根 URL
- `--max-depth N`: 跟随深度（默认：2）
- `--max-breadth N`: 每级别广度（默认：20）
- `--limit N`: 最大 URL 数（默认：50）

**示例：**
```bash
# 映射一个站点
python3 lib/tavily_search.py map "https://example.com"

# 深度映射
python3 lib/tavily_search.py map "https://docs.python.org" --max-depth 3 --limit 100
```

**输出格式：**
```
Sitemap for https://example.com (42 URLs found):

  1. https://example.com/
  2. https://example.com/about
  3. https://example.com/blog
  ...
```

---

### research — 深度研究

带引用的综合性 AI 驱动主题研究。

```bash
python3 lib/tavily_search.py research "query" [options]
```

**参数：**
- `query`: 研究问题
- `--model mini|pro|auto`: 研究模型（默认：auto）
  - `mini`: 更快，更便宜
  - `pro`: 更彻底
  - `auto`: 让 Tavily 决定
- `--json`: JSON 输出（支持结构化输出 schema）

**示例：**
```bash
# 基础研究
python3 lib/tavily_search.py research "Impact of AI on healthcare in 2026"

# 使用 pro 模型进行彻底研究
python3 lib/tavily_search.py research "Comparison of quantum computing approaches" --model pro

# JSON 输出
python3 lib/tavily_search.py research "Electric vehicle market analysis" --json
```

**输出格式：**
```
Research: Impact of AI on healthcare in 2026

<Comprehensive research report with citations>

Sources:
  [1] https://source1.com
  [2] https://source2.com
  ...
```

---

## 选项参考

| 选项 | 适用于 | 描述 | 默认值 |
|---|---|---|---|
| `--depth basic\|advanced` | search, news, finance, extract | 搜索/提取深度 | basic |
| `--time day\|week\|month\|year` | search, news, finance | 时间范围过滤 | none |
| `-n NUM` | search, news, finance | 最大结果数 (0-20) | 5 |
| `--answer` | search, news, finance | 包含 LLM 回答 | off |
| `--raw` | search, news, finance | 包含原始页面内容 | off |
| `--images` | search, news, finance | 包含图片 URL | off |
| `--include-domains d1,d2` | search, news, finance | 仅这些域名 | none |
| `--exclude-domains d1,d2` | search, news, finance | 排除这些域名 | none |
| `--country XX` | search, news, finance | 提升国家/地区结果 | none |
| `--json` | 所有 | 结构化 JSON 输出 | off |
| `--format markdown\|text` | extract, crawl | 内容格式 | markdown |
| `--query "text"` | extract | 相关性重排序查询 | none |
| `--model mini\|pro\|auto` | research | 研究模型 | auto |
| `--max-depth N` | crawl, map | 最大链接深度 | 2 |
| `--max-breadth N` | crawl, map | 每级别最大页面数 | 10/20 |
| `--limit N` | crawl, map | 最大总页面数/URL 数 | 10/50 |
| `--instructions "text"` | crawl | 自然语言指令 | none |
| `--select-paths p1,p2` | crawl | 包含路径模式 | none |
| `--exclude-paths p1,p2` | crawl | 排除路径模式 | none |

---

## 错误处理

- **缺少 API key:** 带有设置说明的清晰错误消息。
- **401 Unauthorized:** API key 无效。
- **429 Rate Limit:** 超出速率限制，请稍后重试。
- **网络错误:** 带有原因的描述性错误。
- **无结果:** 干净的 "No results found." 消息。
- **超时:** 所有 HTTP 请求 30 秒超时。

---

## 积分与定价

| API | Basic | Advanced |
|---|---|---|
| Search | 1 积分 | 2 积分 |
| Extract | 1 积分/URL | 2 积分/URL |
| Crawl | 1 积分/页面 | 2 积分/页面 |
| Map | 1 积分 | 1 积分 |
| Research | 按模型变化 | - |

---

## 安装

```bash
bash skills/tavily/install.sh
```
