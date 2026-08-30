---
name: image-ocr
description: 使用 Tesseract OCR 从图像中提取文本
metadata:
  openclaw:
    emoji: 👁️
    requires:
      bins:
      - tesseract
    install:
    - id: dnf
      kind: dnf
      package: tesseract
      bins:
      - tesseract
      label: Install via dnf
tags:
- 图像处理
---

# Image OCR

使用 Tesseract OCR 从图像中提取文本。支持多种语言和图像格式，包括 PNG、JPEG、TIFF 和 BMP。

## 命令

```bash
# 从图像中提取文本（默认：英语）
image-ocr "screenshot.png"

# 使用特定语言提取文本
image-ocr "document.jpg" --lang eng
```

## 安装

```bash
sudo dnf install tesseract
```
