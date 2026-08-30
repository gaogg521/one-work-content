---
name: aegis-audit
description: 针对 AI agent skills 与 MCP tools 的深度行为安全审计。执行确定性静态分析（AST + Semgrep + 15 个专用扫描器），生成加密 lockfile，并提供可选的 LLM 驱动意图分析。在安装、审查或批准任何 skill、tool、plugin 或 MCP server 前使用。输出包含完整 CWE 映射、OWASP 标记与行号引用的安全报告。触发词：安全审计(security audit)、skill 审查(skill review)、MCP 审计(MCP audit)、静态分析(static analysis)、lockfile。
version: 0.1.10
homepage: https://github.com/Aegis-Scan/aegis-scan
url: https://pypi.org/project/aegis-audit/
metadata: None
openclaw: None
emoji: 🔍
install:
- bins:
  - aegis
  kind: uv
  package: aegis-audit
requires: None
bins:
- aegis
config:
- ~/.aegis/config.yaml
tags:
- AI
- 安全
---

# Aegis Audit

面向 AI agent skills 和 MCP tools 的行为安全扫描器。

Aegis 是一个**防御性**安全审计工具。它检测其他 skills 中的恶意模式，以便用户可以避免危险的安装。本 skill 不教授或启用攻击 —— 它帮助用户在信任之前审查 skills。

> AI agent skills 的 "SSL 证书" —— 在信任之前扫描、认证和治理。

Source: [github.com/Aegis-Scan/aegis-scan](https://github.com/Aegis-Scan/aegis-scan) | Package: [pypi.org/project/aegis-audit](https://pypi.org/project/aegis-audit/) | License: AGPL-3.0

---

## Aegis 的功能

Aegis 回答了每个 agent 用户都应该问的问题：*"这个 skill 到底能做什么，我应该信任它吗？"*

- **确定性静态分析** —— AST 解析 + Semgrep + 15 个专用扫描器。相同的代码 = 每次相同的报告。
- **范围解析的能力** —— 不仅仅是 "访问文件系统"，而是确切地访问哪些文件、URL、主机和端口。
- **风险评分** —— 0-100 的综合评分，包含 CWE/OWASP 映射的发现和严重等级。
- **加密证明** —— 基于 Ed25519 签名的 lockfile，使用 Merkle 树进行防篡改检测。
- **可选的 LLM 分析** —— 使用你自己的 key（Gemini、Claude、OpenAI、Ollama、local）。默认禁用。启用前请参阅下方的隐私声明。

---

## 安装

使用 pip 或 uv 从 [PyPI](https://pypi.org/project/aegis-audit/) 安装：

```bash
pip install aegis-audit
```

```bash
uv tool install aegis-audit
```

两个命令安装的是同一个包。尽可能固定到特定版本（例如 `pip install aegis-audit==1.3.0`），并在安装前在 PyPI 上验证发布者。包源代码位于 [github.com/Aegis-Scan/aegis-scan](https://github.com/Aegis-Scan/aegis-scan)。

安装后，`aegis` CLI 将可在你的 PATH 中使用。

---

## 快速开始

Aegis 默认完全离线运行。无需 API 密钥，无需网络访问，数据不会离开你的机器。

```bash
aegis scan --no-llm
```

这会扫描当前目录并生成安全报告。所有命令在未提供路径时默认使用 `.`（当前目录）。

```bash
aegis scan ./some-skill --no-llm
```

---

## CLI 参考

| Command | Description |
|---|---|
| `aegis scan [path]` | 完整的安全扫描及风险评分 |
| `aegis lock [path]` | 扫描 + 生成签名的 `aegis.lock` |
| `aegis verify [path]` | 验证 lockfile 是否与当前代码一致 |
| `aegis badge [path]` | 生成 shields.io badge markdown |
| `aegis setup` | 交互式 LLM 配置向导 |
| `aegis mcp-serve` | 启动 MCP server（stdio transport） |
| `aegis mcp-config` | 为 Cursor / Claude Desktop 打印 MCP 配置 JSON |
| `aegis version` | 显示 Aegis 版本 |

常用 flags：`--no-llm`（跳过 LLM，默认行为），`--json`（CI 输出），`-v`（verbose）。

---

## Lockfiles

扫描后生成签名的 lockfile：

```bash
aegis lock
```

这会生成 `aegis.lock` —— 一个 skill 安全状态的加密签名快照。将其与 skill 一起提交，以便消费者可以验证没有任何更改。

验证 lockfile：

```bash
aegis verify
```

如果自 lockfile 创建以来任何文件被修改，Merkle root 将不匹配，验证将失败。

---

## 可选：LLM 分析

**隐私声明：** LLM 分析默认禁用。启用后，Aegis 会将扫描的代码发送到配置的第三方 LLM 提供商（Google、OpenAI 或 Anthropic）。除非你明确配置了 API 密钥并在未使用 `--no-llm` 的情况下运行扫描，否则不会传输任何数据。不要在包含 secrets 或敏感代码的仓库上启用 LLM 模式，除非你信任该提供商。

要启用 LLM 分析，请运行交互式设置：

```bash
aegis setup
```

这会将你的配置保存到 `~/.aegis/config.yaml`。或者，设置以下环境变量之一：

- `GEMINI_API_KEY` —— Google Gemini
- `OPENAI_API_KEY` —— OpenAI
- `ANTHROPIC_API_KEY` —— Anthropic Claude

这些环境变量是可选的。Aegis 在没有它们的情况下可以完全离线工作。只有在你想要 AI 第二意见功能并接受扫描的代码将被发送到相应提供商时才设置 key。

对于本地 LLM 服务器（Ollama、LM Studio、llama.cpp、vLLM），请参阅 `aegis setup` —— 使用本地模型时不会发生第三方数据传输。

---

## MCP server

Aegis 作为 MCP server 运行在 Cursor、Claude Desktop 和任何 MCP 兼容的客户端中。暴露了三个 tools：`scan_skill`、`verify_lockfile` 和 `list_capabilities`。

将此添加到你的 `.cursor/mcp.json`：

```json
{
  "mcpServers": {
    "aegis": {
      "command": "aegis",
      "args": ["mcp-serve"]
    }
  }
}
```

或自动生成：

```bash
aegis mcp-config
```

Aegis 使用 stdio transport —— 无需网络 server。

---

## 扫描内容

| Scanner | What it detects |
|---|---|
| AST Parser | 750+ Python 函数/方法模式，涵盖 15+ 类别 |
| Semgrep Rules | 80+ 用于 Python、JavaScript 和 secrets 的正则规则 |
| Secret Scanner | API keys、tokens、private keys、connection strings（30+ 模式） |
| Shell Analyzer | Pipe-to-shell、reverse shells、inline exec |
| JS Analyzer | XSS、eval、prototype pollution、dynamic imports |
| Dockerfile Analyzer | Privilege escalation、secrets in ENV/ARG、unpinned images |
| Config Analyzer | YAML、JSON、TOML、INI 中的危险设置 |
| Social Engineering | Misleading filenames、Unicode tricks、trust manipulation |
| Steganography | Images 中的隐藏 payloads、homoglyph attacks |
| Shadow Module Detector | Stdlib-shadowing files（skill 中的 os.py、sys.py） |
| Combo Analyzer | Multi-capability attack chains（exfiltration、C2、ransomware） |
| Taint Analysis | Source-to-sink data flows（commands、URLs、SQL、paths） |
| Complexity Analyzer | Cyclomatic complexity warnings for hard-to-audit functions |
| Skill Meta Analyzer | SKILL.md 与实际代码的交叉引用 |
| Persona Classifier | Overall trust profile（LGTM、Permission Goblin 等） |

---

## Vibe Check personas

Aegis 基于确定性分析为每个扫描的 skill 分配一个 persona：

- **Cracked Dev** —— 干净的代码，聪明的 patterns，最小的权限。
- **LGTM** —— 权限与意图匹配，范围合理，没有异常。
- **Trust Me Bro** —— 外表光鲜，内部可疑。
- **You Sure About That?** —— 代码混乱，缺少部分，文档过度承诺。
- **Co-Dependent Lover** —— 逻辑很小，依赖树很大。供应链风险。
- **Permission Goblin** —— 想要一切：文件系统、网络、secrets。
- **Spaghetti Monster** —— 难以理解的混乱。高复杂度。
- **The Snake** —— 看起来干净但实际不然的代码。可能具有恶意。

---

## 用于 CI 的 JSON 输出

```bash
aegis scan --json --no-llm
```

```bash
aegis scan --json --no-llm | jq '.deterministic.risk_score_static'
```

```bash
aegis scan --json --no-llm | jq -e '.deterministic.risk_score_static <= 50'
```

JSON 报告包含两个 payloads：

- **Deterministic** —— Merkle tree、capabilities、findings、risk score（可重现、已签名）
- **Ephemeral** —— LLM 分析、risk adjustment（非确定性、未签名）

---

## 面向 skill 开发者

在发布前对你自己的 skill 运行 Aegis：

```bash
cd ./my-skill
aegis scan --no-llm -v
```

修复 PROHIBITED 发现。记录 RESTRICTED 发现。随附 `aegis.lock` 一起发布：

```bash
aegis lock
```

请参阅 [Skill Developer Best Practices](https://github.com/Aegis-Scan/aegis-scan/blob/main/docs/SKILL_DEVELOPER_GUIDE.md) 指南。

---

## 架构

```
aegis scan ./skill
    |
    +-- coordinator.py       File discovery（git-aware / directory walk）
    +-- ast_parser.py        AST 分析 + pessimistic scope extraction
    +-- secret_scanner.py    30+ secret patterns
    +-- shell_analyzer.py    Dangerous shell patterns
    +-- js_analyzer.py       JS/TS vulnerability patterns
    +-- config_analyzer.py   YAML/JSON/TOML/INI risky settings
    +-- combo_analyzer.py    Multi-capability attack chains
    +-- taint_analyzer.py    Source-to-sink data flow tracking
    +-- binary_detector.py   External binary classification
    +-- social_eng_scanner   Social engineering detection
    +-- stego_scanner        Steganography + homoglyphs
    +-- hasher.py            Lazy Merkle tree
    +-- signer.py            Ed25519 signing
    +-- rule_engine.py       Policy evaluation
    +-- reporter/            JSON + Rich console output
         |
         v
    aegis_report.json + aegis.lock
```

---

## License

Aegis 采用双重许可：

- **Open Source:** AGPL-3.0 —— 免费使用、修改和分发。网络服务部署必须发布源代码。
- **Commercial:** 提供专有许可，用于嵌入专有产品、无需源代码披露的运行、SLA 和支持。

有关完整详情，请参阅 [LICENSING.md](https://github.com/Aegis-Scan/aegis-scan/blob/main/aegis-core/LICENSING.md)。

---

## Contributing

欢迎贡献。通过贡献，你同意 [Contributor License Agreement](https://github.com/Aegis-Scan/aegis-scan/blob/main/aegis-core/CLA.md)。

```bash
cd aegis-core
pip install -e ".[dev]"
pytest
```

---

需要 Python 3.11+。确定性扫描无需网络访问。可离线工作。
