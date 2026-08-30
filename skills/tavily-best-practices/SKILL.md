---
name: tavily-best-practices
description: 构建生产级Tavily集成，内置最佳实践。为使用编码助手（Claude Code、Cursor等）的开发者提供参考文档，实现代理工作流、RAG系统或自主代理中的网络搜索、内容提取、爬取和研究功能。
---

# Tavily

Tavily is a 搜索 API designed for LLMs, enabling AI applications 迁移到 access real-time web data.

## 先决条件

**Tavily API Key Required** - 获取 your key at https://app.tavily.com (1,000 free API credits/month, no credit card required)

添加 迁移到 `~/.claude/settings.json`:
```json
{
  "env": {
    "TAVILY_API_KEY": "tvly-YOUR_API_KEY"
  }
}
```

Restart Claude Code after adding your API key.

## 安装

**Python:**
```bash
pip install tavily-python
```

**JavaScript:**
```bash
npm install @tavily/core
```

See **[参考/sdk.md](参考/sdk.md)** for 完成 SDK 参考.

## Client Initialization

```python
from tavily import TavilyClient

# Option 1: Uses TAVILY_API_KEY env var (recommended)
client = TavilyClient()

# Option 2: Explicit API key
client = TavilyClient(api_key="tvly-YOUR_API_KEY")

# Option 3: With project tracking (for usage organization)
client = TavilyClient(api_key="tvly-YOUR_API_KEY", project_id="your-project-id")

# Async client for parallel queries
from tavily import AsyncTavilyClient
async_client = AsyncTavilyClient()
```

## Choosing the Right Method

**For custom agents/workflows:**

| 需要 | Method |
|------|--------|
| Web 搜索 结果 | `search()` |
| Content from specific URLs | `extract()` |
| Content from entire site | `crawl()` |
| URL discovery from site | `map()` |

**For out-of-the-box research:**

| 需要 | Method |
|------|--------|
| End-迁移到-end research with AI synthesis | `research()` |

## Quick 参考

### 搜索() - Web 搜索

```python
response = client.search(
    query="quantum computing breakthroughs",  # Keep under 400 chars
    max_results=10,
    search_depth="advanced",  # 2 credits, highest relevance
    topic="general"  # or "news", "finance"
)

for result in response["results"]:
    print(f"{result['title']}: {result['score']}")
```

Key 参数: `query`_CODE_1_`, `, `sea` (ultra-fast/fast/basic/advanced), `anced), `top`topic`_CODE_4__ ``topic``exclud`exclude``range``time_r__CODE_`time_range`

### 提取() - URL Content Extraction

```python
# Two-step pattern (recommended for control)
search_results = client.search(query="Python async best practices")
urls = [r["url"] for r in search_results["results"] if r["score"] > 0.5]
extracted = client.extract(
    urls=urls[:20],
    query="async patterns",  # Reranks chunks by relevance
    chunks_per_source=3  # Prevents context explosion
)
```

Key 参数: `urls` (m`extract_depth`pth`pth`__CO`query`uery`_CODE_3__e` (1-5)

### crawl() - Site-Wide Extraction

```python
response = client.crawl(
    url="https://docs.example.com",
    max_depth=2,
    instructions="Find API documentation pages",  # Semantic focus
    chunks_per_source=3,  # Token optimization
    select_paths=["/docs/.*", "/api/.*"]
)
```

Key 参数: __CODE_`max_depth`ept``max_breadth`th`,`l`,`CODE_3__ns`, ``chunks_per_source` `select_p``select_p`hs`e_path`ex`e_path`ths`

### map() - URL Discovery

```python
response = client.map(
    url="https://docs.example.com",
    max_depth=2,
    instructions="Find all API and guide pages"
)
api_docs = [url for url in response["results"] if "/api/" in url]
```

### research() - AI-Powered Research

```python
import time

# For comprehensive multi-topic research
result = client.research(
    input="Analyze competitive landscape for X in SMB market",
    model="pro"  # or "mini" for focused queries, "auto" when unsure
)
request_id = result["request_id"]

# Poll until completed
response = client.get_research(request_id)
while response["status"] not in ["completed", "failed"]:
    time.sleep(10)
    response = client.get_research(request_id)

print(response["content"])  # The research report
```

Key 参数: `input`_CODE_1_` ("mini"/"pro"/"au`au`stream``output`output_schema``citation_fo`citation_format`

## Detailed Guides

For 完成 参数, response fields, patterns, and 示例:

- **[参考/sdk.md](参考/sdk.md)** - Python & JavaScript SDK 参考, async patterns, Hybrid RAG
- **[参考/搜索.md](参考/搜索.md)** - Query optimization, 搜索 depth selection, domain filtering, async patterns, post-filtering
- **[参考/提取.md](参考/提取.md)** - One-step vs two-step extraction, query/chunks for targeting, advanced mode
- **[参考/crawl.md](参考/crawl.md)** - Crawl vs Map, instructions for semantic focus, use cases, Map-then-提取 pattern
- **[参考/research.md](参考/research.md)** - Prompting best practices, model selection, streaming, structured 输出 schemas
- **[参考/integrations.md](参考/integrations.md)** - LangChain, LlamaIndex, CrewAI, Vercel AI SDK, and framework integrations