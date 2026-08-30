---
name: github-token
description: 使用个人访问令牌（PAT）与GitHub交互。安全、用户可控的访问方式——无需OAuth，无需完整账户权限。支持克隆、推送、分支管理、PR和Issues操作。当用户需要操作GitHub仓库时触发。
---

# GitHub PAT

Interact with GitHub using Personal Access Tokens. User controls access via PAT scopes.

## 设置

User provides their PAT:
```
1. Create PAT at github.com/settings/tokens
2. Select scopes (repo for full, public_repo for public only)
3. Provide token to agent
```

Store in TOOLS.md or pass via `--token`.

## 命令

```bash
# List repos you have access to
python3 scripts/gh.py repos [--token TOKEN]

# Clone a repo
python3 scripts/gh.py clone owner/repo [--token TOKEN]

# Create branch
python3 scripts/gh.py branch <branch-name> [--repo owner/repo]

# Commit and push
python3 scripts/gh.py push "<message>" [--branch branch] [--repo owner/repo]

# Open a pull request
python3 scripts/gh.py pr "<title>" [--body "description"] [--base main] [--head branch]

# Create an issue
python3 scripts/gh.py issue "<title>" [--body "description"] [--repo owner/repo]

# View repo info
python3 scripts/gh.py info owner/repo
```

## Security 模型

- **User controls access** via PAT scopes
- **No OAuth** - no "allow full access" prompts
- **Least privilege** - user creates PAT with minimal needed scopes
- **Fine-grained PATs** supported for specific repo access

## 代币 存储

Agent stores 代币 in TOOLS.md under `### GitHub` 截面. Never expose in logs or messages.