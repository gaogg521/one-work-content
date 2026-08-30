---
name: vlmrun-cli-skill
description: 使用 VLM Run CLI (`vlmrun`) 与 Orion 视觉 AI 代理交互。使用自然语言处理图像、视频和文档。触发词：图像理解/生成、目标检测、OCR、视频摘要、文档提取、图像生成、视觉 AI 聊天、'生成图像/视频'、'分析此图像/视频'、'从中提取文本'、'总结此视频'、'处理此 PDF'。
tags:
- AI
---

# VLM Run CLI

通过 CLI 与 VLM Run 的 Orion 视觉 AI 代理聊天。

## 设置
```bash
uv venv && source .venv/bin/activate
uv pip install "vlmrun[cli]"
```

## 环境变量

你必须在环境中加载以下变量，以便 CLI 可以使用它们。你可以将 [./env](./env) 文件加载到你的环境中。

| 变量 | 类型 | 描述 |
|----------|------|-------------|
| `VLMRUN_API_KEY` | 必需 | 你的 VLM Run API 密钥（必需） |
| `VLMRUN_BASE_URL` | 可选 | 基础 URL（默认：`https://agent.vlm.run/v1`） |
| `VLMRUN_CACHE_DIR` | 可选 | 缓存目录（默认：`~/.vlmrun/cache/artifacts/`） |

## 命令

```bash
vlmrun chat "<prompt>" -i input.jpg [options]
```

## 选项

| 标志 | 描述 |
|------|-------------|
| `-p, --prompt` | 提示文本、文件路径或 `stdin` |
| `-i, --input` | 输入文件 - 图像、视频、文档（可重复） |
| `-o, --output` | 产物目录（默认：`~/.vlmrun/cache/artifacts/`） |
| `-m, --model` | `vlmrun-orion-1:fast`、`vlmrun-orion-1:auto`（默认）、`vlmrun-orion-1:pro` |
| `-s, --session` | 可选会话 ID 以继续之前的会话 |
| `-j, --json` | 原始 JSON 输出 |
| `-ns, --no-stream` | 禁用流式传输 |
| `-nd, --no-download` | 跳过产物下载 |

## 示例

### 图像
```bash
vlmrun chat "Describe what you see in this image in detail" -i photo.jpg
vlmrun chat "Detect and list all objects visible in this scene" -i scene.jpg
vlmrun chat "Extract all text and numbers from this document image" -i document.png
vlmrun chat "Compare these two images and describe the differences" -i before.jpg -i after.jpg
```

### 图像生成
```bash
vlmrun chat "Generate a photorealistic image of a cozy cabin in a snowy forest at sunset" -o ./generated
vlmrun chat "Remove the background from this product image and make it transparent" -i product.jpg -o ./output
```

### 视频
```bash
vlmrun chat "Summarize the key points discussed in this meeting video" -i meeting.mp4
vlmrun chat "Find the top 3 highlight moments and create short clips from them" -i sports.mp4
vlmrun chat "Transcribe this lecture with timestamps for each section" -i lecture.mp4 --json
```

### 视频生成
```bash
vlmrun chat "Generate a 5-second video of ocean waves crashing on a rocky beach at golden hour" -o ./videos
vlmrun chat "Create a smooth slow-motion video from this image" -i ocean.jpg -o ./output
```

### 文档
```bash
vlmrun chat "Extract the vendor name, line items, and total amount" -i invoice.pdf --json
vlmrun chat "Summarize the key terms and obligations in this contract" -i contract.pdf
```

### 提示来源
```bash
# 直接提示
vlmrun chat "What objects and people are visible in this image?" -i photo.jpg

# 从文件获取提示
vlmrun chat -p long_prompt.txt -i photo.jpg

# 从标准输入获取提示
echo "Describe this image in detail" | vlmrun chat - -i photo.jpg
```

### 继续之前的会话
如果你想保留过去的对话并将生成的产物保留在上下文中，可以使用 `-s` 标志通过开始会话时生成的会话 ID 继续之前的会话。

```bash
# 开始一个新的图像生成任务会话，其中生成一个新角色
vlmrun chat "Create an iconic scene of a ninja in a forest, practicing his skills with a katana?" -i photo.jpg

# 在上下文中使用之前的聊天会话以保留相同的角色和场景上下文（其中会话 ID 为 <session_id>）
vlmrun chat "Create a new scene with the same character meditating under a tree" -i photo.jpg -s <session_id>
```

### 跳过产物下载
如果你想跳过产物下载，可以使用 `-nd` 标志。
```bash
vlmrun chat "What objects and people are visible in this image?" -i photo.jpg -nd
```

## 注意事项

- 使用 `-o ./<directory>` 将生成的产物（图像、视频）保存到相对于当前工作目录的位置
- 不带 `-o` 时，产物保存到 `~/.vlmrun/cache/artifacts/<session_id>/`
- 多个输入文件并发上传
