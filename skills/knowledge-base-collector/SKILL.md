---
name: knowledge-base-collector
description: 从URL（网页/X/微信）和截图中收集并整理个人知识库。当用户表示他们想保存URL、摄取链接、将内容归档到KB、标记/分类笔记、存储截图或在Telegram中搜索他们保存的知识时使用。当云获取被阻止时，通过连接的macOS节点支持微信。
tags:
- 办公
- 即时通讯
- 社交媒体
---

## 摘要

- 摄取：网页URL、X/Twitter链接、微信公众号链接（mp.weixin.qq.com）和截图
- 存储：写入共享KB文件夹，每个项目包含 `content.md` + `meta.json` 以及全局 `index.jsonl`
- 整理：以标签优先的分类，使用更丰富的标签（例如 `#agent`, `#coding-agent`, `#claude-code`, `#mcp`, `#rag`, `#prompt-injection`, `#security`, `#pricing`, `#database`）
- 微信：云获取可能被阻止；当macOS节点（例如 Reed-Mac）在线时，优先使用节点端获取以提高成功率；否则创建占位条目
- 搜索：设计用于支持基于索引和内容的Telegram问答/搜索流程

把用户发来的链接/截图沉淀到共享知识库（KB），并做标签化整理。

## 默认 KB 位置
- KB Root（可改）：`/home/ubuntu/.openclaw/kb`
- 索引：`kb/20_Inbox/urls/index.jsonl`
- 每条内容目录：`kb/20_Inbox/urls/<YYYY-MM>/<item>/content.md + meta.json`

> 目标：**先入库不丢**，再迭代“摘要/标签/检索”。

## 你要做的事（按输入类型）

### 1) 普通网页 / X(Twitter) / 公众号 URL 入库
运行脚本：

```bash
python3 /home/ubuntu/.openclaw/skills/knowledge-base-collector/scripts/ingest_url.py "<URL>" --tags "#optional" --note "context"
```

行为：
- 自动识别来源（web/x/wechat）
- 优先用 `r.jina.ai` 抽取正文（无需登录）
- 公众号遇到风控会写占位条目：`status=blocked_verification` + tag `#needs-manual`
- 对同一 URL 做 key 去重（已存在则跳过）

#### WeChat 更高成功率（推荐路径）
当云端抓取命中“环境异常/验证”时：
- 如果有已连接的 macOS 节点（例如 `Reed-Mac`）且该节点能访问该文章，可用 `nodes.run` 在节点上执行抓取（requests+bs4），然后写入 KB。
- 注意：这条路径依赖节点在线与网络环境；无法承诺 100%。

### 2) 截图/图片入库（含 OCR 文本）
脚本：

```bash
python3 /home/ubuntu/.openclaw/skills/knowledge-base-collector/scripts/ingest_image.py /path/to/image.jpg \
  --text-file /path/to/ocr.txt \
  --title "..." --tags "#ai #product" --note "..."
```

说明：
- `ingest_image.py` 负责“落盘+索引”。OCR 可用：
  - 本机 tesseract（若安装了 `tesseract-ocr` + `chi_sim`）
  - 或用多模态 LLM 抽取文字后写入 `--text-file`

## Telegram 里直接问（检索）
推荐先用脚本（本机/服务器）：

```bash
python3 /home/ubuntu/.openclaw/skills/knowledge-base-collector/scripts/search_kb.py --q "claude code" --limit 10
python3 /home/ubuntu/.openclaw/skills/knowledge-base-collector/scripts/search_kb.py --tags "#claude-code #coding-agent" --limit 20
python3 /home/ubuntu/.openclaw/skills/knowledge-base-collector/scripts/search_kb.py --source wechat --since 7d --q "Elys"
```

## 公众号待补抓队列（占位条目）

```bash
python3 /home/ubuntu/.openclaw/skills/knowledge-base-collector/scripts/wechat_backlog.py --limit 30
```

## 周报/主题报告候选清单（给 LLM 写总结用）

```bash
python3 /home/ubuntu/.openclaw/skills/knowledge-base-collector/scripts/weekly_digest.py --days 7 --limit 30
```

## 重要注意事项（安全/隐私）
- 截图/网页可能包含 token/验证码/密钥：入库前应做脱敏（替换为 `REDACTED`）。
- 公众号抓取受风控影响：建议允许“占位入库”，后续再补全。
