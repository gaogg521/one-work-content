---
name: openclaw-policy-check
description: 在代码执行前扫描仓库中的高风险安全模式。适用于快速预检安全检查、策略执行扫描、可疑代码分类，以及检测不安全命令、密钥泄漏和危险 shell 行为。触发词：安全检查(security check)、策略扫描(policy scan)、密钥泄漏(secret leak)、shell 安全
tags:
- Shell
- Vault
- 安全
---

# OpenClaw Policy Check

运行轻量级策略扫描，以捕获代码和脚本中的常见高风险模式。

## Inputs

- `target_path` (required): 要扫描的文件或目录。
- `fail_on` (optional): 非零退出的严重级别阈值。可选值为 `critical`、`high`、`medium`、`low` 之一。
- `json_output` (optional): 打印原始 JSON 输出。

## Workflow

1. 在目标路径上运行 `scripts/policy_check.py`。
2. 查看严重级别计数和主要发现。
3. 如果存在发现，优先处理 `critical` 和 `high` 级别项。
4. 为每个标记的模式建议具体修复方案。

## Commands

```bash
python3 scripts/policy_check.py "<target_path>"
python3 scripts/policy_check.py "<target_path>" --json
python3 scripts/policy_check.py "<target_path>" --fail-on high
```

## Response Contract

- 始终包含发现总数和严重级别细分。
- 包含主要发现，包括 `file:line`、规则 id 和原因。
- 如果没有发现，明确说明未检测到策略违规。
- 修复指导应具体且简洁。
