---
name: openclaw-skill-scanner
description: Skill 安全扫描器，在安装前后扫描 ClawHub skills 中的恶意模式。检测 base64 payloads、reverse shells、数据外泄(data exfiltration)、加密挖矿(crypto mining)、混淆 URL(obfuscated URL) 等威胁。触发词：skill 扫描、恶意模式检测、安全扫描(security scan)、ClawHub 安全
tags:
- Shell
- 安全
---

# Skill Scanner

**Name:** skill-scanner
**Version:** 1.0.0
**Author:** vrtlly.us
**Category:** Security

## 描述

在安装前后扫描 ClawHub skills 中的恶意模式。检测 base64 payloads、reverse shells、数据外泄、加密挖矿、混淆 URL 等。

## 用法

### 扫描所有已安装 skills
```bash
python3 scanner.py
```

### 扫描特定 skill
```bash
python3 scanner.py --skill <skill-name>
```

### 扫描特定文件
```bash
python3 scanner.py --file <path-to-file>
```

### 安装前扫描 (download → scan → report → cleanup)
```bash
python3 scanner.py --pre-install <clawhub-slug>
```

### JSON 输出
```bash
python3 scanner.py --json
python3 scanner.py --skill <name> --json
```

### 安全安装 hook
```bash
bash install-hook.sh <clawhub-slug>
bash install-hook.sh <clawhub-slug> --force
```

## 检测模式

| 类别 | 捕获内容 |
|---|---|
| Base64 payloads | exec/bash/eval 附近的长 base64 字符串 |
| Pipe to shell | `curl ... \| bash`, `wget ... \| sh` |
| Raw IP 连接 | `http://1.2.3.4` 风格 URL |
| 危险函数 | `eval()`, `exec()`, `os.system()`, `subprocess(shell=True)` |
| 隐藏文件 | 在意外位置创建 dotfile |
| Env 外泄 | 读取 `.env`、API keys 发送到外部 |
| 混淆 URL | rentry.co, pastebin, hastebin redirectors |
| 虚假依赖 | 引用不存在的包 |
| 数据外泄端点 | webhook.site, requestbin 等 |
| 加密挖矿 | xmrig, stratum, mining pool 引用 |
| 密码存档 | 带密码保护的 zip/tar 下载 |

## 风险评分

- **0-29 (Green):** 干净 — 未发现可疑模式
- **30-69 (Yellow):** 可疑 — 使用前审查警告
- **70-100 (Red):** 危险 — 可能恶意，不要安装

## 文件

- `scanner.py` — 主扫描引擎
- `install-hook.sh` — 安全安装包装器
- `whitelist.json` — 已知良好和已知不良 skill 列表
- `report-template.md` — Markdown 报告模板
