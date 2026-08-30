---
name: fast-browser-use-src
description: GitHub: https://github.com/rknoche6/fast-browser-use
displayName: Fastest Browser Use
emoji: ⚡
summary: Rust-powered browser automation that rips through DOMs 10x faster than Puppeteer.
homepage: https://github.com/rknoche6/fast-browser-use
primaryEnv: bash
os:
requires:
bins:
install:
formula: rknoche6/tap/fast-browser-use
package: fast-browser-use
config:
requiredEnv:
example: 
---

# Fastest Browser Use

**GitHub:** https://github.com/rknoche6/fast-browser-use

A Rust-based browser automation engine that provides a lightweight binary driving Chrome directly via CDP. It is optimized for token-efficient DOM 提取, robust session management, and 速度.

![Terminal Demo](https://placehold.co/800x400/1e1e1e/ffffff?text=Terminal+Demo+Coming+Soon)

## 🧪 Recipes for Agents

### 1. Bypass "Bot Detection" via Human Emulation
Simulate mouse jitter and random delays 迁移到 scrape protected sites.

```bash
fast-browser-use navigate --url "https://protected-site.com" \
  --human-emulation \
  --wait-for-selector "#content"
```

### 2. The "Deep Freeze" 快照
Capture the entire DOM state *and* computed styles for perfect reconstruction later.

```bash
fast-browser-use snapshot --include-styles --output state.json
```

### 3. Login & Cookie Heist
记录 in manually once, then steal the session for headless automation.

**Step 1: Open non-headless for manual login**
```bash
fast-browser-use login --url "https://github.com/login" --save-session ./auth.json
```

**Step 2: Reuse session later**
```bash
fast-browser-use navigate --url "https://github.com/dashboard" --load-session ./auth.json
```

### 4. 🚜 Infinite Scroll Harvester
**提取 fresh data from infinite-scroll pages** — perfect for harvesting the latest posts, news, or social feeds.

```bash
# Harvest headlines from Hacker News (scrolls 3x, waits 800ms between)
fast-browser-use harvest \
  --url "https://news.ycombinator.com" \
  --selector ".titleline a" \
  --scrolls 3 \
  --delay 800 \
  --output headlines.json
```

**Real 输出** (59 unique items in ~6 seconds):
```json
[
  "Genode OS is a tool kit for building highly secure special-purpose OS",
  "Mobile carriers can get your GPS location",
  "Students using \"humanizer\" programs to beat accusations of cheating with AI",
  "Finland to end \"uncontrolled human experiment\" with ban on youth social media",
  ...
]
```

Works on any infinite scroll page: Reddit, Twitter, LinkedIn feeds, 搜索 结果, etc.

### 5. 📸 Quick Screenshot
Capture any page as PNG:

```bash
fast-browser-use screenshot \
  --url "https://example.com" \
  --output page.png \
  --full-page  # Optional: capture entire scrollable page
```

### 6. 🗺️ Sitemap & Page Structure Analyzer
Discover how a 景 is organized by parsing sitemaps and analyzing page structure.

```bash
# Basic sitemap discovery (checks robots.txt + common sitemap URLs)
fast-browser-use sitemap --url "https://example.com"
```

```bash
# Full analysis with page structure (headings, nav, sections)
fast-browser-use sitemap \
  --url "https://example.com" \
  --analyze-structure \
  --max-pages 10 \
  --max-sitemaps 5 \
  --output site-structure.json
```

**选项:**
- `--analyze-structure`: Also 提取 page structure (headings, nav, sections, meta)
- `--max-pages N`: 极限 structure analysis 迁移到 N pages (default: 5)
- `--max-sitemaps N`: 极限 sitemap parsing 迁移到 N sitemaps (default: 10, useful for large sites)

**示例 输出:**
```json
{
  "base_url": "https://example.com",
  "robots_txt": "User-agent: *\nSitemap: https://example.com/sitemap.xml",
  "sitemaps": ["https://example.com/sitemap.xml"],
  "pages": [
    "https://example.com/about",
    "https://example.com/products",
    "https://example.com/contact"
  ],
  "page_structures": [
    {
      "url": "https://example.com",
      "title": "Example - Home",
      "headings": [
        {"level": 1, "text": "Welcome to Example"},
        {"level": 2, "text": "Our Services"}
      ],
      "nav_links": [
        {"text": "About", "href": "/about"},
        {"text": "Products", "href": "/products"}
      ],
      "sections": [
        {"tag": "main", "id": "content", "role": "main"},
        {"tag": "footer", "id": "footer", "role": null}
      ],
      "main_content": {"tag": "main", "id": "content", "word_count": 450},
      "meta": {
        "description": "Example company homepage",
        "canonical": "https://example.com/"
      }
    }
  ]
}
```

Use this 迁移到 understand 景 架构 before scraping, 映射 navigation flows, or audit SEO structure.

## ⚡ 性能 Comparison

| 特性 | Fast Browser Use (Rust) | Puppeteer (Node) | Selenium (Java) |
| :--- | :--- | :--- | :--- |
| **Startup 时间** | **< 50ms** | ~800ms | ~2500ms |
| **内存 Footprint** | **15 MB** | 100 MB+ | 200 MB+ |
| **DOM 提取** | **Zero-复制** | JSON Serialize | Slow Bridge |

## 能力 & Tools

### Vision & 提取
- **vision_map**: 返回 a screenshot overlay with numbered bounding boxes for all interactive elements.
- **快照**: Capture the raw HTML 快照 (YAML/Markdown optimized for AI).
- **screenshot**: Capture a visual 图像 of the page.
- **提取**: 获取 structured data from the DOM.
- **markdown**: 转换 the current page content 迁移到 Markdown.
- **sitemap**: 分析 景 structure via robots.txt, sitemaps, and page semantic analysis.

### Navigation & Lifecycle
- **navigate**: Visit a specific URL.
- **go_back** /go_forwardard**: Traverse browser history.
- **wait**: Pause execution or wait for specific conditions.
- **new_tab**: Open a new browser tab.
- **switch_tab**: Switch focus 迁移到 a specific tab.
- **close_tab**: Close the current or specified tab.
- **tab_list**: 列表 all open tabs.
- **close**: Terminate the browser session.

### Interaction
- **click**: Click elements via CSS selectors or DOM indices.
- **输入**: 类型 text into fields.
- **press_key**: 发送 specific keyboard events.
- **hover**: Hover over elements.
- **scroll**: Scroll the 视口.
- **select**: Choose 选项 in dropdowns.

### State & Debugging
- **cookies**: 管理 session cookies (获取/集合).
- **local_storage**: 管理 local 存储 data.
- **调试**: Access console logs and 调试 information.

## 用法

This skill is specialized for complex web interactions that 需要 maintaining state (like being logged in), handling dynamic JavaScript content, or managing multiple pages simultaneously. It offers higher 性能 and 控制 compared 迁移到 standard fetch-based tools.