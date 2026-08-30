---
name: ggshield-scanner
description: 在 secrets 泄露到 git 之前检测 500+ 种硬编码 secrets（API keys、credentials、tokens）。包装 GitGuardian 的 ggshield CLI。
homepage: https://github.com/GitGuardian/ggshield-skill
metadata: None
clawdbot: None
requires: None
bins:
- ggshield
env:
- GITGUARDIAN_API_KEY
tags:
- Vault
- API
- Git
---

# ggshield Secret Scanner

## 概述

**ggshield** 是一个检测代码库中硬编码 secrets 的 CLI 工具。此 Moltbot skill 为你的 AI agent 带来了 secret 扫描能力。

### 什么是 "Secrets"？

Secrets 是绝不应该提交到版本控制的敏感凭证：
- AWS Access Keys, GCP Service Accounts, Azure credentials
- API tokens (GitHub, Slack, Stripe, etc.)
- Database passwords and connection strings
- Private encryption keys and certificates
- OAuth tokens and refresh tokens
- PayPal/Stripe API keys
- Email server credentials

### 为什么这很重要

一个泄露的 secret 可以：
- 🔓 危害你的基础设施
- 💸 产生巨额云账单（攻击者滥用你的 AWS 账户）
- 📊 暴露客户数据（GDPR/CCPA 违规）
- 🚨 触发安全事件和审计

ggshield 在它们到达你的仓库之前**拦截**这些 secrets。

## 功能

### 可用命令

#### 1. `scan-repo`
扫描整个 git 仓库中的 secrets（包括历史记录）。

```
@clawd scan-repo /path/to/my/project
```

**输出**：
```
🔍 Scanning repository...
✅ Repository clean: 1,234 files scanned, 0 secrets found
```

**检测到 secrets 时的输出**：
```
❌ Found 2 secrets:

- AWS Access Key ID in config/prod.py:42
- Slack API token in .env.backup:8

Use 'ggshield secret ignore --last-found' to ignore, or remove them.
```

#### 2. `scan-file`
扫描单个文件中的 secrets。

```
@clawd scan-file /path/to/config.py
```

#### 3. `scan-staged`
仅扫描已暂存的 git 更改（有用的 pre-commit 检查）。

```
@clawd scan-staged
```

此命令仅在你 `git add` 的更改上运行（快速！）。

#### 4. `install-hooks`
将 ggshield 安装为 git pre-commit hook。

```
@clawd install-hooks
```

安装后，每次提交都会自动扫描：
```
$ git commit -m "Add config"
🔍 Running ggshield pre-commit hook...
❌ Secrets detected! Commit blocked.
Remove the secrets and try again.
```

#### 5. `scan-docker`
扫描 Docker 镜像的 layers 中的 secrets。

```
@clawd scan-docker my-app:latest
```

## 安装

### 前置条件

1. **ggshield CLI**: 通过 pip 安装
   ```bash
   pip install ggshield>=1.15.0
   ```

2. **GitGuardian API Key**: Secret 检测所需
   - 注册：https://dashboard.gitguardian.com (free)
   - 在 Settings 中生成 API key
   - 设置环境变量：

```bash
export GITGUARDIAN_API_KEY="your-api-key-here"
```

3. **Python 3.8+**: ggshield 所需

### 安装 Skill

```bash
clawdhub install ggshield-scanner
```

该 skill 现在在你的 Moltbot workspace 中可用。

### 在你的 Moltbot Workspace 中

启动一个新的 Moltbot 会话以加载该 skill：

```bash
moltbot start
# or via messaging: @clawd list-skills
```

## 使用模式

### 模式 1：推送前（安全检查）

```
Dev: @clawd scan-repo .
Moltbot: ✅ Repository clean. All good to push!

Dev: git push
```

### 模式 2：审计现有仓库

```
Dev: @clawd scan-repo ~/my-old-project
Moltbot: ❌ Found 5 secrets in history!
         - AWS keys in config/secrets.json
         - Database password in docker-compose.yml
         - Slack webhook in .env.example
Moltbot: Recommendation: Rotate these credentials immediately.
         Consider using git-filter-repo to remove from history.
```

### 模式 3：Pre-Commit 强制检查

```
Dev: @clawd install-hooks
Moltbot: ✅ Installed pre-commit hook

Dev: echo "SECRET_TOKEN=xyz" > config.py
Dev: git add config.py
Dev: git commit -m "Add config"
Moltbot: ❌ Pre-commit hook detected secret!
Dev: rm config.py && git reset
Dev: (add config to .gitignore and to environment variables instead)
Dev: git commit -m "Add config" # Now works!
```

### 模式 4：Docker 镜像安全

```
Dev: @clawd scan-docker my-api:v1.2.3
Moltbot: ✅ Docker image clean
```

## 配置

### 环境变量

以下是该 skill 运行所需的：

| Variable | Value | Where to Set |
| :-- | :-- | :-- |
| `GITGUARDIAN_API_KEY` | Your API key from https://dashboard.gitguardian.com | `~/.bashrc` or `~/.zshrc` |
| `GITGUARDIAN_ENDPOINT` | `https://api.gitguardian.com` (default, optional) | Usually not needed |

### 可选的 ggshield 配置

创建 `~/.gitguardian/.gitguardian.yml` 以持久化设置：

```yaml
verbose: false
output-format: json
exit-code: true
```

详情：https://docs.gitguardian.com/ggshield-docs/

## 隐私与安全

### 哪些数据会发送到 GitGuardian？

✅ **仅发送元数据**：

- Secret 模式的哈希值（不是实际的 secret）
- 文件路径（仅相对路径）
- 行号

❌ **永远不会发送**：

- 你的实际 secrets 或凭证
- 文件内容
- 私钥
- 凭证

**参考**：GitGuardian Enterprise 客户可以使用本地扫描，无需向任何外部发送数据。

### Secrets 是如何被检测到的

ggshield 使用：

1. **基于熵的检测**：识别高熵字符串（随机 token）
2. **模式匹配**：查找已知的 secret 格式（AWS key 前缀等）
3. **公开 CVE**：交叉引用已泄露的 secrets
4. **机器学习**：基于泄露 secrets 数据库训练

## 故障排除

### "ggshield: command not found"

ggshield 未安装或不在你的 PATH 中。

**修复**：

```bash
pip install ggshield
which ggshield  # Should return a path
```

### "GITGUARDIAN_API_KEY not found"

环境变量未设置。

**修复**：

```bash
export GITGUARDIAN_API_KEY="your-key"
# For persistence, add to ~/.bashrc or ~/.zshrc:
echo 'export GITGUARDIAN_API_KEY="your-key"' >> ~/.bashrc
source ~/.bashrc
```

### "401 Unauthorized"

API key 无效或已过期。

**修复**：

```bash
# Test the API key
ggshield auth status

# If invalid, regenerate at https://dashboard.gitguardian.com → API Tokens
# Then: export GITGUARDIAN_API_KEY="new-key"
```

### "Slow on large repositories"

扫描 50GB 的 monorepo 需要时间。ggshield 正在执行大量工作。

**变通方法**：

```bash
# Scan only staged changes (faster):
@clawd scan-staged

# Or specify a subdirectory:
@clawd scan-file ./app/config.py
```

## 高级主题

### 忽略误报

有时 ggshield 会将不是 secret 的字符串标记出来（例如，测试 key）：

```bash
# Ignore the last secret found
ggshield secret ignore --last-found

# Ignore all in a file
ggshield secret ignore --path ./config-example.py
```

这会创建 `.gitguardian/config.json` 并添加忽略规则。

### 集成到 CI/CD

你可以将 secret 扫描添加到 GitHub Actions / GitLab CI：

```yaml
# .github/workflows/secret-scan.yml
name: Secret Scan
on: [push]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: pip install ggshield
      - run: ggshield secret scan repo .
        env:
          GITGUARDIAN_API_KEY: ${{ secrets.GITGUARDIAN_API_KEY }}
```

### 企业版：本地扫描

如果你的公司使用 GitGuardian Enterprise，你可以在不将数据发送到云端的情况下进行扫描：

```bash
export GITGUARDIAN_ENDPOINT="https://your-instance.gitguardian.com"
export GITGUARDIAN_API_KEY="your-enterprise-key"
```

## 相关资源

- **ggshield 文档**: https://docs.gitguardian.com/ggshield-docs/
- **GitGuardian Dashboard**: https://dashboard.gitguardian.com (查看所有发现的 secrets)
- **Moltbot Skills**: https://docs.molt.bot/tools/clawdhub
- **Secret Management 最佳实践**: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html

## 支持

- **Bug 报告**: https://github.com/GitGuardian/ggshield-skill/issues
- **问题**: 提交 issue 或在 ClawdHub 上评论
- **ggshield 问题**: https://github.com/GitGuardian/ggshield/issues

## 许可证

MIT License - 参见 LICENSE 文件

## 贡献者

- GitGuardian Team
- [欢迎你的贡献！]

---

**Version**: 1.0.0
**Last updated**: January 2026
**Maintainer**: GitGuardian
