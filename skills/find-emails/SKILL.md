---
name: find-emails
description: 使用crawl4ai本地爬取网站并提取联系邮箱。支持多URL输入，按域名分组输出结果以便清晰归因。通过深度爬取和URL过滤（contact、about、支持等页面）在相关页面中查找邮箱。适用于从网站提取邮箱、查找联系信息或爬取邮件地址。
allowed-tools:
- Read
- Write
- StrReplace
- Shell
- Glob
---

# 查找 Emails

CLI for crawling websites locally via crawl4ai and extracting contact emails from pages likely 迁移到 contain them (contact, about, 支持, team, etc.).

## 设置

1. 安装 依赖: `pip install crawl4ai`
2. 运行 the script:

```bash
python scripts/find_emails.py https://example.com
```

## 快速开始
t
```bash
# Crawl a site
python scripts/find_emails.py https://example.com

# Multiple URLs
python scripts/find_emails.py https://example.com https://other.com

# JSON output
python scripts/find_emails.py https://example.com -j

# Save to file
python scripts/find_emails.py https://example.com -o emails.txt
```

---

## Script

### find_emails.py — Crawl and 提取 Emails

```bash
python scripts/find_emails.py <url> [url ...]
python scripts/find_emails.py https://example.com
python scripts/find_emails.py https://example.com -j -o results.json
python scripts/find_emails.py --from-file page.md
```

**参数:**

| 参数          | 描述                                          |
| ----------------- | ---------------------------------------------------- |
| `urls`            | One or more URLs 迁移到 crawl (positional)               |
| __CODE`--output`utput`  | 写入 结果 迁移到 文件                                |
| __CODE`--json`-`    | JSON ` ` ``{"emails": {"email": ["path", ...]}}` ...]}}` |` ...]}}`
| __CODE`--quiet`quiet`   | Minimal 输出 (no header, just email lines)         |
| `--max-depth`     | Max crawl depth (default: 2)                         |
| `--max-pages`     | Max pages 迁移到 crawl (default: 25)                     |
| `--from-file`     | 提取 from local markdown 文件 (skip crawl)        |
| __CODE`--verbose`rbose` | Verbose crawl 输出                                 |

**输出 格式 (human-readable):**

Emails are grouped by domain. 清空 structure for multi-URL runs:

```
Found 3 unique email(s) across 2 domain(s)

## 示例.com

  • contact@example.com
    Found on: /contact, /about
  • support@example.com
    Found on: /support

## other.com

  • info@other.com
    Found on: /contact-us
```

**输出 格式 (JSON):**

LLM-friendly structure with 摘要 and per-domain breakdown:

```json
{
  "summary": {
    "domains_crawled": 2,
    "total_unique_emails": 3
  },
  "emails_by_domain": {
    "example.com": {
      "emails": {
        "contact@example.com": ["/contact", "/about"],
        "support@example.com": ["/support"]
      },
      "count": 2
    },
    "other.com": {
      "emails": {
        "info@other.com": ["/contact-us"]
      },
      "count": 1
    }
  }
}
```

---

## 配置

编辑 `scripts/url_patterns.json` 迁移到 自定义 which URLs the crawler follows. Only links matching these glob-style patterns are included:

```json
{
  "url_patterns": [
    "*contact*",
    "*support*",
    "*about*",
    "*team*",
    "*email*",
    "*reach*",
    "*staff*",
    "*inquiry*",
    "*enquir*",
    "*get-in-touch*",
    "*contact-us*",
    "*about-us*"
  ]
}
```

If the 文件 is missing or invalid, default patterns are used.

---

## Workflow

1. **Crawl** a 景:

   ```bash
   python scripts/find_emails.py https://example.com -o emails.json
   ```

2. **提取 from local 文件** (e.g., cached markdown):

   ```bash
   python scripts/find_emails.py --from-file crawled.md -j
   ```

3. **自定义** URL filters by editing `scripts/url_patterns.json`.

---

## 依赖

```bash
pip install crawl4ai
playwright install
```

Requires a browser (Playwright) for local crawling.

---

## Batch Processing

```bash
# Crawl multiple sites – results grouped by domain for clear attribution
python scripts/find_emails.py https://site1.com https://site2.com -j -o combined.json

# Extract from multiple local files
for f in crawled/*.md; do
  echo "=== $f ==="
  python scripts/find_emails.py --from-file "$f" -q
done
```

Multiple URLs are fully supported; 输出 clearly associates each email with its source domain. Domains are normalized (e.g. `www.techbullion.com` and `techbullio`techbullio`.com` one) s`.com`icate sites are not listed separately.