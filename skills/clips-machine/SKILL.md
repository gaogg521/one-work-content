---
name: clips-machine
description: 将长视频转换为病毒式短视频。自动检测最佳时刻，添加流行字幕，导出用于 TikTok/Reels/Shorts。自包含，无外部模块。100% 免费工具。
version: 1.1.0
author: Mayank8290
homepage: https://github.com/Mayank8290/openclaw-video-skills
tags:
- 视频
- clips
- tiktok
- reels
- shorts
metadata: None
openclaw: None
requires: None
bins:
- ffmpeg
- yt-dlp
- whisper-cpp
---

# Clips Machine

将长视频转换为病毒式短视频。自动检测最佳时刻，添加流行字幕，导出用于 TikTok/Reels/Shorts。

**100% 免费工具** - 完全在你的机器上运行。

> **喜欢此技能？** 支持创作者并帮助保持免费：**[Buy Me a Coffee](https://buymeacoffee.com/mayank8290)**

## 此技能的功能

1. **输入** 任何长视频（YouTube URL、播客、直播、本地文件）
2. **转录** 使用 Whisper（免费，本地）带时间戳
3. **检测** 使用 AI 分析检测病毒式时刻
4. **剪切** 最佳的 30-60 秒片段
5. **字幕** 使用动画、流行文本样式
6. **导出** 为垂直 9:16 格式，可立即上传

## 快速开始

```
Turn this podcast into viral clips: https://youtube.com/watch?v=xyz
```

```
Extract the 5 best moments from my-interview.mp4 and add captions
```

## 命令

### 从 YouTube URL
```
/clips-machine https://youtube.com/watch?v=VIDEO_ID
```

### 从本地文件
```
/clips-machine /path/to/video.mp4
```

### 自定义剪辑数量
```
/clips-machine VIDEO --clips 10
```

### 字幕样式
```
/clips-machine VIDEO --style [style]
```

可用样式：
- `hormozi` - Alex Hormozi 样式（粗体，逐字高亮）- **最病毒式**
- `minimal` - 简洁白色文本
- `karaoke` - 逐字变色
- `news` - 下三分之一样式
- `meme` - Impact 字体，顶部/底部

## 病毒检测的工作原理

AI 分析转录文本，寻找：

1. **钩子潜力** - 强有力的开场白
2. **情感高峰** - 激情、兴奋、惊喜
3. **可引用台词** - 令人难忘的一句话
4. **争议性观点** - 值得辩论的观点
5. **令人惊讶的事实** - "你知道吗" 时刻
6. **可操作的建议** - 明确的要点

每个时刻都会获得 1-100 的 "病毒式评分"。

## 输出结构

```
~/Videos/OpenClaw/clips-[video-name]/
├── transcript.json      # 带时间戳的完整转录
├── viral_moments.json   # 检测到的时刻及评分
├── clip_001.mp4         # 第一个病毒式剪辑（垂直，带字幕）
├── clip_002.mp4         # 第二个病毒式剪辑
├── clip_003.mp4         # ...
└── summary.md           # 所有剪辑的概述
```

## 支持的来源

| 来源 | 示例 |
|--------|---------|
| YouTube | `https://youtube.com/watch?v=...` |
| TikTok | `https://tiktok.com/@user/video/...` |
| Twitter/X | `https://twitter.com/user/status/...` |
| Twitch VOD | `https://twitch.tv/videos/...` |
| 本地 MP4 | `/path/to/file.mp4` |

## 要求

- FFmpeg (`brew install ffmpeg`)
- yt-dlp (`brew install yt-dlp`)
- Whisper.cpp (`brew install whisper-cpp`)

## 设置

```bash
# 安装依赖
brew install ffmpeg yt-dlp whisper-cpp

# 或在 Linux 上
sudo apt install ffmpeg
pip install yt-dlp
# 从源代码构建 whisper.cpp
```

## 变现

| 方法 | 潜力 |
|--------|-----------|
| 为创作者提供剪辑服务 | $50-150/视频 |
| 月度聘用 | $500-2,000/客户 |
| 播客剪辑机构 | $2,000-5,000/月 |
| 出售此技能 | $100-300 on ClawHub |

## 示例

### 播客到剪辑
```
Take this 2-hour podcast and find the 10 best moments:
https://youtube.com/watch?v=PODCAST_ID
Make them Hormozi-style with bold captions.
```

### 采访亮点
```
/clips-machine interview.mp4 --clips 5 --style minimal
```

### 游戏直播
```
Extract funny moments from my Twitch VOD:
https://twitch.tv/videos/12345
Add meme-style captions
```

---

## 支持此项目

如果此技能为你节省了时间或让你赚钱，请考虑给我买杯咖啡！

**[Buy Me a Coffee](https://buymeacoffee.com/mayank8290)**

每杯咖啡都帮助我为社区构建更多免费工具。

---

Built for OpenClaw | Powered by Whisper + FFmpeg | [Support the Creator](https://buymeacoffee.com/mayank8290)
