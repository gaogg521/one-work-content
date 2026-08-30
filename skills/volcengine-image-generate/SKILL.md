---
name: volcengine-image-generate
description: 使用 volcengine image_generate.py 脚本生成图像，需要提供清晰明确的 `prompt`。
license: 完整条款见 LICENSE.txt
tags:
- 云服务
- 图像生成
---

# 图像生成

## 使用场景

当你需要基于文本描述生成图像时，使用此技能调用 `image_generate` 函数。

## 步骤

1. 准备清晰明确的 `prompt`。
2. 运行脚本 `python scripts/image_generate.py "<prompt>"`。运行前，请先切换到对应目录。
3. 脚本将返回生成的图像 URL。

## 认证和凭据

- 首先，它会尝试读取 `MODEL_IMAGE_API_KEY` 或 `ARK_API_KEY` 环境变量。
- 如果未配置，它会尝试使用 `VOLCENGINE_ACCESS_KEY` 和 `VOLCENGINE_SECRET_KEY` 获取 Ark API Key。

## 输出格式

- 控制台将输出生成的图像 URL。
- 如果调用失败，将打印错误信息。

## 示例

```bash
python scripts/image_generate.py "a cute cat"
```
