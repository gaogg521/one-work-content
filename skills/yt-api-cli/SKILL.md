---
name: yt-api-cli
description: 从命令行管理你的 YouTube 账户。YouTube Data API v3 的完整 CLI — 列出/搜索视频、上传、管理播放列表等。
metadata: None
clawdbot: None
emoji: ▶️
os:
- darwin
- linux
requires: None
env:
- YT_API_AUTH_TYPE
- YT_API_CLIENT_ID
- YT_API_CLIENT_SECRET
tags:
- API
---

# yt-api-cli

从终端管理你的 YouTube 账户。YouTube Data API v3 的完整 CLI。

## 安装

```bash
# 使用 go install
go install github.com/nerveband/youtube-api-cli/cmd/yt-api@latest

# 或从 releases 下载
curl -L -o yt-api https://github.com/nerveband/youtube-api-cli/releases/latest/download/yt-api-darwin-arm64
chmod +x yt-api
sudo mv yt-api /usr/local/bin/
```

## 设置

### 1. Google Cloud Console 设置

1. 前往 [Google Cloud Console](https://console.cloud.google.com)
2. 创建/启用 YouTube Data API v3
3. 创建 OAuth 2.0 凭据（桌面应用）
4. 下载客户端配置

### 2. 配置 yt-api

```bash
mkdir -p ~/.yt-api
cat > ~/.yt-api/config.yaml << EOF
default_auth: oauth
default_output: json
oauth:
  client_id: "YOUR_CLIENT_ID"
  client_secret: "YOUR_CLIENT_SECRET"
EOF
```

### 3. 认证

```bash
yt-api auth login  # 打开浏览器进行 Google 登录
yt-api auth status # 检查认证状态
```

## 命令

### 列表操作

```bash
# 列出你的视频
yt-api list videos --mine

# 列出频道视频
yt-api list videos --channel-id UC_x5XG1OV2P6uZZ5FSM9Ttw

# 列出播放列表
yt-api list playlists --mine

# 列出订阅
yt-api list subscriptions --mine
```

### 搜索

```bash
# 基本搜索
yt-api search --query "golang tutorial"

# 带过滤条件
yt-api search --query "music" --type video --duration medium --order viewCount
```

### 上传操作

```bash
# 上传视频
yt-api upload video ./video.mp4 \
  --title "My Video" \
  --description "Description here" \
  --tags "tag1,tag2" \
  --privacy public

# 上传缩略图
yt-api upload thumbnail ./thumb.jpg --video-id VIDEO_ID
```

### 播放列表管理

```bash
# 创建播放列表
yt-api insert playlist --title "My Playlist" --privacy private

# 添加视频到播放列表
yt-api insert playlist-item --playlist-id PLxxx --video-id VIDxxx
```

### 频道操作

```bash
# 获取频道信息
yt-api list channels --id UCxxx --part snippet,statistics

# 更新频道描述
yt-api update channel --id UCxxx --description "New description"
```

## 输出格式

```bash
# JSON（默认 - 适合 LLM）
yt-api list videos --mine

# 表格（人类可读）
yt-api list videos --mine -o table

# YAML
yt-api list videos --mine -o yaml

# CSV
yt-api list videos --mine -o csv > videos.csv
```

## 全局标志

| 标志 | 简写 | 描述 |
|------|-------|-------------|
| `--output` | `-o` | 输出格式：json（默认）、yaml、csv、table |
| `--quiet` | `-q` | 抑制 stderr 消息 |
| `--config` | | 配置文件路径 |
| `--auth-type` | | 认证方法：oauth（默认）、service-account |

## 环境变量

| 变量 | 描述 |
|----------|-------------|
| `YT_API_AUTH_TYPE` | 认证方法：oauth 或 service-account |
| `YT_API_OUTPUT` | 默认输出格式 |
| `YT_API_CLIENT_ID` | OAuth 客户端 ID |
| `YT_API_CLIENT_SECRET` | OAuth 客户端密钥 |
| `YT_API_CREDENTIALS` | 服务账户 JSON 路径 |

## 认证方法

### OAuth 2.0（默认）
最适合交互式使用和访问你自己的 YouTube 账户。

```bash
yt-api auth login  # 打开浏览器
```

### 服务账户
最适合服务器端自动化。

```bash
yt-api --auth-type service-account --credentials ./key.json list videos
```

## 快速诊断命令

```bash
yt-api info                      # 完整系统状态
yt-api info --test-connectivity  # 验证 API 访问
yt-api info --test-permissions   # 检查凭据能力
yt-api auth status               # 认证详情
yt-api version                   # 版本信息
```

## 错误处理

退出代码：
- `0` - 成功
- `1` - 一般错误
- `2` - 认证错误
- `3` - API 错误（配额、权限）
- `4` - 输入错误

## 面向 LLM 和自动化

- 默认 JSON 输出
- 结构化错误为 JSON 对象
- `--quiet` 模式用于解析
- `--dry-run` 验证而不执行
- 支持标准输入管道数据

## 注意事项

- 需要有效的 Google Cloud 凭据并启用 YouTube Data API v3
- OAuth 令牌存储在 `~/.yt-api/tokens.json` 中（权限 0600）
- 默认输出为 JSON（针对 LLM 优化）
- 支持所有 YouTube Data API v3 资源

## 来源

GitHub: https://github.com/nerveband/youtube-api-cli
