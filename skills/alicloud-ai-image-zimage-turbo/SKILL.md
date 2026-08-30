---
name: alicloud-ai-image-zimage-turbo
description: 通过 DashScope multimodal-generation API 使用阿里云 Model Studio Z-Image Turbo (z-image-turbo) 生成图像。在创建文生图输出、控制 size/seed/prompt_extend 或记录 Z-Image 的请求/响应映射时使用。
tags:
- AI
- API
- 云服务
- 阿里云
---

Category: provider

# Model Studio Z-Image Turbo

使用 Z-Image Turbo 通过 DashScope multimodal-generation API 进行快速的文生图生成。

## 关键模型名称

仅使用以下精确的模型字符串：
- `z-image-turbo`

## 前置条件

- 在环境中设置 `DASHSCOPE_API_KEY`，或将 `dashscope_api_key` 添加到 `~/.alibabacloud/credentials`（环境变量优先）。
- 选择区域端点（北京或新加坡）。如果不确定，选择最合理的区域或询问用户。

## 规范化接口 (image.generate)

### 请求
- `prompt` (string, required)
- `size` (string, optional) 例如 `1024*1024`
- `seed` (int, optional)
- `prompt_extend` (bool, optional; 默认 false)
- `base_url` (string, optional) 覆盖 API 端点

### 响应
- `image_url` (string)
- `width` (int)
- `height` (int)
- `prompt` (string)
- `rewritten_prompt` (string, optional)
- `reasoning` (string, optional)
- `request_id` (string)

## 快速开始 (curl)

```bash
curl -sS 'https://dashscope.aliyuncs.com/api/v1/services/aigc/multimodal-generation/generation' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $DASHSCOPE_API_KEY" \
  -d '{
    "model": "z-image-turbo",
    "input": {
      "messages": [
        {
          "role": "user",
          "content": [{"text": "A calm lake at dawn, a lone angler casting a line, cinematic lighting"}]
        }
      ]
    },
    "parameters": {
      "size": "1024*1024",
      "prompt_extend": false
    }
  }'
```

## 本地辅助脚本

```bash
python skills/ai/image/alicloud-ai-image-zimage-turbo/scripts/generate_image.py \
  --request '{"prompt":"a fishing scene at dawn, cinematic, realistic","size":"1024*1024"}' \
  --output output/ai-image-zimage-turbo/images/fishing.png \
  --print-response
```

## Size 说明

- 总像素数必须在 `512*512` 和 `2048*2048` 之间。
- 优先使用常见尺寸如 `1024*1024`、`1280*720`、`1536*864`。

## 费用说明

- `prompt_extend=true` 的计费高于 `false`。仅在需要重写 prompt 时启用。

## 输出位置

- 默认输出：`output/ai-image-zimage-turbo/images/`
- 使用 `OUTPUT_DIR` 覆盖基础目录。

## 参考

- `references/api_reference.md` 包含请求/响应 schema 和区域端点。
- `references/sources.md` 包含官方文档。
