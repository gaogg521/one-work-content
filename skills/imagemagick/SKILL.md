---
name: imagemagick
description: 使用 ImageMagick 进行图像处理，支持移除背景、调整大小、格式转换、圆角、水印和批量缩略图生成
tags:
- 图像处理
---

# ImageMagick Moltbot Skill

Moltbot 中用于图像处理的全面 ImageMagick 操作。

## 安装

**macOS:**
```bash
brew install imagemagick
```

**Linux:**
```bash
sudo apt install imagemagick  # Debian/Ubuntu
sudo dnf install ImageMagick  # Fedora
```

**验证：**
```bash
convert --version
```

## 可用操作

### 1. 移除背景（白色/纯色 → 透明）
```bash
./scripts/remove-bg.sh input.png output.png [tolerance] [color]
```

| 参数 | 默认值 | 范围 | 描述 |
|-----------|---------|-------|-------------|
| input.png | — | — | 源图像 |
| output.png | — | — | 输出透明 PNG |
| tolerance | 20 | 0-255 | 颜色匹配模糊因子 |
| color | #FFFFFF | hex | 要移除的颜色 |

**示例：**
```bash
./scripts/remove-bg.sh icon.png icon-clean.png              # 默认白色
./scripts/remove-bg.sh icon.png icon-clean.png 30           # 宽松容差
./scripts/remove-bg.sh icon.png icon-clean.png 10 "#000000" # 移除黑色
```

### 2. 调整图像大小
```bash
convert input.png -resize 256x256 output.png
```

### 3. 转换格式
```bash
convert input.png output.webp          # PNG → WebP
convert input.jpg output.png           # JPG → PNG
convert input.png -quality 80 output.jpg  # 压缩
```

### 4. 圆角（iOS 风格）
```bash
convert input.png -alpha set -virtual pixel transparent \
    -distort viewport 512x512+0+0 \
    -channel A -blur 0x10 -threshold 50% \
    output-rounded.png
```

### 5. 添加水印
```bash
convert base.png watermark.png -gravity southeast -composite output.png
```

### 6. 批量缩略图生成
```bash
for f in *.png; do convert "$f" -resize 128x128 "thumbs/$f"; done
```

### 7. 颜色调整
```bash
convert input.png -brightness-contrast 10x0 output.png      # 更亮
convert input.png -grayscale output.png                     # 灰度
convert input.png -modulate 100,150,100 output.png          # 更高饱和度
```

## 常见模式

### 扁平图标 → 透明背景
```bash
./scripts/remove-bg.sh icon.png icon-clean.png 15
```

### 生成应用图标集（iOS）
```bash
for size in 1024 512 256 128 64 32 16; do
    convert icon.png -resize ${size}x${size} icon-${size}.png
done
```

### 网页优化
```bash
convert large.png -quality 85 -resize 2000x2000\> optimized.webp
```

## 提示

- **更高容差 (20-50):** 更适合抗锯齿边缘，可能会移除一些前景
- **更低容差 (5-15):** 保留细节，可能会留下颜色边缘
- **对于扁平图标:** 10-20 通常效果最好
- 使用 `-quality` 进行 JPEG/WebP 压缩 (0-100)
- 使用 `-strip` 移除元数据以减小文件大小
