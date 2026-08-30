---
name: veo3-video-gen
description: 通过 Gemini API（google-genai）使用 Google Veo 3.x 生成并拼接短视频。当你需要根据提示创建视频片段（广告、UGC 风格片段、产品演示）并想要可复现的 CLI 工作流（生成、轮询、下载 MP4，可选择拼接多个片段）时使用。
tags:
- API
---

# Veo 3 视频生成（Gemini API）

使用捆绑脚本从文本提示生成 MP4。

## 生成（文本 → 视频）

```bash
uv run {baseDir}/scripts/generate_video.py \
  --prompt "A close up of ..." \
  --filename "out.mp4" \
  --model "veo-3.1-generate-preview" \
  --aspect-ratio "9:16" \
  --poll-seconds 10
```

## 通过拼接片段生成更长的视频

Veo 通常每次请求输出约 8 秒的片段。使用 `--segments` 生成多个片段并使用 ffmpeg 拼接它们。

**重要：** 此技能为 **每个片段发送一个提示**（每个片段一个 Veo 请求）。使用 `--base-style` 保持跨片段的风格一致。

```bash
uv run {baseDir}/scripts/generate_video.py \
  --prompt "Same scene, consistent style..." \
  --filename "out-24s.mp4" \
  --model "veo-3.1-generate-preview" \
  --aspect-ratio "9:16" \
  --segments 3 \
  --segment-style continuation
```

选项：
- `--base-style "..."`：添加到每个片段提示的开头（推荐）。
- `--segment-prompt "..."`（可重复）：为每个片段提供一个提示（覆盖 `--prompt`）。
- `--segment-style continuation`（默认）：为每个片段添加连续性指令（仅在使用 `--prompt` 时）。
- `--segment-style same`：为每个片段使用完全相同的提示（仅在使用 `--prompt` 时）。
- `--use-last-frame`：对于片段 >=2，提取前一个片段的最后一帧并将其作为 `lastFrame` 传递以保持连续性。
- `--emit-segment-media`：在每个片段完成时打印 `MEDIA:`（用于进度显示）。
- `--keep-segments`：保留中间的 `*.segXX.mp4` 文件。
- `--reference-image path.jpg`（可重复）：使用产品/风格参考引导生成。

## 要求

- `GEMINI_API_KEY` 环境变量（或 `--api-key`）。
- 使用 `--segments > 1` 时 PATH 上需要有 `ffmpeg`。

## 故障排除

- 429/RESOURCE_EXHAUSTED：API 密钥没有视频配额/计费。
- 503/UNAVAILABLE：模型过载；稍后重试。
