---
name: alicloud-ai-video-wan-r2v
description: 使用阿里云 Model Studio Wan R2V (wan2.6-r2v-flash) 生成基于参考的视频。在从参考视频/图像素材创建多镜头视频、保持角色风格或记录 reference-to-video 请求/响应流时使用。
tags:
- AI
- 云服务
- 阿里云
---

Category: provider

# Model Studio Wan R2V

使用 Wan R2V 进行参考到视频的生成。这与 i2v（单图生视频）不同。

## 关键模型名称

仅使用以下精确的模型字符串：
- `wan2.6-r2v-flash`

## 前置条件

- 在虚拟环境中安装 SDK：

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install dashscope
```
- 在环境中设置 `DASHSCOPE_API_KEY`，或将 `dashscope_api_key` 添加到 `~/.alibabacloud/credentials`。

## 规范化接口 (video.generate_reference)

### 请求
- `prompt` (string, required)
- `reference_video` (string | bytes, required)
- `reference_image` (string | bytes, optional)
- `duration` (number, optional)
- `fps` (number, optional)
- `size` (string, optional)
- `seed` (int, optional)

### 响应
- `video_url` (string)
- `task_id` (string, 异步时)
- `request_id` (string)

## 异步处理

- 生产流量优先使用异步提交。
- 以 15-20s 间隔轮询任务结果。
- 当返回 `SUCCEEDED` 或终端失败状态时停止轮询。

## 本地辅助脚本

准备规范化请求 JSON 并验证响应 schema：

```bash
.venv/bin/python skills/ai/video/alicloud-ai-video-wan-r2v/scripts/prepare_r2v_request.py \
  --prompt "Generate a short montage with consistent character style" \
  --reference-video "https://example.com/reference.mp4"
```

## 输出位置

- 默认输出：`output/ai-video-wan-r2v/videos/`
- 使用 `OUTPUT_DIR` 覆盖基础目录。

## 参考

- `references/sources.md`
