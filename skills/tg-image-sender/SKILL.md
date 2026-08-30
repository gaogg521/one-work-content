---
name: tg-image-sender
description: 使用 message 工具配合 Picsum.photos URL 或自定义媒体，直接发送测试或生成的图片到 Telegram 聊天。当用户请求 'send photo'、'generate image here in TG' 或在 Telegram 中展示/测试图片时使用（例如 'пришли фото'、'покажи картинку'）。
tags:
- 即时通讯
---

# TG Image Sender

## 快速用法

直接调用 `message` 工具：

```
message action=send channel=telegram media="https://picsum.photos/800/600?random=1" caption="Test image 🦞"
```

- **Size**：调整宽度/高度，例如 `https://picsum.photos/400/300`
- **Seed**：`https://picsum.photos/800/600?random=1234` 用于可复现。
- **Real image**：替换为实际的 URL/媒体路径。
- **Caption**：可选描述。

## 示例

- 随机照片：`media="https://picsum.photos/800/600?random=1"`
- 特定照片：`media="https://picsum.photos/seed/cat/800/600"`

发送后，使用 `NO_REPLY` 以避免重复文本。

## 工作流

1. 匹配用户的 TG 图片请求。
2. 生成 Picsum URL 或使用提供的 URL。
3. 通过 `message` 工具发送。
4. NO_REPLY。

无需脚本——纯工具调用。