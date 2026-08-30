---
name: alicloud-ai-image-qwen-image-edit
description: 使用阿里云 Model Studio Qwen Image Edit Max (qwen-image-edit-max) 编辑图像。在修改现有图像（inpaint、替换、风格迁移、局部编辑）、保持主体一致性或记录图像编辑请求/响应映射时使用。
tags:
- AI
- 云服务
- 阿里云
---

Category: provider

# Model Studio Qwen Image Edit

使用 Qwen Image Edit 模型进行基于指令的图像编辑，而非文生图生成。

## 关键模型名称

使用以下精确的模型字符串之一：
- `qwen-image-edit-max`
- `qwen-image-edit-max-2026-01-16`

## 前置条件

- 在虚拟环境中安装 SDK：

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install dashscope
```
- 在环境中设置 `DASHSCOPE_API_KEY`，或将 `dashscope_api_key` 添加到 `~/.alibabacloud/credentials`。

## 规范化接口 (image.edit)

### 请求
- `prompt` (string, required)
- `image` (string | bytes, required) 源图像 URL/路径/字节
- `mask` (string | bytes, optional) inpaint 区域遮罩
- `size` (string, optional) 例如 `1024*1024`
- `seed` (int, optional)

### 响应
- `image_url` (string)
- `seed` (int)
- `request_id` (string)

## 操作指导

- 保持 prompt 面向任务：描述要更改的内容和要保留的内容。
- 使用 mask 进行确定性的局部编辑。
- 将输出资源保存到对象存储，仅持久化 URL。
- 对于主体一致性，在 prompt 中提供显式约束。

## 本地辅助脚本

准备规范化请求 JSON 并验证响应 schema：

```bash
.venv/bin/python skills/ai/image/alicloud-ai-image-qwen-image-edit/scripts/prepare_edit_request.py \
  --prompt "Replace the sky with sunset, keep buildings unchanged" \
  --image "https://example.com/input.png"
```

## 输出位置

- 默认输出：`output/ai-image-qwen-image-edit/images/`
- 使用 `OUTPUT_DIR` 覆盖基础目录。

## 参考

- `references/sources.md`
