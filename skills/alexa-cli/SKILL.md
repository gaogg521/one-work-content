---
name: alexa-cli
description: 通过 `alexacli` CLI 控制 Amazon Alexa 设备和智能家居。当用户要求在 Echo 设备上说话/广播、控制灯光/恒温器/锁、发送语音命令或查询 Alexa 时使用。
homepage: https://github.com/buddyh/alexa-cli
metadata:
  openclaw:
    emoji: 🔊
    requires:
      bins:
      - alexacli
    env:
      ALEXA_REFRESH_TOKEN: optional
    install:
    - id: brew
      kind: brew
      formula: buddyh/tap/alexacli
      bins:
      - alexacli
      label: 安装 alexacli (brew)
    - id: go
      kind: go
      module: github.com/buddyh/alexa-cli/cmd/alexa@latest
      bins:
      - alexacli
      label: 安装 alexa-cli (go)
tags:
- 自动化
- CLI
---

# Alexa CLI

使用 `alexacli` 通过非官方的 Alexa API 控制 Amazon Echo 设备和智能家居。

## 认证

```bash
# 浏览器登录（推荐）
alexacli auth

# 非美国账户
alexacli auth --domain amazon.de
alexacli auth --domain amazon.co.uk

# 检查认证状态
alexacli auth status
alexacli auth status --verify    # 针对 API 验证令牌

# 移除凭据
alexacli auth logout
```

令牌有效期约 14 天。配置存储在 `~/.alexa-cli/config.json` 中。

## 设备

```bash
alexacli devices
alexacli devices --json
```

## 文本转语音

```bash
# 在特定设备上说话
alexacli speak "Hello world" -d "Kitchen Echo"

# 向所有设备广播
alexacli speak "Dinner is ready!" --announce

# 设备名称匹配是灵活的
alexacli speak "Build complete" -d Kitchen
```

## 语音命令（智能家居控制）

发送任何命令，就像你对 Alexa 说话一样：

```bash
# 灯光、开关、插头
alexacli command "turn off the living room lights" -d Kitchen
alexacli command "dim the bedroom lights to 50 percent" -d Bedroom

# 恒温器
alexacli command "set thermostat to 72 degrees" -d Bedroom
alexacli command "what's the temperature inside" -d Kitchen

# 锁
alexacli command "lock the front door" -d Kitchen

# 音乐
alexacli command "play jazz music" -d "Living Room"
alexacli command "stop" -d "Living Room"

# 问题
alexacli command "what's the weather" -d Kitchen

# 计时器
alexacli command "set a timer for 10 minutes" -d Kitchen
```

## Ask（获取响应）

发送命令并捕获 Alexa 的文本响应：

```bash
alexacli ask "what's the thermostat set to" -d Kitchen
# 输出：The thermostat is set to 68 degrees.

alexacli ask "what's on my calendar today" -d Kitchen --json
```

## Alexa+（LLM 对话）

与 Amazon 的 LLM 驱动助手交互：

```bash
# 快速开始 - 自动选择对话
alexacli askplus -d "Echo Show" "What's the capital of France?"

# 多轮保留上下文
alexacli askplus -d "Echo Show" "What about Germany?"

# 列出对话
alexacli conversations

# 查看对话历史
alexacli fragments "amzn1.conversation.xxx"
```

## 音频播放

通过 Echo 设备播放 MP3 音频：

```bash
alexacli play --url "https://example.com/audio.mp3" -d "Echo Show"
```

要求：48kbps 的 MP3，22050Hz 采样率，HTTPS URL。

## 历史

```bash
alexacli history
alexacli history --limit 5
alexacli history --json
```

## 命令参考

| 命令 | 描述 |
|---------|-------------|
| `alexacli devices` | 列出所有 Echo 设备 |
| `alexacli speak <text> -d <device>` | 设备上的文本转语音 |
| `alexacli speak <text> --announce` | 向所有设备广播 |
| `alexacli command <text> -d <device>` | 语音命令（智能家居、音乐等） |
| `alexacli ask <text> -d <device>` | 发送命令，获取响应 |
| `alexacli conversations` | 列出 Alexa+ 对话 ID |
| `alexacli fragments <id>` | 查看 Alexa+ 对话历史 |
| `alexacli askplus -d <device> <text>` | Alexa+ LLM 对话 |
| `alexacli play --url <url> -d <device>` | 通过 SSML 播放 MP3 |
| `alexacli auth` | 浏览器登录或手动令牌 |
| `alexacli auth status [--verify]` | 显示认证状态 |
| `alexacli auth logout` | 移除凭据 |
| `alexacli history` | 查看最近的语音活动 |

## 注意

- 使用 Amazon 的非官方 API（与 Alexa 应用程序相同）
- 刷新令牌有效期约 14 天，如果过期请重新运行 `alexacli auth`
- 设备名称支持部分、不区分大小写的匹配
- 对于 AI/agentic 使用，`alexacli command` 使用自然语言是首选
- 向任何命令添加 `--verbose` 或 `-v` 以获取调试输出
