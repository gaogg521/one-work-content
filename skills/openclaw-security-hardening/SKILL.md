---
name: openclaw-security-hardening
description: 保护 OpenClaw 安装免受提示注入、数据外泄、恶意技能和工作空间篡改
version: 1.0.0
author: openclaw-community
tags:
- 安全
- hardening
- protection
---

# OpenClaw Security Hardening

一个全面的安全工具包，用于保护 OpenClaw 安装免受通过恶意技能文件、提示注入、数据外泄和工作空间篡改发起的攻击。

## 威胁模型

此技能可抵御：

| 威胁 | 描述 | 工具 |
|--------|-------------|------|
| **Prompt Injection** | 包含覆盖系统提示、忽略安全规则或操纵代理行为指令的恶意技能 | `scan-skills.sh` |
| **Data Exfiltration** | 指示代理将敏感数据（凭证、内存、配置）发送到外部端点的技能 | `audit-outbound.sh` |
| **Skill Tampering** | 初始审查后对已安装技能的未经授权修改 | `integrity-check.sh` |
| **Workspace Exposure** | 权限错误的敏感文件、缺少 .gitignore 规则、不安全的网关配置 | `harden-workspace.sh` |
| **Supply Chain** | 安装包含隐藏恶意模式的新技能 | `install-guard.sh` |

## 快速开始

```bash
# 对所有已安装技能运行完整安全扫描
./scripts/scan-skills.sh

# 审计出站数据流模式
./scripts/audit-outbound.sh

# 初始化完整性基线
./scripts/integrity-check.sh --init

# 加固你的工作空间
./scripts/harden-workspace.sh --fix

# 在安装前检查新技能
./scripts/install-guard.sh /path/to/new-skill/
```

## 工具

### 1. `scan-skills.sh` — 技能文件扫描器

扫描所有已安装的技能文件，查找恶意模式，包括提示注入、数据外泄尝试、可疑 URL、隐藏 unicode、混淆命令和社会工程学。

**用法：**
```bash
# 扫描所有技能目录
./scripts/scan-skills.sh

# 仅扫描特定目录
./scripts/scan-skills.sh --path /path/to/skills/

# 以 JSON 格式输出以用于自动化
./scripts/scan-skills.sh --json

# 显示帮助
./scripts/scan-skills.sh --help
```

**检测内容：**
- 提示注入模式（覆盖指令、新系统提示、管理员覆盖）
- 数据外泄（curl/wget 到外部 URL、发送文件内容）
- 可疑 URL（webhooks、pastebin、requestbin、ngrok 等）
- 可能隐藏指令的 Base64 编码内容
- 隐藏 unicode 字符（零宽空格、RTL 覆盖、同形异义字）
- 对敏感文件的引用（.env、凭证、API 密钥、令牌）
- 修改系统文件（AGENTS.md、SOUL.md）的指令
- 混淆命令（十六进制编码、unicode 转义）
- 社会工程学（"不要告诉用户"、"秘密地"、"不提及"）

**严重级别：**
- 🔴 **CRITICAL** — 可能恶意，需要立即采取行动
- 🟡 **WARNING** — 可疑，手动审查
- 🔵 **INFO** — 值得注意但可能良性

---

### 2. `integrity-check.sh` — 技能完整性监控器

创建所有技能文件的 SHA256 哈希基线，并检测未经授权的修改。

**用法：**
```bash
# 初始化基线（首次运行）
./scripts/integrity-check.sh --init

# 检查更改（定期运行）
./scripts/integrity-check.sh

# 审查更改后更新基线
./scripts/integrity-check.sh --update

# 检查特定目录
./scripts/integrity-check.sh --path /path/to/skills/

# 显示帮助
./scripts/integrity-check.sh --help
```

**报告：**
- ✅ 未更改的文件
- ⚠️ 修改的文件（哈希不匹配）
- 🆕 新文件（不在基线中）
- ❌ 已删除的文件（在基线中但缺失）

**自动化：** 添加到你的 heartbeat 或 cron 以每天运行：
```bash
# 在 HEARTBEAT.md 或 cron 中
0 8 * * * /path/to/scripts/integrity-check.sh 2>&1 | grep -E '(MODIFIED|NEW|REMOVED)'
```

---

### 3. `audit-outbound.sh` — 出站数据流审计器

扫描技能文件中的可能导致数据离开你的机器的模式。

**用法：**
```bash
# 审计所有技能
./scripts/audit-outbound.sh

# 审计特定目录
./scripts/audit-outbound.sh --path /path/to/skills/

# 显示白名单域名
./scripts/audit-outbound.sh --show-whitelist

# 将域名添加到白名单
./scripts/audit-outbound.sh --whitelist example.com

# 显示帮助
./scripts/audit-outbound.sh --help
```

**检测：**
- 嵌入在技能指令中的 HTTP/HTTPS URL
- 对 curl、wget、fetch、web_fetch、browser navigate 的引用
- 电子邮件/消息/webhook 发送指令
- 指令中的原始 IP 地址
- 非白名单外部域名

---

### 4. `harden-workspace.sh` — 工作空间加固器

检查和修复 OpenClaw 工作空间中的常见安全配置错误。

**用法：**
```bash
# 仅检查（报告问题）
./scripts/harden-workspace.sh

# 自动修复安全问题
./scripts/harden-workspace.sh --fix

# 显示帮助
./scripts/harden-workspace.sh --help
```

**检查：**
- 敏感文件（MEMORY.md、USER.md、SOUL.md、凭证）的文件权限
- 敏感模式的 .gitignore 覆盖
- 网关认证配置
- DM 策略设置
- 版本控制文件中的敏感内容

---

### 5. `install-guard.sh` — 安装前安全门

在安装任何新技能之前运行以检查恶意内容。

**用法：**
```bash
# 在安装前检查技能
./scripts/install-guard.sh /path/to/new-skill/

# 严格模式（警告也失败）
./scripts/install-guard.sh --strict /path/to/new-skill/

# 显示帮助
./scripts/install-guard.sh --help
```

**检查：**
- scan-skills.sh 的所有模式
- 脚本中的危险 shell 模式（rm -rf、curl|bash、eval 等）
- 可疑的 npm 依赖项（如果存在 package.json）
- 退出代码 0 = 安全，1 = 可疑（用于 CI/自动化）

---

## 安全规则模板

将 `assets/security-rules-template.md` 复制到你的 `AGENTS.md` 中，为你的代理添加运行时安全规则。这些规则指示代理拒绝提示注入尝试并保护敏感数据。

```bash
cat assets/security-rules-template.md >> /path/to/AGENTS.md
```

## 推荐设置

1. **初始设置：**
   ```bash
   ./scripts/scan-skills.sh              # 扫描现有技能
   ./scripts/audit-outbound.sh           # 审计出站模式
   ./scripts/integrity-check.sh --init   # 创建基线
   ./scripts/harden-workspace.sh --fix   # 修复工作空间问题
   ```

2. **将安全规则从模板添加到 AGENTS.md**

3. **在安装新技能之前：**
   ```bash
   ./scripts/install-guard.sh /path/to/new-skill/
   ```

4. **定期检查**（添加到 heartbeat 或 cron）：
   ```bash
   ./scripts/integrity-check.sh          # 检测篡改
   ./scripts/scan-skills.sh              # 重新扫描新模式
   ```
