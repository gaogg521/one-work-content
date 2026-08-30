---
name: kusa-image
description: 使用 Kusa.pics API 生成图像，支持自定义风格、宽度和高度
tags:
- API
---

# Kusa.pics 图像生成器

使用 Kusa.pics API 生成图像。

## 配置
- API Key: 设置 `KUSA_API_KEY` 环境变量。

## 用法
```bash
export KUSA_API_KEY="your_api_key_here"
node skills/kusa-image/index.js "Your prompt here" [--style <id>] [--width <w>] [--height <h>]
```

## 选项
- `--style`: Style ID (默认: 6)
- `--width`: 宽度 (默认: 960)
- `--height`: 高度 (默认: 1680)
