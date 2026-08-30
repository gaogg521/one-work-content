---
name: gemini-tg-image-gen
description: 通过 OpenRouter (google/gemini-2.5-flash-image) 生成图像并发送到 Telegram。当用户在 TG 中请求 AI 生成图像时使用。
tags:
- AI
---

# Gemini TG Image Gen (OpenRouter)

## 工作流

1. 立即在 Telegram 中通知用户：`"⏳ Идёт генерация, подождите немного..."`
2. 使用 OpenRouter 模型 `google/gemini-2.5-flash-image`。
3. 从环境变量读取 API key：`OPENROUTER_API_KEY`。
4. 运行脚本在本地生成并保存图像。
5. 使用 `message` 工具将图像发送到 Telegram（本地文件路径）。
6. NO_REPLY。

## 用法

```bash
OPENROUTER_API_KEY=... python3 scripts/generate_image.py "<prompt>"
```

脚本会打印一个带有 `paths` 的 JSON 对象。

## Telegram 发送

```
# step 1: waiting message
message action=send channel=telegram text="⏳ Идёт генерация, подождите немного..."

# step 5: send image
message action=send channel=telegram media="/root/.openclaw/workspace/tmp/openrouter_image_*.png" caption="Generated: <prompt>"
```

发送后，使用 `NO_REPLY`。
