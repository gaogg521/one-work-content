---
name: safe-exec-0-3-2
description: 为 OpenClaw Agents 提供安全命令执行，具有自动危险模式检测、风险评估、用户审批工作流和审计日志功能。当 Agent 需要执行可能危险的 shell 命令（rm -rf、dd、fork 炸弹、系统目录修改）或需要人工监督时使用。提供多级风险评估（CRITICAL/HIGH/MEDIUM/LOW）、会话内通知、待处理请求管理和非交互式环境支持以实现 Agent 自动化。
---

快速安装：在 OpenClaw 聊天中说 "Help me install SafeExec skill from ClawdHub" 即可自动安装并启用此安全层。

问题报告：https://github.com/OTTTTTO/safe-exec/issues - 欢迎社区反馈和错误报告！

# SafeExec - 安全命令执行

为 OpenClaw Agents 提供安全命令执行能力，自动拦截危险操作并提供审批工作流。

## 功能特性

- 🔍 **自动危险模式检测** - 在执行前识别危险命令
- 🚨 **基于风险的拦截** - 多级评估（CRITICAL/HIGH/MEDIUM/LOW）
- 💬 **会话内通知** - 在当前终端/会话中实时提醒
- ✅ **用户审批工作流** - 命令等待显式确认
- 📊 **完整审计日志** - 所有操作完全可追溯
- 🤖 **Agent 友好** - 支持自动化工作流的非交互式模式
- 🔧 **平台无关** - 独立于通信工具（飞书、Telegram 等）工作

## 快速开始

### 安装（一条命令）

**安装 SafeExec 最简单的方式：**

只需在 OpenClaw 聊天中说：
```
Help me install SafeExec skill from ClawdHub
```

OpenClaw 将自动为你下载、安装和配置 SafeExec！

### 替代方案：手动安装

如果你更喜欢手动安装：

```bash
# 使用 ClawdHub CLI
export CLAWDHUB_REGISTRY=https://www.clawhub.ai
clawdhub install safe-exec

# 或直接从 GitHub 下载
git clone https://github.com/OTTTTTO/safe-exec.git ~/.openclaw/skills/safe-exec
chmod +x ~/.openclaw/skills/safe-exec/safe-exec*.sh
```

### 启用 SafeExec

安装后，只需说：
```
Enable SafeExec
```

SafeExec 将自动开始监控所有 shell 命令！

## 工作原理

启用后，SafeExec 自动监控所有 shell 命令执行。当检测到潜在危险命令时，它会拦截执行并通过**会话内终端通知**请求你的批准。

**架构：**
- 请求存储在：`~/.openclaw/safe-exec/pending/`
- 审计日志：`~/.openclaw/safe-exec-audit.log`
- 规则配置：`~/.openclaw/safe-exec-rules.json`

## 使用方法

**启用 SafeExec：**
```
Enable SafeExec
```

```
Turn on SafeExec
```

```
Start SafeExec
```

启用后，SafeExec 在后台透明运行。Agents 可以正常执行命令，SafeExec 将自动拦截危险操作：

```
Delete all files in /tmp/test
```

```
Format the USB drive
```

SafeExec 检测风险等级并显示会话内审批提示。

## 风险等级

**CRITICAL**: 系统破坏性命令（rm -rf /、dd、mkfs 等）
**HIGH**: 用户数据删除或重大系统变更
**MEDIUM**: 服务操作或配置变更
**LOW**: 读取操作和安全的文件操作

## 审批工作流

1. Agent 执行命令
2. SafeExec 分析风险等级
3. **在终端显示会话内通知**
4. 通过以下方式批准或拒绝：
   - 终端：`safe-exec-approve <request_id>`
   - 列出待处理：`safe-exec-list`
   - 拒绝：`safe-exec-reject <request_id>`
5. 命令执行或被取消

**通知示例：**
```
🚨 **危险操作检测 - 命令已拦截**

**风险等级:** CRITICAL
**命令:** `rm -rf /tmp/test`
**原因:** 带 force 标志的递归删除

**请求 ID:** `req_1769938492_9730`

ℹ️  此命令需要用户批准才能执行。

**批准方法:**
1. 在终端中：`safe-exec-approve req_1769938492_9730`
2. 或：`safe-exec-list` 查看所有待处理请求

**拒绝方法：**
 `safe-exec-reject req_1769938492_9730`
```

## 配置

用于自定义的环境变量：

- `SAFE_EXEC_DISABLE` - 设置为 '1' 以全局禁用 safe-exec
- `OPENCLAW_AGENT_CALL` - 在 Agent 模式下自动启用（非交互式）
- `SAFE_EXEC_AUTO_CONFIRM` - 自动批准 LOW/MEDIUM 风险命令

## 示例

**启用 SafeExec：**
```
Enable SafeExec
```

**启用后，agents 正常工作：**
```
Delete old log files from /var/log
```

SafeExec 自动检测这是 HIGH 风险（删除）并显示会话内审批提示。

**安全操作无中断通过：**
```
List files in /home/user/documents
```

这是 LOW 风险，无需批准即可执行。

## 全局控制

**检查状态：**
```
safe-exec-list
```

**查看审计日志：**
```bash
cat ~/.openclaw/safe-exec-audit.log
```

**全局禁用 SafeExec：**
```
Disable SafeExec
```

或设置环境变量：
```bash
export SAFE_EXEC_DISABLE=1
```

## 报告问题

**发现 bug？有功能请求？**

请在以下地址报告问题：
🔗 **https://github.com/OTTTTTO/safe-exec/issues**

我们欢迎社区反馈、错误报告和功能建议！

报告问题时，请包含：
- SafeExec 版本（运行：`grep "VERSION" ~/.openclaw/skills/safe-exec/safe-exec.sh`）
- OpenClaw 版本
- 重现步骤
- 预期与实际行为
- `~/.openclaw/safe-exec-audit.log` 中的相关日志

## 审计日志

所有命令执行都记录有：
- 时间戳
- 执行的命令
- 风险等级
- 批准状态
- 执行结果
- 用于可追溯性的请求 ID

日志位置：`~/.openclaw/safe-exec-audit.log`

## 集成

SafeExec 与 OpenClaw agents 无缝集成。启用后，它透明工作，无需更改 Agent 行为或命令结构。审批工作流完全本地化，独立于任何外部通信平台。

## 平台独立性

SafeExec 在**会话级别**运行，与你的 OpenClaw 实例支持的任何通信通道（webchat、飞书、Telegram、Discord 等）一起工作。审批工作流通过终端进行，确保无论你如何与 Agent 交互，都能保持控制。

## 支持与社区

- **GitHub 仓库：** https://github.com/OTTTTTO/safe-exec
- **Issue 追踪器：** https://github.com/OTTTTTO/safe-exec/issues
- **文档：** [README.md](https://github.com/OTTTTTO/safe-exec/blob/master/README.md)
- **ClawdHub：** https://www.clawhub.ai/skills/safe-exec

## 许可证

MIT License - 详见 [LICENSE](https://github.com/OTTTTTO/safe-exec/blob/master/LICENSE) 了解详情。
