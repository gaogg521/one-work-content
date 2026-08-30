---
name: dashlane
description: 从 Dashlane vault 访问 passwords、secure notes、secrets 和 OTP codes。
homepage: https://cli.dashlane.com
metadata: None
clawdbot: None
emoji: 🔐
requires: None
bins:
- dcli
tags:
- Vault
---

# Dashlane CLI

从命令行访问你的 Dashlane vault。对 passwords、secure notes、secrets 和 OTP codes 的只读访问。

## 安装

```bash
brew install dashlane/tap/dashlane-cli
```

## 认证

首先 sync 以触发认证：
```bash
dcli sync
```

**步骤：**
1. 输入你的 Dashlane email
2. **重要：在浏览器中打开显示的 URL**（device registration）
3. 输入通过 email 收到的 code
4. 输入你的 Master Password

检查当前 account：
```bash
dcli accounts whoami
```

## 获取 Password

```bash
# 按 URL 或 title 搜索（默认将 password 复制到剪贴板）
dcli p mywebsite
dcli password mywebsite

# 获取特定字段
dcli p mywebsite -f login      # Username/login
dcli p mywebsite -f email      # Email
dcli p mywebsite -f otp        # TOTP 2FA code
dcli p mywebsite -f password   # Password（默认）

# 输出格式
dcli p mywebsite -o clipboard  # 复制到剪贴板（默认）
dcli p mywebsite -o console    # 打印到 stdout
dcli p mywebsite -o json       # 完整 JSON 输出（所有匹配项）

# 按特定字段搜索
dcli p url=example.com
dcli p title=MyBank
dcli p id=xxxxxx               # 按 vault ID
dcli p url=site1 title=site2   # 多个 filters（OR）
```

## 获取 Secure Note

```bash
dcli note [filters]
dcli n [filters]               # 简写

# 按 title 过滤（默认）
dcli n my-note
dcli n title=api-keys

# 输出格式：text（默认）、json
dcli n my-note -o json
```

## 获取 Secret

Dashlane secrets 是用于敏感数据的专用 content type。

```bash
dcli secret [filters]

# 按 title 过滤（默认）
dcli secret api_keys
dcli secret title=api_keys -o json
```

## 其他命令

```bash
# 手动 sync vault（默认每小时自动 sync）
dcli sync

# 锁定 vault（需要 master password 解锁）
dcli lock

# 完全登出
dcli logout

# 将 vault 备份到当前目录
dcli backup
dcli backup --directory /path/to/backup
```

## 配置

```bash
# 在 OS keychain 中保存 master password（默认：true）
dcli configure save-master-password true

# 禁用自动 sync
dcli configure disable-auto-sync true

# 启用 biometrics 解锁（仅限 macOS）
dcli configure user-presence --method biometrics

# 禁用 user presence 检查
dcli configure user-presence --method none
```

## 按平台持久化

### macOS
Master password 默认存储在 **Keychain** 中。重启后仍然保留。
```bash
dcli configure save-master-password true
```

### Linux（server/headless）
没有原生 keychain。选项：
1. **环境变量**（安全性较低，但简单）：
   ```bash
   export DASHLANE_MASTER_PASSWORD="..."
   ```
2. **本地加密文件**：`save-master-password true` 存储在 `~/.local/share/dcli/`
3. **外部 secret manager**（Vault、AWS Secrets 等）注入变量

### Docker / CI
将 `DASHLANE_MASTER_PASSWORD` 环境变量传递给 container。
```bash
docker run -e DASHLANE_MASTER_PASSWORD="..." myimage
```

### SSO / Passwordless
尚不受 dcli 支持 —— 需要 classic master password。

## 高级：注入 Secrets

```bash
# 将 secrets 注入环境变量
dcli exec -- mycommand

# 注入到模板文件
dcli inject < template.txt > output.txt

# 按路径读取 secret
dcli read "dl://vault/secret-id"
```

## 示例

### 获取 2FA 的 OTP
```bash
dcli p github -f otp
# 返回：123456（剩余 25s）
```

### 来自 Vault 的 SSH Keys
将 private key 存储在 secure note 中，然后：
```bash
dcli n SSH_KEY | ssh-add -
```

### 脚本化
```bash
# 为脚本获取 password
PASSWORD=$(dcli p myservice -o console)

# 获取 JSON 并用 jq 解析
dcli p myservice -o json | jq -r '.[0].password'
```

## 故障排查

- **Locked?** 运行 `dcli sync` 解锁
- **SSO users:** 需要安装 Chrome + visual interface
- **Password-less:** 尚不支持
- **Debug mode:** `dcli --debug <command>`

Docs: https://cli.dashlane.com
