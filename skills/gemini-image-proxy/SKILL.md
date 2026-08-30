---
name: gemini-image-proxy
version: 1.0.0
description: 使用 OpenAI Python SDK 通过 Gemini API 生成和编辑图像。
metadata: None
openclaw: None
emoji: 🎨
requires: None
env:
- GOOGLE_PROXY_API_KEY
- GOOGLE_PROXY_BASE_URL
tags:
- API
- Nginx
- Python
---

# Gemini Image Simple

使用 **Gemini 3 Pro Image** 通过 OpenAI Python SDK 和 OpenAI 兼容的 API 端点生成和编辑图像。

## 为什么使用此 Skill

| 特性                  | 此 Skill                | 其他 (nano-banana-pro 等) |
| ------------------------ | ------------------------- | ------------------------------ |
| **依赖**         | openai (SDK)              | google-genai, pillow, etc.     |
| **需要 pip/uv**      | ✅ Yes                    | ✅ Yes                         |
| **可在 Fly.io 免费版运行** | ✅ Yes (with pip)         | ❌ Fails                       |
| **可在容器中运行**  | ✅ Yes (with pip)         | ❌ Often fails                 |
| **图像生成**     | ✅ Full                   | ✅ Full                        |
| **图像编辑**        | ✅ Yes                    | ✅ Yes                         |
| **设置复杂度**     | Install SDK + set API key | Install packages first         |

**底线：** 此 skill 使用 OpenAI SDK，因此你必须先用 pip 安装 `openai`。

## 安装

```bash
python3 -m pip install openai
```

## 快速开始

```bash
# 设置环境变量
export GOOGLE_PROXY_API_KEY="your_api_key"
export GOOGLE_PROXY_BASE_URL="https://example.com/v1"

# 生成
python3 /data/clawd/skills/gemini-image-simple/scripts/generate.py "A cat wearing a tiny hat" cat.png

# 编辑现有图像
python3 /data/clawd/skills/gemini-image-simple/scripts/generate.py "Make it sunset lighting" edited.png --input original.png
```

## 用法

### 生成新图像

```bash
python3 {baseDir}/scripts/generate.py "your prompt" output.png
```

### 编辑现有图像

```bash
python3 {baseDir}/scripts/generate.py "edit instructions" output.png --input source.png
```

支持的输入格式：PNG, JPG, JPEG, GIF, WEBP

## 环境

设置这些环境变量：

- `GOOGLE_PROXY_API_KEY` (your API key)
- `GOOGLE_PROXY_BASE_URL` (OpenAI-compatible base URL, e.g. https://example.com/v1)

## 工作原理

使用 **Gemini 3 Pro Image** (`gemini-3-pro-image`) 通过 OpenAI Python SDK：

- `client.images.generate(...)` 用于生成新图像
- `client.images.edits(...)` 用于编辑
- 需要 `openai` 包

就是这样。在任何安装了 `openai` 的 Python 3.10+ 环境中均可运行。

## 模型

当前使用：`gemini-3-pro-image`

其他可用模型（如需可在 generate.py 中更改）：

- `gemini-3-pro-image-preview` - Preview variant
- `imagen-4.0-ultra-generate-001` - Imagen 4.0 Ultra
- `imagen-4.0-generate-001` - Imagen 4.0
- `gemini-2.5-flash-image` - Gemini 2.5 Flash with image gen

## 示例

```bash
# Landscape
python3 {baseDir}/scripts/generate.py "Misty mountains at sunrise, photorealistic" mountains.png

# Product shot
python3 {baseDir}/scripts/generate.py "Minimalist product photo of a coffee cup, white background" coffee.png

# Edit: change style
python3 {baseDir}/scripts/generate.py "Convert to watercolor painting style" watercolor.png --input photo.jpg

# Edit: add element
python3 {baseDir}/scripts/generate.py "Add a rainbow in the sky" rainbow.png --input landscape.png
```
