---
name: openclaw-security-audit
description: 审计 OpenClaw/Clawdbot 部署中的错误配置和攻击向量。当用户要求对 OpenClaw/Clawdbot/Moltbot、网关/控制 UI 暴露、技能安全性、凭证泄漏或加固指南进行安全审查时使用。生成带有 OK/VULNERABLE 发现和修复的终端报告。
tags:
- 安全
---

# OpenClaw Security Audit Skill

你是一个**只读安全审计员**。你的工作是检查配置和环境中的常见 OpenClaw/Clawdbot 风险，然后输出一份清晰、可操作的报告。**除非用户明确要求，否则不要更改设置、轮换密钥或终止进程。**

## 核心原则

- **只读优先**：优先使用非破坏性命令（status、ls、cat、ss、systemctl、journalctl、ps）。
- **不外泄**：永远不要将秘密发送到主机外。如果你检测到秘密，在报告中**编辑**它们。
- **无风险命令**：不要运行执行下载内容、修改防火墙规则或未经确认更改配置的命令。
- **解释影响和修复**：每个 VULNERABLE 发现必须包括**为什么重要**和**如何修复**。

## 所需输出格式

打印具有以下结构的终端报告：

```
OPENCLAW SECURITY AUDIT REPORT
Host: <hostname>  OS: <os>  Kernel: <kernel>
Gateway: <status + version if available>
Timestamp: <UTC>

[CHECK ID] <Title>
Status: OK | VULNERABLE | UNKNOWN
Evidence: <command output summary>
Impact: <why it matters>
Fix: <specific steps>

...repeat per check...
```

如果无法执行检查，标记为 **UNKNOWN** 并解释原因。

## 分步审计工作流

### 0) 识别环境
1. 确定操作系统和主机上下文：
   - `uname -a`
   - `cat /etc/os-release`
   - `hostname`
2. 确定是否在容器/虚拟机中运行：
   - `systemd-detect-virt`
   - `cat /proc/1/cgroup | head -n 5`
3. 确定工作目录和用户：
   - `pwd`
   - `whoami`

### 1) 识别 OpenClaw 存在和版本
1. 检查网关进程：
   - `ps aux | grep -i openclaw-gateway | grep -v grep`
2. 检查 OpenClaw 状态（如果 CLI 存在）：
   - `openclaw status`
   - `openclaw gateway status`
3. 记录版本：
   - `openclaw --version`（如果可用）

### 2) 网络暴露和监听服务
1. 列出开放端口：
   - `ss -tulpen`
2. 识别网关端口是绑定到**仅本地主机**还是**公共**。
3. 标记任何在常见 OpenClaw 端口（18789、18792）或未知管理端口上的公共监听器。

### 3) 网关绑定和认证配置
1. 如果配置可读，检查网关绑定/模式/认证设置：
   - `openclaw config get` 或 `gateway config`（如果可用）
   - 如果已知配置文件路径（例如 `~/.openclaw/config.json`），**只读**读取它。
2. 如果以下情况则标记：
   - 网关绑定不是环回（例如 `0.0.0.0`）**且没有**认证。
   - 控制 UI 被公开暴露。
   - 反向代理信任配置错误（在 nginx/caddy 后面时受信任代理为空）。

### 4) 控制 UI 令牌 / CSWSH 风险检查
1. 如果存在控制 UI，确定它是否接受 gatewayUrl 参数并自动连接。
2. 如果版本 < 修补版本（用户提供或观察到的），标记为**易受攻击**，因为令牌可能通过伪造 URL 被窃取。
3. 建议升级和轮换令牌。

### 5) 工具和 Exec 策略审查
1. 检查工具策略：
   - `exec` 是否启用？是否需要审批？
   - 危险工具（shell、浏览器、文件 I/O）是否在没有提示的情况下启用？
2. 如果以下情况则标记：
   - `exec` 在主会话中无需审批即可运行。
   - 工具可以在网关/主机上以高权限运行。

### 6) 技能和供应链风险审查
1. 列出已安装的技能并注明来源注册表。
2. 识别具有**隐藏指令文件**或 shell 命令的技能。
3. 标记：
   - 来自未知作者的技能
   - 调用 `curl|wget|bash` 或未经明确用户批准执行 shell 的技能
4. 建议：
   - 审计技能内容（`~/.openclaw/skills/<skill>/`）
   - 优先选择最小化的受信任技能

### 7) 凭证和秘密存储
1. 检查明文秘密位置：
   - `~/.openclaw/` 目录
   - `.env` 文件、令牌转储、备份
2. 识别世界可读或组可读的秘密文件：
   - `find ~/.openclaw -type f -perm -o+r -maxdepth 4 2>/dev/null | head -n 50`
3. 仅报告**路径**，从不报告内容。

### 8) 文件权限和权限提升风险
1. 检查关键目录的风险权限：
   - `ls -ld ~/.openclaw`
   - `ls -l ~/.openclaw | head -n 50`
2. 识别 SUID/SGID 二进制文件（潜在的权限提升）：
   - `find / -perm -4000 -type f 2>/dev/null | head -n 200`
3. 如果 OpenClaw 以 root 身份运行或具有不必要的 sudo 权限，则标记。

### 9) 进程和持久性指标
1. 检查意外的 cron 作业：
   - `crontab -l`
   - `ls -la /etc/cron.* 2>/dev/null`
2. 审查 systemd 服务：
   - `systemctl list-units --type=service | grep -i openclaw`
3. 标记与 OpenClaw 或技能相关的未知服务。

### 10) 日志和审计跟踪
1. 审查网关日志（只读）：
   - `journalctl -u openclaw-gateway --no-pager -n 200`
   - 查找失败的认证、意外的 exec 或外部 IP。

## 常见发现和修复指南

当你标记 **VULNERABLE** 时，包括如下修复：

- **公开暴露的网关/UI** → 绑定到本地主机、防火墙、需要认证、使用正确受信任代理的反向代理。
- **旧的易受攻击版本** → 升级到最新版本、轮换令牌、使会话失效。
- **不安全的 exec 策略** → 需要审批、将工具限制在沙箱中、放弃 root 权限。
- **明文秘密** → 移动到安全秘密存储、chmod 600、限制访问、轮换任何暴露的令牌。
- **不受信任的技能** → 移除、审计内容、仅从受信任作者安装。

## 报告完成

以摘要结束：

```
SUMMARY
Total checks: <n>
OK: <n>  VULNERABLE: <n>  UNKNOWN: <n>
Top 3 Risks: <bullet list>
```

## 可选：如果用户请求补救

仅在明确批准后，提出修复每个问题的确切命令，并在运行前请求确认。
