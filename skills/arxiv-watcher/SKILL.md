---
name: arxiv-watcher
description: 搜索和总结 ArXiv 论文。当用户需要获取最新研究、特定主题论文或每日 AI 论文摘要时使用。
tags:
- AI
---

# ArXiv Watcher

此技能与 ArXiv API 交互以查找和总结最新研究论文。

## 能力

- **搜索**：通过关键词、作者或类别查找论文。
- **总结**：获取摘要并提供简洁的总结。
- **保存到记忆**：自动将总结的论文记录到 `memory/RESEARCH_LOG.md` 以供长期追踪。
- **深入分析**：如果需要，在 PDF 链接上使用 `web_fetch` 以提取更多详情。

## 工作流

1. 使用 `scripts/search_arxiv.sh "<query>"` 获取 XML 结果。
2. 解析 XML（查找 `<entry>`、`<title>`、`<summary>` 和 `<link title="pdf">`）。
3. 向用户展示发现。
4. **强制**：将任何讨论过的论文的标题、作者、日期和摘要追加到 `memory/RESEARCH_LOG.md`。使用格式：
   ```markdown
   ### [YYYY-MM-DD] TITLE_OF_PAPER
   - **Authors**: Author List
   - **Link**: ArXiv Link
   - **Summary**: Brief summary of the paper and its relevance.
   ```

## 示例

- "Busca los últimos papers sobre LLM reasoning en ArXiv."
- "Dime de qué trata el paper con ID 2512.08769."
- "Hazme un resumen de las novedades de hoy en ArXiv sobre agentes."

## 资源

- `scripts/search_arxiv.sh`: 直接 API 访问脚本。
