---
name: docling
description: 使用 docling CLI 和 GPU 加速从网页、PDF、文档（docx、pptx）和图像中提取和解析内容。当你需要干净、结构化的文本时，使用 docling 替代 web_fetch 从特定 URL 提取内容。使用 Brave（web_search）进行搜索/发现页面。当你拥有 URL 并需要解析其内容时使用 docling。
version: 1.0.2
metadata: None
requires: None
bins:
- docling
tags:
- Web
- 搜索
---

# Docling - 文档与网页内容提取

用于将文档和网页解析为干净、结构化文本的 CLI 工具。使用 GPU 加速进行 OCR 和 ML 模型。

## 先决条件

- 必须安装 `docling` CLI（例如，通过 `pipx install docling`）
- 对于 GPU 支持：具有 CUDA 驱动程序的 NVIDIA GPU

## 何时使用

- **从 URL 提取内容** → 使用 docling（而非 web_fetch）
- **搜索信息** → 使用 web_search（Brave）
- **解析 PDF、DOCX、PPTX** → 使用 docling
- **图像上的 OCR** → 使用 docling

## 快速命令

### 网页 → Markdown（默认）
```bash
docling "<URL>" --from html --to md
```
输出：在当前目录中创建一个 `.md` 文件（或使用 `--output`）

### 网页 → 纯文本
```bash
docling "<URL>" --from html --to text --output /tmp/docling_out
```

### 带 OCR 的 PDF
```bash
docling "/path/to/file.pdf" --ocr --device cuda --output /tmp/docling_out
```

## 关键选项

| 选项 | 值 | 描述 |
|--------|--------|-------------|
| `--from` | html, pdf, docx, pptx, image, md, csv, xlsx | 输入格式 |
| `--to` | md, text, json, yaml, html | 输出格式 |
| `--device` | auto, cuda, cpu | 加速器（默认：auto） |
| `--output` | path | 输出目录（推荐：使用受控临时目录） |
| `--ocr` | flag | 为图像/扫描的 PDF 启用 OCR |
| `--tables` | flag | 提取表格（默认：开启） |

## 安全说明

⚠️ **除非你信任来源，否则避免使用这些标志：**
- `--enable-remote-services` - 可以将数据发送到远程端点
- `--allow-external-plugins` - 加载第三方代码
- 带有不受信任值的自定义 `--headers` - 可以重定向请求

## 工作流

1. **对于网页内容提取**：使用 `docling "<URL>" --from html --to text --output /tmp/docling_out`
2. **从指定的输出目录读取输出文件**
3. **读取后清理** 输出目录

## GPU 支持

Docling 通过 CUDA（NVIDIA）支持 GPU 加速。验证 CUDA 是否可用：
```bash
python -c "import torch; print(torch.cuda.is_available())"
```

## 完整 CLI 参考

有关完整选项列表，请参阅 [references/cli-reference.md](references/cli-reference.md)。
