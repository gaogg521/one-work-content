---
name: clawhub-summarize
description: 使用 summarize CLI 对 URLs 或文件进行摘要 (web, PDFs, images, audio, YouTube)。
---

# Summarize

快速对 URLs、本地文件和 YouTube 链接进行摘要的 CLI。

## 快速开始

```bash
summarize "https://example.com" --model google/gemini-3-flash-preview
summarize "/path/to/file.pdf" --model google/gemini-3-flash-preview
summarize "https://youtu.be/dQw4w9WgXcQ" --youtube auto
```

## Model + keys

为你选择的 provider 设置 API key:
- OpenAI: `OPENAI_API_KEY`
- Anthropic: `ANTHROPIC_API_KEY`
- xAI: `XAI_API_KEY`
- Google: `GEMINI_API_KEY` (别名: `GOOGLE_GENERATIVE_AI_API_KEY`, `GOOGLE_API_KEY`)

如果没有设置，默认 model 是 `google/gemini-3-flash-preview`。

## 有用的 flags

- `--length short|medium|long|xl|xxl|<chars>`
- `--max-output-tokens <count>`
- `--extract-only` (仅 URLs)
- `--json` (机器可读)
- `--firecrawl auto|off|always` (fallback extraction)
- `--youtube auto` (如果设置了 `APIFY_API_TOKEN`，则使用 Apify fallback)

## 配置

可选配置文件: `~/.summarize/config.json`

```json
{ "model": "openai/gpt-5.2" }
```

可选服务:
- `FIRECRAWL_API_KEY` 用于被拦截的站点
- `APIFY_API_TOKEN` 用于 YouTube fallback
