---
name: literature-review
version: 1.2.0
description: 通过 Semantic Scholar、OpenAlex、Crossref 和 PubMed API 搜索学术来源，协助撰写文献综述。当用户需要查找某个主题的论文、获取特定 DOI 的详情，或使用 proper citations 起草文献综述章节时使用。
tags:
- API
---

# Literature Review

使用多引擎搜索集成（S2、OA、CR、PM）帮助撰写学术文献综述。

## 能力

- **多来源搜索**：使用 Semantic Scholar (S2)、OpenAlex (OA)、Crossref (CR) 和 PubMed (PM) 查找相关学术论文。
- **完整摘要**：所有来源现在都返回完整摘要（PubMed 使用 `efetch` 获取完整 XML 记录）。
- **DOI 提取**：从所有来源提取 DOI 用于交叉引用和去重。
- **自动去重**：搜索多个来源时（`--source all` 或 `--source both`），结果按 DOI 自动去重。
- **礼貌访问**：自动为 OpenAlex/Crossref "Polite Pool" 提供 email 标识（通过 `USER_EMAIL` 环境变量）。
- **摘要重建**：从 OpenAlex inverted index 格式重建摘要。
- **综合**：按主题对论文进行分组，并根据元数据起草综述章节。

## 环境变量

| Variable | Purpose | Default |
|----------|---------|---------|
| `USER_EMAIL` | 礼貌 API 访问的 email | `anonymous@example.org` |
| `CLAWDBOT_EMAIL` | 如果 USER_EMAIL 未设置则作为 fallback | — |
| `SEMANTIC_SCHOLAR_API_KEY` | 可选的 S2 API key 以获得更高 rate limits | — |
| `OPENALEX_API_KEY` | 可选的 OpenAlex API key | — |

## 工作流

### 1. 广泛搜索（所有数据库）
从所有主要学术数据库获取全面概述。结果按 DOI 自动去重。
```bash
python3 scripts/lit_search.py search "impact of glycyrrhiza on bifidobacterium" --limit 5 --source all
```

### 2. 定向搜索
- **OpenAlex** (`oa`)：快速全面，摘要质量好。
- **Semantic Scholar** (`s2`)：高质量的 citation data 和 TL;DRs。
- **Crossref** (`cr`)：精确的基于 DOI 的元数据（无摘要）。
- **PubMed** (`pm`)：生物医学研究的黄金标准，完整的摘要和 PMIDs。

```bash
python3 scripts/lit_search.py search "prebiotic effects of liquorice" --source pm
```

### 3. 比较来源
同时搜索 S2 和 OA 以确保不遗漏任何内容。默认去重。
```bash
python3 scripts/lit_search.py search "Bifidobacterium infantis growth" --source both
```

### 4. 获取完整详情（S2）
检索包含 TL;DR 摘要的详细元数据。
```bash
python3 scripts/lit_search.py details "DOI:10.1016/j.foodchem.2023.136000"
```

### 5. 撰写综述
1. **提取**：从找到的摘要中提取关键发现。
2. **组织**：将发现分组为逻辑结构（例如按时间或主题）。
3. **起草**：使用 "Think step-by-step" 方法将多个来源综合成一个连贯的叙述。

## 输出格式

每个结果包含：
- `id`：Source-specific identifier（PubMed 的 PMID、OpenAlex ID、S2 paper ID、Crossref 的 DOI）
- `doi`：可用时的 DOI（用于去重）
- `title`：论文标题
- `year`：出版年份
- `authors`：作者姓名列表
- `abstract`：完整摘要文本（可用时）
- `venue`：期刊或会议名称
- `citationCount`：引用次数（S2、OA）
- `source`：结果来自哪个数据库

## 成功提示

- **Citations**：始终在 bibliography 中交叉引用 DOI 或 PMID 以确保准确性。
- **Filtering**：关注 `citationCount` 较高或近年份的论文，以获得更现代的综述。
- **PubMed for Medicine**：使用 `--source pm` 获取最可靠的生物医学文献。
- **Deduplication**：多来源搜索自动移除重复项；如果你需要原始计数，请使用单一来源。
