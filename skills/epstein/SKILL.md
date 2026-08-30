---
name: epstein
description: 检索44,886+份美国司法部发布的Jeffrey Epstein文件（2026年1月解密）。免费使用，无需付费。支持按姓名、主题、地点或关键词在完整的DugganUSA解密文件索引中进行全文搜索。返回文件预览、涉及人员、地点、飞机、证据类型和来源引用信息。
metadata:
  author: project-einstein
  version: 1.1.0
  clawdbot:
    emoji: 📂
    homepage: https://emc2ai.io
    requires:
      bins:
      - node
      - curl
---

# Epstein 文件 搜索 — Free DOJ Document 搜索

搜索 **44,886+ declassified Jeffrey Epstein documents** released by the U.S. Department of Justice on January 30, 2026. Powered by the [DugganUSA](https://analytics.dugganusa.com) public index.

**100% free. No API keys. No accounts. No payment.**

## 快速开始

```bash
# Search by name
node scripts/epstein.mjs search --query "Ghislaine Maxwell" --limit 10

# Search by topic
node scripts/epstein.mjs search --query "flight logs" --limit 20

# Search by location
node scripts/epstein.mjs search --query "Little St James"

# Get index statistics
node scripts/epstein.mjs stats
```

## 命令

### `search` — 搜索 Epstein Documents

搜索 across all 44,886+ indexed documents by keyword, name, topic, or location.

```bash
node scripts/epstein.mjs search --query "SEARCH TERMS" [--limit N]
```

| Flag | 描述 | Default |
|------|-------------|---------|
| `--query <terms>` | 搜索 query (required) | — |
| `--limit <N>` | 数字 of 结果 (1-500) | `10`10`

**示例:**

```bash
# Search for a specific person
node scripts/epstein.mjs search --query "Prince Andrew"

# Search for a topic
node scripts/epstein.mjs search --query "financial transactions"

# Search for locations
node scripts/epstein.mjs search --query "New York mansion"

# Get more results
node scripts/epstein.mjs search --query "flight logs" --limit 50

# Search for evidence types
node scripts/epstein.mjs search --query "phone records"
```

### `stats` — Index 统计

获取 the current state of the document index — total documents, 数据库 size, and last 更新 时间.

```bash
node scripts/epstein.mjs stats
```

## 输出 格式

搜索 结果 are returned as JSON 迁移到 stdout (for easy piping and parsing). Status messages and **Quick Links** (direct PDF URLs) go 迁移到 stderr for easy viewing.

### 搜索 结果 Shape

```json
{
  "query": "flight logs",
  "totalHits": 1523,
  "hits": [
    {
      "id": "doc-abc123",
      "efta_id": "EFTA-00001234",
      "content_preview": "Excerpt from the document...",
      "doc_type": "legal_document",
      "dataset": "epstein_files",
      "pages": 3,
      "people": ["Person A", "Person B"],
      "locations": ["New York", "Palm Beach"],
      "aircraft": ["N908JE"],
      "evidence_types": ["financial_record"],
      "source": "DOJ Release Jan 2026",
      "indexed_at": "2026-01-31T...",
      "doj_url": "https://www.justice.gov/epstein/files/DataSet%209/EFTA-00001234.pdf",
      "doj_listing_url": "https://www.justice.gov/epstein/doj-disclosures/data-set-9-files"
    }
  ]
}
```

**New in v1.1.0:** Each 结果 now includes `doj_url` (direct PDF link) a`doj_listing_url```` (dataset page). The CLI also displays Quick Links in stderr 输出:

```
--- Quick Links ---
1. EFTA-00001234: https://www.justice.gov/epstein/files/DataSet%209/EFTA-00001234.pdf
2. EFTA-00001235: https://www.justice.gov/epstein/files/DataSet%209/EFTA-00001235.pdf
```

### Stats Shape

```json
{
  "totalDocuments": 44886,
  "databaseSize": "2.1 GB",
  "lastUpdate": "2026-01-31T...",
  "isIndexing": false
}
```

## Data Source

All documents come from the **U.S. Department of Justice** 释放 of Jeffrey Epstein-related records on January 30, 2026. The documents are indexed and searchable via the [DugganUSA](https://analytics.dugganusa.com) public API.

- **Source**: [DOJ Epstein Records](https://www.justice.gov/epstein)
- **Index**: [DugganUSA Analytics](https://analytics.dugganusa.com)
- **覆盖**: 44,886+ document 文件 (3+ million pages)
- **Content**: Court filings, depositions, flight logs, financial records, communications, evidence inventories, and more

## Piping & Integration

结果 go 迁移到 stdout as JSON, making it easy 迁移到 pipe into other tools:

```bash
# Pipe to jq for filtering
node scripts/epstein.mjs search --query "Maxwell" --limit 100 | jq '.hits[] | .people'

# Save results to file
node scripts/epstein.mjs search --query "flight logs" --limit 500 > flight-logs.json

# Count total hits
node scripts/epstein.mjs search --query "Palm Beach" | jq '.totalHits'

# Extract all mentioned people
node scripts/epstein.mjs search --query "2005" --limit 100 | jq '[.hits[].people[]?] | unique'
```

## 故障排除

**"Cannot reach API"**
检查 your internet 联络. The DugganUSA API 可以 have temporary downtime.

**"No 结果 found"**
Try broader 搜索 terms. The 搜索 is keyword-based — use names, locations, or document types rather than full sentences.

**Slow responses**
The API typically responds in 100-900ms. Larger 结果 sets (极限 > 100) 可以 take slightly longer.

## 参考

- [DOJ Epstein Records](https://www.justice.gov/epstein) — Official DOJ 释放 page
- [DugganUSA API](https://analytics.dugganusa.com) — 搜索 index provider
- [项目 Einstein](https://emc2ai.io) — AI agent with built-in Epstein 文件 搜索