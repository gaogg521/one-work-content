---
name: agent-skills-tools
description: Agent Skills 生态系统的安全审计和验证工具。扫描技能包中的凭据泄露、危险文件访问和 Git 历史 secrets，确保安装前安全合规。
license: MIT
metadata: None
openclaw: None
category: security
emoji: 🔒
tags:
- Vault
- AI
- Git
---

# Agent Skills Tools 🔒

Agent Skills 生态系统的安全和验证工具。

## 概述

本技能提供工具来审计和验证 Agent Skills 包中的安全漏洞和标准合规性。

## 工具

### 1. 安全审计工具 (skill-security-audit.sh)

扫描技能包中的常见安全问题：

**检查：**
- 🔐 凭据泄露（硬编码 API 密钥、密码、令牌）
- 📁 危险的文件访问（~/.ssh, ~/.aws, ~/.config）
- 🌐 外部网络请求
- 📋 环境变量使用（推荐做法）
- 🔑 文件权限（credentials.json）
- 📜 Git 历史中的泄露 secrets

**用法：**
```bash
./skill-security-audit.sh path/to/skill
```

**示例输出：**
```
🔒 技能安全审计报告：path/to/skill
==========================================

📋 检查1: 凭据泄露 (API key, password, secret, token)
----------------------------------------
✅ 未发现凭据泄露

📋 检查2: 危险的文件操作 (~/.ssh, ~/.aws, ~/.config)
----------------------------------------
✅ 未发现危险的文件访问

[... more checks ...]

==========================================
🎯 安全审计完成
```

## 背景

eudaemon_0 在 286 个技能中发现了 1 个凭据窃取器。Agent 被训练为乐于助人且信任他人，这使它们容易受到恶意技能的攻击。

这些工具有助于在造成损害之前发现此类漏洞。

## 最佳实践

1. **永远不要硬编码凭据**
   - ❌ `API_KEY="sk_live_abc123..."`
   - ✅ 从环境变量或配置文件读取

2. **使用环境变量**
   ```bash
   export MOLTBOOK_API_KEY="sk_live_..."
   ```
   ```python
   import os
   api_key = os.environ.get('MOLTBOOK_API_KEY')
   ```

3. **检查 Git 历史**
   ```bash
   git log -S 'api_key'
   git-secrets --scan-history
   ```

4. **将敏感文件添加到 .gitignore**
   ```
   credentials.json
   *.key
   .env
   ```

## License

MIT
