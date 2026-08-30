---
name: hefestoai-auditor
version: 1.2.0
description: 使用 HefestoAI 进行 AI 驱动的代码分析。运行安全审计(security audit)、检测代码异味(code smell)、分析复杂度(complexity)，并获取跨 17 种语言的 ML 增强建议。触发词：代码分析(code analysis)、安全审计(security audit)、代码质量(code quality)、漏洞检测(vulnerability detection)
metadata: None
openclaw: None
emoji: 🔨
install:
- bins:
  - hefesto
  id: pip
  kind: pip
  label: Install HefestoAI (pip)
  package: hefesto-ai
requires: None
bins:
- hefesto
tags:
- 安全
- AI
- 代码审查
---

# HefestoAI Auditor Skill

AI驱动的代码质量守护者。分析代码中的安全漏洞、复杂度问题、代码异味和最佳实践违规，支持17种语言。

## 快速开始

### 运行完整审计

```bash
# IMPORTANTE: Cargar environment primero para activar licencia
source /home/user/.hefesto_env 2>/dev/null
hefesto analyze /ruta/absoluta/al/proyecto --severity HIGH --exclude venv,node_modules,.git
```

### 严重级别

```bash
hefesto analyze /path/to/project --severity CRITICAL   # Solo criticos
hefesto analyze /path/to/project --severity HIGH        # High y Critical
hefesto analyze /path/to/project --severity MEDIUM      # Medium, High, Critical
hefesto analyze /path/to/project --severity LOW         # Todo
```

### 输出格式

```bash
hefesto analyze /path/to/project --output text          # Default, terminal
hefesto analyze /path/to/project --output json          # JSON estructurado
hefesto analyze /path/to/project --output html --save-html report.html  # Reporte HTML
hefesto analyze /path/to/project --quiet                # Solo resumen
```

### 检查状态和版本

```bash
hefesto status
hefesto --version
```


## 推荐：包装脚本

为了获得可靠的结果，创建一个始终加载你的 license 的包装脚本：

```bash
#!/bin/bash
# Save as /usr/local/bin/hefesto (replaces direct binary)
source /path/to/.hefesto_env 2>/dev/null
exec /path/to/venv/bin/hefesto "$@"
```

这确保你的 license tier 始终处于激活状态，无论 hefesto 如何被调用。

### 预构建审计脚本

```bash
# Save as ~/hefesto_tools/run_audit.sh
#!/bin/bash
SEVERITY="${1:-HIGH}"
TARGET="${2:-/path/to/your/project}"
source /path/to/.hefesto_env 2>/dev/null
exec hefesto analyze "$TARGET" --severity "$SEVERITY" --exclude venv,node_modules,.git
```

用法：
```bash
bash ~/hefesto_tools/run_audit.sh              # HIGH severity, default project
bash ~/hefesto_tools/run_audit.sh CRITICAL     # CRITICAL only
bash ~/hefesto_tools/run_audit.sh MEDIUM /other/project  # Custom
```

## 重要说明

- **SIEMPRE** 使用绝对路径，永远不要使用 `。` 或相对路径
- **SIEMPRE** 在执行前加载 environment (`source /home/user/.hefesto_env`) 以激活你的 license
- **SIEMPRE** 排除 `venv,node_modules,.git` 以避免依赖项的误报
- **仅报告** hefesto 在其输出中返回的内容。不要编造或添加额外的问题。

## 支持的语言 (17)

**代码：** Python, TypeScript, JavaScript, Java, Go, Rust, C#
**DevOps/配置：** Dockerfile, Jenkins/Groovy, JSON, Makefile, PowerShell, Shell, SQL, Terraform, TOML, YAML

## 检测内容

### 安全问题
- SQL 注入漏洞
- 硬编码的 secrets 和 API keys
- 命令注入风险
- 不安全的配置

### 代码质量
- 圈复杂度（函数过于复杂）
- 深层嵌套（>4 层）
- 长函数（>50 行）
- 代码异味和反模式

### DevOps 问题
- Dockerfile: 缺少 USER, 没有 HEALTHCHECK, 以 root 运行
- Shell: 缺少 `set -euo pipefail`, 未引用的变量
- Terraform: 缺少 tags, 硬编码的值

## 解读结果

HefestoAI 以如下格式输出结果：

```
📄 <file>:<line>:<col>
├─ Issue: <description>
├─ Function: <name>
├─ Type: <issue_type>
├─ Severity: CRITICAL | HIGH | MEDIUM | LOW
└─ Suggestion: <fix recommendation>
```

### 严重级别指南
- **CRITICAL**: 圈复杂度 >20。立即修复。
- **HIGH**: 复杂度 10-20, 深层嵌套, SQL 注入风险。在当前 sprint 中修复。
- **MEDIUM**: 风格问题, 轻微改进。方便时修复。
- **LOW**: 信息性, 最佳实践建议。

### 问题类型
- `VERY_HIGH_COMPLEXITY`: 圈复杂度 >20
- `HIGH_COMPLEXITY`: 圈复杂度 10-20
- `DEEP_NESTING`: 嵌套级别超过阈值 (default: 4)
- `SQL_INJECTION_RISK`: 通过字符串拼接潜在的 SQL 注入
- `LONG_FUNCTION`: 函数超过行阈值

## 专业提示

### 排除目录
始终排除依赖项以避免误报：
```bash
hefesto analyze /path/to/project --severity HIGH --exclude venv,node_modules,.git
```

### CI/CD 门禁
如果发现问题则构建失败：
```bash
hefesto analyze /path/to/project --fail-on HIGH --exclude venv
```

### 安装 pre-push hook
```bash
hefesto install-hook
```

### 限制输出
```bash
hefesto analyze /path/to/project --max-issues 10
```

### 排除特定问题类型
```bash
hefesto analyze /path/to/project --exclude-types VERY_HIGH_COMPLEXITY,LONG_FUNCTION
```

## License Tiers

| Tier | Price | Key Features |
|------|-------|-------------|
| **FREE** | USD0/mo | 静态分析, 17 种语言, pre-push hooks |
| **PRO** | USD8/mo (launch) | ML 语义分析, REST API, BigQuery, 自定义规则 |
| **OMEGA** | USD19/mo (launch) | IRIS 监控, 自动关联, 实时告警, 团队仪表板 |

所有付费 tiers 包含 **14 天免费试用**。

### 升级链接
- **PRO**: https://buy.stripe.com/4gM00i6jE6gV3zE4HseAg0b
- **OMEGA**: https://buy.stripe.com/14A9AS23o20Fgmqb5QeAg0c

### 激活 License
```bash
export HEFESTO_LICENSE_KEY=<your-key>
hefesto status  # verify tier
```

## 关于

Created by **Narapa LLC** (Miami, FL) — Arturo Velasquez (@artvepa)
GitHub: https://github.com/artvepa80/Agents-Hefesto
Support: support@narapallc.com

> "El codigo limpio es codigo seguro"
