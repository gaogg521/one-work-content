---
name: clawshield
display_name: ClawShield
version: 1.1.0
description: OpenClaw 安全审计 + 提示注入检测器。扫描 gateway/vulns/cron/PI 模式。用于 frenzy-proofing 安装。
category: security
author: Jeffrey Coleman (smallbizailab79@gmail.com)
price: 9.99
inputs: []
outputs:
- JSON report printed to stdout
tags:
- 自动化
---

# ClawShield

## 目的
审计本地 OpenClaw 安装的安全态势和常见的提示注入指标。生成 JSON 报告以供审查和警报。

## 工作流
1. **展示面板**：启动面板服务器并展示 UI。
2. **用户配置**：更新 `config.yaml`（扫描频率、警报、灵敏度）。
3. **Cron 设置**：按所选频率安排 `scripts/audit.sh`。
4. **报告/警报**：审查 JSON 输出，并在发现提示注入命中或意外开放端口时发出警报。

## 用法
### 面板（推荐）
```bash
node scripts/panel-server.js
```
然后展示 UI：
- `canvas.present` → `http://localhost:8133` (扫描 / 设置 / 日志)

### 配置 (CLI)
```bash
node scripts/config.js get
node scripts/config.js set Scan_freq daily alerts telegram sensitivity high
```

### 审计 (CLI)
```bash
bash scripts/audit.sh > report.json
```

## 说明
- 仅限本地扫描；除 localhost 外不进行网络调用。
- 面板服务器是本地的，并将最后一份报告存储在 `logs/last-report.json`。
- `config.yaml` 默认值：Scan_freq=daily, alerts=telegram, sensitivity=high。
- 适用于常规安全检查和 "frenzy-proofing"。

联系：Jeffrey Coleman | smallbizailab79@gmail.com | 定制审计/企业版。
