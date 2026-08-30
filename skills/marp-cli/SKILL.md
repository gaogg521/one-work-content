---
name: marp-cli
description: 通过 CLI 将 Markdown 转换为演示文稿。输出 HTML, PDF, PowerPoint (PPTX), 和图像 (PNG/JPEG)。
homepage: https://github.com/marp-team/marp-cli
metadata:
  openclaw:
    emoji: 📽️
    requires:
      anyBins:
      - marp
tags:
- 文档
- CLI
- 前端
---

# Marp CLI

通过 CLI 将 Markdown 转换为演示文稿。输出 HTML, PDF, PowerPoint (PPTX), 和图像 (PNG/JPEG)。

**浏览器要求：** 标记为 🌐 的转换需要系统上安装兼容的浏览器（Chrome, Edge, 或 Firefox）。

## 快速开始

```bash
# 转换为 HTML
marp slide-deck.md

# 转换为 PDF（需要浏览器）
marp --pdf slide-deck.md

# 转换为 PowerPoint
marp --pptx slide-deck.md

# 转换为图像
marp --images png slide-deck.md
```

📖 **详细指南：** [QUICKSTART.md](QUICKSTART.md)

## 格式转换

### HTML
```bash
marp slide-deck.md
marp slide-deck.md -o output.html
```

### PDF 🌐
```bash
marp --pdf slide-deck.md
marp slide-deck.md -o output.pdf

# 带 PDF 大纲
marp --pdf --pdf-outlines slide-deck.md

# 在左下角包含演示者备注作为注释
marp --pdf --pdf-notes slide-deck.md
```

### PowerPoint (PPTX) 🌐
```bash
marp --pptx slide-deck.md
marp slide-deck.md -o output.pptx

# 可编辑 PPTX（实验性，需要 LibreOffice Impress）
marp --pptx --pptx-editable slide-deck.md
```

### 图像 🌐
```bash
# 多个图像
marp --images png slide-deck.md
marp --images jpeg slide-deck.md

# 仅标题幻灯片图像
marp --image png slide-deck.md
marp slide-deck.md -o output.png

# 高分辨率（缩放因子）
marp slide-deck.md -o title.png --image-scale 2
```

### 演示者备注
```bash
marp --notes slide-deck.md
marp slide-deck.md -o output.txt
```

## 监视模式

```bash
# 监视文件并在更改时自动转换
marp -w slide-deck.md

# 带浏览器预览的监视
marp -w -p slide-deck.md
```

## 服务器模式

```bash
# 使用按需转换服务目录
marp -s ./slides

# 通过环境变量指定端口
PORT=5000 marp -s ./slides

# 通过查询字符串访问转换后的格式
# http://localhost:8080/deck.md?pdf
# http://localhost:8080/deck.md?pptx
```

## 预览窗口

```bash
# 打开预览窗口（自动启用监视模式）
marp -p slide-deck.md

# 带 PDF 输出的预览
marp -p --pdf slide-deck.md
```

## 多个文件

```bash
# 转换多个文件
marp slide1.md slide2.md slide3.md

# 转换目录
marp ./slides/

# 使用 glob 模式
marp **/*.md

# 使用并行处理转换（默认：5 个并发）
marp -P 10 ./*.md

# 禁用并行处理
marp --no-parallel ./*.md
```

## 选项

| 选项 | 描述 |
|--------|-------------|
| `-o, --output <path>` | 输出文件路径 |
| `-w, --watch` | 监视模式 - 更改时自动转换 |
| `-s, --server <dir>` | 服务器模式 - 服务目录 |
| `-p, --preview` | 打开预览窗口 |
| `--pdf` | 转换为 PDF（需要 Chrome/Edge/Firefox） |
| `--pptx` | 转换为 PowerPoint PPTX（需要浏览器） |
| `--pptx-editable` | 生成可编辑 PPTX（实验性） |
| `--images [png\|jpeg]` | 转换为多个图像 |
| `--image` | 将标题幻灯片转换为单个图像 |
| `--image-scale <factor>` | 图像的缩放因子 |
| `--notes` | 将演示者备注导出为 TXT |
| `--pdf-notes` | 添加 PDF 备注注释 |
| `--pdf-outlines` | 添加 PDF 大纲/书签 |
| `--allow-local-files` | 允许访问本地文件（安全说明） |
| `--browser <chrome\|edge\|firefox>` | 选择用于转换的浏览器 |
| `--browser-path <path>` | 指定浏览器可执行路径 |
| `-P, --parallel <num>` | 并行转换计数 |
| `--no-parallel` | 禁用并行转换 |
| `--template <name>` | HTML 模板（默认: bespoke） |

## 常见模式

```bash
# 编辑时监视和预览
marp -w -p deck.md

# 服务幻灯片目录
marp -s ./presentations

# 将所有幻灯片转换为 PDF
marp --pdf *.md

# 从标题创建 OG 图像
marp deck.md -o og.png --image-scale 3

# 导出演示者备注
marp --notes deck.md
```

## 文档

| 文档 | 描述 |
|----------|-------------|
| [QUICKSTART.md](QUICKSTART.md) | 快速入门指南 |
| [EXAMPLES.md](EXAMPLES.md) | 详细示例 |
| [README.md](README.md) | 项目概述 |
| 官方文档 | https://github.com/marp-team/marp-cli |
```
