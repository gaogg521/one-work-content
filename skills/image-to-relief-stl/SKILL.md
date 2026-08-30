---
name: image-to-relief-stl
description: 将源图像（或多色遮罩图像）转换为可3D打印的浅浮雕STL，通过将颜色（或灰度）映射到高度。当你从图像生成技能（如nano-banana-pro等）获得图像并希望获得一个真实的、可打印的模型（STL）时，通过确定性管道使用此技能。
metadata:
  openclaw:
    requires:
      bins:
      - python3
      - potrace
      - mkbitmap
    install:
    - id: apt
      kind: apt
      package: potrace
      bins:
      - potrace
      - mkbitmap
      label: Install potrace + mkbitmap (apt)
    - id: brew
      kind: brew
      formula: potrace
      bins:
      - potrace
      - mkbitmap
      label: Install potrace + mkbitmap (brew)
tags:
- 图像处理
- 图像生成
---

# image-to-relief-stl

通过将颜色（或灰度）映射到高度，从输入图像生成**水密、可打印的STL**。

这是一个对编排器友好的工作流：
- 使用**nano-banana-pro**（或任何图像模型）生成**纯色**图像。
- 运行此技能将其转换为**浅浮雕**模型。

## 实际约束（为了获得良好效果）

要求图像模型生成：
- **恰好N种纯色**（无渐变）
- **无阴影/无抗锯齿**
- 边缘清晰的大胆形状

这样分割才可靠。

## 快速开始（给定图像）

```bash
bash scripts/image_to_relief.sh input.png --out out.stl \
  --mode palette \
  --palette '#000000=3.0,#ffffff=0.0' \
  --base 1.5 \
  --pixel 0.4
```

### 灰度模式

```bash
bash scripts/image_to_relief.sh input.png --out out.stl \
  --mode grayscale \
  --min-height 0.0 \
  --max-height 3.0 \
  --base 1.5 \
  --pixel 0.4
```

## 输出

- `out.stl`（ASCII STL）
- 可选的`out-preview.svg`（通过potrace生成的矢量预览；尽力而为）

## 注意事项

- 此v0使用**栅格高度场**网格化方法（稳健，无重型CAD依赖）。
- `--pixel`参数控制分辨率（越小=细节越高，STL越大）。
