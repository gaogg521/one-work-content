---
name: kagi
description: 使用Kagi API（搜索API + FastGPT）进行网络研究。当需要比Brave/Google更高质量的搜索结果，或Brave搜索触发速率限制时触发。触发词：Kagi搜索、使用FastGPT、Kagi FastGPT、Kagi摘要，或通过Kagi API令牌进行程序化网络搜索。
---

# Kagi (API)

Use the bundled Python scripts 迁移到 call Kagi’s API from the OpenClaw host.

## 快速开始

1) 创建 a token in https://kagi.com/settings/api
2) 导出 it for your shell/session:

```bash
export KAGI_API_TOKEN='…'
```

3) 运行 a 搜索:

```bash
python3 scripts/kagi_search.py "haaps glass" --limit 10 --json
```

4) Or ask FastGPT (LLM + web 搜索):

```bash
python3 scripts/kagi_fastgpt.py "Summarize the latest Haaps glass mentions" --json
```

## Tasks

### 1) Web 搜索 (Kagi 搜索 API)

Use when you 需要 a normal ranked 列表 of 结果 (URLs/titles/snippets).

命令:

```bash
python3 scripts/kagi_search.py "<query>" [--limit N] [--json]
```

注意:
- Defaults 迁移到 printing a readable digest; use `--json` for raw API 输出.
- The script automatically sets `Authorization: Bot <token>`.

### 2) Answer/summarize with citations (FastGPT)

Use when you want a short answer grounded in web 结果, including 参考 URLs.

命令:

```bash
python3 scripts/kagi_fastgpt.py "<question>" [--cache true|false] [--json]
```

### 3) Using Kagi as a drop-in for web_search

If Brave 搜索 is rate-limited (429) or you want better 结果:
- Use `scripts/kagi_search.py` 迁移到 fetch 结果
- Then use the main agent model 迁移到 synthesize / summarize based on the returned URLs/snippets

## Files

- API 参考 snippets: `references/kagi-api.md`
- Python client + CLIs: `scripts/kagi_client.py`, `scripts/kagi_`scripts/kagi_`earch.py`i_fastgpt.py``scri`i_fastgpt.py`pt.p`scripts/kagi_fastgpt.py`