---
name: nvidia-image-gen
version: 1.0.0
description: 使用 NVIDIA FLUX 模型生成和编辑图像。当用户要求生成图像、创建图片、编辑照片或使用 AI 修改现有图像时使用。支持文本到图像生成和带有文本提示的图像编辑。
tags:
- AI
- ArgoCD
---

# NVIDIA 图像生成

使用 NVIDIA 的 FLUX 模型生成和编辑图像。

## 模型

| 模型 | 用例 | 速度 | 质量 |
|-------|----------|-------|---------|
| `dev` | 高质量文本到图像 | 正常 | 最佳 |
| `schnell` | 快速文本到图像 | 快速 | 良好 |
| `kontext` | 图像编辑 | 正常 | 最佳 |

## 快速开始

```bash
# 生成图像
python scripts/generate.py "A cute cat in space"

# 编辑现有图像
python scripts/generate.py "Add sunglasses" -i photo.jpg -o edited.png
```

## 参数

### 文本到图像 (dev/schnell)

| 参数 | 短选项 | 默认值 | 描述 |
|-----------|-------|---------|-------------|
| `prompt` | | (必需) | 文本描述 |
| `-o, --output` | | output.png | 输出文件路径 |
| `--width` | | 1024 | 输出宽度（像素） |
| `--height` | | 1024 | 输出高度（像素） |
| `--aspect-ratio` | `-ar` | 1:1 | 宽高比预设 |
| `--steps` | `-s` | 30 | 扩散步数 |
| `--seed` | | 0 | 随机种子 (0=随机) |
| `--model` | `-m` | auto | 模型选择 |

### 图像编辑 (kontext)

| 参数 | 短选项 | 默认值 | 描述 |
|-----------|-------|---------|-------------|
| `prompt` | | (必需) | 编辑指令 |
| `-i, --input` | | (必需) | 输入图像路径 |
| `-o, --output` | | output.png | 输出文件路径 |
| `--steps` | `-s` | 30 | 扩散步数 |
| `--cfg` | | 3.5 | 引导比例 |
| `--seed` | | 0 | 随机种子 |

## 支持的宽高比

| 比例 | 分辨率 |
|-------|------------|
| 1:1 | 1024×1024 |
| 16:9 | 1344×768 |
| 9:16 | 768×1344 |
| 4:3 | 1216×832 |
| 3:4 | 832×1216 |

## 示例

### 基本生成
```bash
python scripts/generate.py "A mountain landscape at sunset"
```

### 宽屏格式 (16:9)
```bash
python scripts/generate.py "A panoramic beach view" -ar 16:9
```

### 竖屏模式 (9:16)
```bash
python scripts/generate.py "A professional headshot" -ar 9:16
```

### 自定义尺寸
```bash
python scripts/generate.py "A banner image" --width 1344 --height 768
```

### 快速生成
```bash
python scripts/generate.py "Quick sketch of a robot" -m schnell
```

### 编辑图像
```bash
python scripts/generate.py "Make the background a sunset" -i input.jpg -o output.png
```

### 可复现的结果
```bash
python scripts/generate.py "A robot" --seed 12345
```

## 输出

脚本输出 `MEDIA:/path/to/image.png`，可以直接发送到聊天。

## API 密钥

API 密钥嵌入在脚本中。要使用不同的密钥，请设置 `NVIDIA_API_KEY` 环境变量。
