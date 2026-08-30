---
name: gemini-image-remix
description: 使用 Gemini 进行高级图像生成和混音。支持 Gemini 2.5 Flash Image（默认）和 Gemini 3.0 Pro (Nano Banana Pro) 等模型。
metadata:
  openclaw:
    emoji: 🎨
    requires:
      bins:
      - uv
      env:
      - GEMINI_API_KEY
    primaryEnv: GEMINI_API_KEY
    install:
    - id: uv-brew
      kind: brew
      formula: uv
      bins:
      - uv
      label: Install uv (brew)
tags:
- 图像生成
---

# Gemini Image Remix

一个用于文本到图像生成和复杂图像到图像混音的多功能工具。默认情况下，它使用 **Gemini 2.5 Flash Image** 实现快速、高质量的结果。它还支持 **Gemini 3.0 Pro (Nano Banana Pro)** 等旗舰模型，用于高级艺术任务。

## 生成图像

从文本提示创建令人惊叹的视觉效果。

```bash
uv run {baseDir}/scripts/remix.py --prompt "a cybernetic owl in a neon forest" --filename "owl.png"
```

## 混音/修改图像

使用一个或多个参考图像来指导生成。非常适合风格迁移、背景更改或角色修改。

```bash
uv run {baseDir}/scripts/remix.py --prompt "change the art style to a pencil sketch" --filename "sketch.png" -i "original.png"
```

## 多图像合成

将多达 14 张不同图像的元素组合成一个连贯的场景。

```bash
uv run {baseDir}/scripts/remix.py --prompt "place the character from image 1 into the environment of image 2" --filename "result.png" -i "character.png" -i "env.png"
```

## 高级模型选择

切换到 **Nano Banana Pro** 等高级模型以进行高保真工作。

```bash
uv run {baseDir}/scripts/remix.py --model "gemini-3-pro-image-preview" --prompt "highly detailed oil painting of a dragon" --filename "dragon.png"
```

## 选项

- `--prompt`, `-p`: 图像描述或具体的编辑说明。
- `--filename`, `-f`: 生成的 PNG 的输出路径。
- `--input-image`, `-i`: 输入图像的路径（最多可重复 14 次）。
- `--resolution`, `-r`: `1K`（默认）、`2K` 或 `4K`。
- `--aspect-ratio`, `-a`: 输出宽高比（例如，`1:1`、`16:9`、`9:16`、`4:3`、`3:4`）。
- `--model`, `-m`: 要使用的模型（默认为 `gemini-2.5-flash-image`）。支持：`gemini-2.5-flash-image`、`gemini-3-pro-image-preview`。
- `--api-key`, `-k`: Gemini API key（默认为 `GEMINI_API_KEY` 环境变量）。
