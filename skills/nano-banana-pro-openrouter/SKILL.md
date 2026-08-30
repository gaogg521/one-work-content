---
name: nano-banana-pro-openrouter
description: 通过 OpenRouter 使用 Nano Banana Pro 生成图像。当用户请求图像生成、提到 Nano Banana Pro、Gemini 3 Pro Image 或 OpenRouter 图像生成时使用。
tags:
- 图像生成
---

# Nano Banana Pro 图像生成

使用 OpenRouter 的 Nano Banana Pro (Gemini 3 Pro Image Preview) 生成新图像。

## 用法

使用绝对路径运行脚本（不要先 cd 到技能目录）：

生成新图像：
```bash
sh ~/.openclaw/workspace/skills/nano-banana-pro-openrouter/scripts/generate_image.sh --prompt "your image description" [--filename "output-name.png" | --filename auto] [--resolution 1K|2K|4K] [--api-key KEY]
```
注意：shell 版本目前仅支持生成（无输入图像编辑）。

重要：
- 图像始终保存在 `~/.openclaw/workspace/outputs/nano-banana-pro-openrouter`
- 如果 `--filename` 包含路径，则仅使用基本名称

## 默认工作流（草稿 -> 迭代 -> 最终）

目标：在提示正确之前，快速迭代而不在 4K 上浪费时间。

- 草稿 (1K): 快速反馈循环
  - `sh ~/.openclaw/workspace/skills/nano-banana-pro-openrouter/scripts/generate_image.sh --prompt "<draft prompt>" --filename auto --resolution 1K`
- 迭代：以小差异调整提示；每次运行保持文件名新
- 最终 (4K): 仅在提示确定后
  - `sh ~/.openclaw/workspace/skills/nano-banana-pro-openrouter/scripts/generate_image.sh --prompt "<final prompt>" --filename auto --resolution 4K`

## 分辨率选项

Gemini 3 Pro Image API 支持三种分辨率（需要大写 K）：

- 1K (默认) - ~1024px 分辨率
- 2K - ~2048px 分辨率
- 4K - ~4096px 分辨率

将用户请求映射到 API 参数：
- 未提及分辨率 -> 1K
- "low resolution", "1080", "1080p", "1K" -> 1K
- "2K", "2048", "normal", "medium resolution" -> 2K
- "high resolution", "high-res", "hi-res", "4K", "ultra" -> 4K

## API Key 和 Base URL

脚本按此顺序检查 API 密钥：
1. --api-key 参数（如果用户在聊天中提供了密钥）
2. OPENROUTER_API_KEY 环境变量

API base URL 必须通过 OPENROUTER_BASE_URL 设置。使用完整的聊天补全端点（对于 OpenRouter: `https://openrouter.ai/api/v1/chat/completions`）。

脚本还自动加载 .env 文件（如果存在）：
- 当前工作目录 .env
- 技能目录 .env

重要：如果存在 .env 文件，不要预先询问用户密钥。
直接运行脚本，仅当它以 "No API key provided." 错误时才询问。

### OpenClaw 聊天执行规则

- OpenClaw 不会自动加载技能 .env 文件
- 如果 `~/.openclaw/workspace/skills/nano-banana-pro-openrouter/.env` 存在：
  1. 使用 `read` 工具读取 `.env`
  2. 提取 `OPENROUTER_API_KEY` 和 `OPENROUTER_BASE_URL`
  3. 运行脚本时始终通过 `--api-key` 传递密钥
- 仅在 .env 缺失或无法读取密钥时询问用户
- 如果用户要求带时间戳的文件名，优先使用 `--filename auto`（不要手写日期）

如果两者都不可用，脚本将以错误消息退出。

## 预检和常见故障（快速修复）

预检：
- `command -v sh`（必须存在）
- `command -v curl`（必须存在）
- `command -v base64`（必须存在）

常见故障：
- `Error: No API key provided.` -> 读取 .env 并使用 --api-key 重试；如果仍然失败，要求用户设置 OPENROUTER_API_KEY
- `Error: No API base URL provided.` -> 确保 OPENROUTER_BASE_URL 设置为完整的聊天补全端点
- `Error loading input image:` -> 路径错误或文件不可读；验证 --input-image 指向真实图像
- "quota/permission/403" 风格的 API 错误 -> 错误的密钥、无访问权限或配额已用完；尝试不同的密钥/帐户

## 文件名生成

使用模式生成文件名：`yyyy-mm-dd-hh-mm-ss-name.png`

格式：`{timestamp}-{descriptive-name}.png`
- 时间戳：当前日期/时间，格式为 `yyyy-mm-dd-hh-mm-ss`（24小时制）
- 名称：描述性小写文本，带连字符
- 保持描述性部分简洁（通常1-5个词）
- 使用用户提示或对话中的上下文
- 如果不清楚，使用 `image`

示例：
- 提示 "A serene Japanese garden" -> `2025-11-23-14-23-05-japanese-garden.png`
- 提示 "sunset over mountains" -> `2025-11-23-15-30-12-sunset-mountains.png`
- 提示 "create an image of a robot" -> `2025-11-23-16-45-33-robot.png`
- 上下文不清楚 -> `2025-11-23-17-12-48-image.png`

提示：为避免不正确的时间戳，传递 `--filename auto` 并让脚本
使用系统时钟生成文件名。

## 图像编辑（Shell 版本中不支持）

Shell 脚本仅支持生成新图像。此版本中不提供输入图像编辑。

## 提示处理

对于生成：将用户的图像描述原样传递给 --prompt。仅当明显不足时才重新处理。

## 提示模板（高命中率）

当用户含糊或编辑必须精确时使用模板。

生成模板：
- "Create an image of: <subject>. Style: <style>. Composition: <camera/shot>. Lighting: <lighting>. Background: <background>. Color palette: <palette>. Avoid: <list>."

## 输出

- 将 PNG 保存到 `~/.openclaw/workspace/outputs/nano-banana-pro-openrouter`
- 如果 `--filename` 包含路径，则仅使用基本名称
- 脚本输出生成的图像的完整路径
- 脚本还为每个图像输出 `MEDIA_URL=file:///absolute/path`

### 在回复中显示图像

- 使用 MEDIA_URL 值在模型响应中附加图像
- 在 OpenClaw 中，优先发送带有 mediaUrl 设置为该 file:// URL 的消息
- 同时在文本中包含文件路径以供参考

## 示例

生成新图像：
```bash
sh ~/.openclaw/workspace/skills/nano-banana-pro-openrouter/scripts/generate_image.sh --prompt "A serene Japanese garden with cherry blossoms" --filename auto --resolution 4K
```
