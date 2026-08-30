---
name: bw-cli
description: 使用 bw CLI 与 Bitwarden 密码管理器交互。涵盖认证（login/unlock/logout/status）、vault 操作（list/get/create/edit/delete/restore items、folders、attachments、collections）、密码/口令生成、组织管理以及 Send/receive。用于 \"bitwarden\"、\"bw\"、\"password safe\"、\"vaultwarden\"、\"vault\"、\"password manager\"、\"generate password\"、\"get password\"、\"unlock vault\"、\"share send\"。
metadata: None
api-key-docs: https://bitwarden.com/help/personal-api-key/
author: tfm
docs: https://bitwarden.com/help/cli/
docs-md: https://bitwarden.com/help/cli.md
version: 1.9.0
tags:
- API
- Vault
---

# Bitwarden CLI

通过命令行界面与 Bitwarden 交互的完整参考。

**官方文档：** https://bitwarden.com/help/cli/  
**Markdown 版本（供 agent 使用）：** https://bitwarden.com/help/cli.md

## 快速参考

### 安装

```bash
# 原生可执行文件（推荐）
# https://bitwarden.com/download/?app=cli

# npm
npm install -g @bitwarden/cli

# Linux 包管理器
choco install bitwarden-cli  # Windows 通过 Chocolatey
snap install bw              # Linux 通过 Snap
```

### 认证流程（首选：先 unlock）

**标准工作流（unlock-first）：**
```bash
# 1. 先尝试 unlock（快速，最常见的情况）
export BW_SESSION=$(bw unlock --passwordenv BW_PASSWORD --raw 2>/dev/null)

# 2. 只有 unlock 失败时才回退到 login
if [ -z "$BW_SESSION" ]; then
  bw login "$BW_EMAIL" "$BW_PASSWORD"
  export BW_SESSION=$(bw unlock --passwordenv BW_PASSWORD --raw)
fi

# 3. 在任何 vault 操作之前先 sync
bw sync

# 4. 结束会话
bw lock                      # Lock（保持 login 状态）
bw logout                    # 完全登出
```

**替代方案：直接 login 方法**
```bash
bw login                     # 交互式 login（email + password）
bw login --apikey           # API key login（使用 .secrets 中的 BW_CLIENTID/BW_CLIENTSECRET）
bw login --sso              # SSO login
bw unlock                    # 交互式 unlock
bw unlock --passwordenv BW_PASSWORD     # 从已 source 的 .secrets 自动可用
```

## 会话与配置命令

### status

检查认证和 vault 状态：

```bash
bw status
```

返回：`unauthenticated`、`locked` 或 `unlocked`。

### config

配置 CLI 设置：

```bash
# 设置服务器（自托管或区域）
bw config server https://vault.example.com
bw config server https://vault.bitwarden.eu   # EU 云
bw config server                              # 检查当前配置

# 单独的服务 URL
bw config server --web-vault <url> --api <url> --identity <url>
```

### sync

将本地 vault 与服务器同步（在 vault 操作之前始终运行）：

```bash
bw sync                     # 完整同步
bw sync --last             # 仅显示上次同步时间戳
```

### update

检查更新（不会自动安装）：

```bash
bw update
```

### serve

启动 REST API 服务器：

```bash
bw serve --port 8087 --hostname localhost
```

## Vault Item 命令

### list

列出 vault 对象：

```bash
# Items
bw list items
bw list items --search github
bw list items --folderid <id> --collectionid <id>
bw list items --url https://example.com
bw list items --trash                        # 回收站中的 items

# Folders
bw list folders

# Collections
bw list collections                          # 所有 collections
bw list org-collections --organizationid <id>  # 组织 collections

# Organizations
bw list organizations
bw list org-members --organizationid <id>
```

### get

检索单个值或 items：

```bash
# 获取特定字段（按名称或 ID）
bw get password "GitHub"
bw get username "GitHub"
bw get totp "GitHub"                         # 2FA 验证码
bw get notes "GitHub"
bw get uri "GitHub"

# 获取完整 item JSON
bw get item "GitHub"
bw get item <uuid> --pretty

# 其他对象
bw get folder <id>
bw get collection <id>
bw get organization <id>
bw get org-collection <id> --organizationid <id>

# 用于 create 操作的模板
bw get template item
bw get template item.login
bw get template item.card
bw get template item.identity
bw get template item.securenote
bw get template folder
bw get template collection
bw get template item-collections

# 安全
bw get fingerprint <user-id>
bw get fingerprint me
bw get exposed <password>                    # 检查密码是否已泄露

# Attachments
bw get attachment <filename> --itemid <id> --output /path/
```

### create

创建新对象：

```bash
# 创建 folder
bw get template folder | jq '.name="Work"' | bw encode | bw create folder

# 创建 login item
bw get template item | jq \
  '.name="Service" | .login=$(bw get template item.login | jq '.username="user@example.com" | .password="secret"')' \
  | bw encode | bw create item

# 创建 secure note（type=2）
bw get template item | jq \
  '.type=2 | .secureNote.type=0 | .name="Note" | .notes="Content"' \
  | bw encode | bw create item

# 创建 card（type=3）
bw get template item | jq \
  '.type=3 | .name="My Card" | .card=$(bw get template item.card | jq '.number="4111..."')' \
  | bw encode | bw create item

# 创建 identity（type=4）
bw get template item | jq \
  '.type=4 | .name="My Identity" | .identity=$(bw get template item.identity)' \
  | bw encode | bw create item

# 创建 SSH key（type=5）
bw get template item | jq \
  '.type=5 | .name="My SSH Key"' \
  | bw encode | bw create item

# 将文件附加到现有 item
bw create attachment --file ./doc.pdf --itemid <uuid>
```

Item 类型：`1=Login`、`2=Secure Note`、`3=Card`、`4=Identity`、`5=SSH Key`。

### edit

修改现有对象：

```bash
# 编辑 item
bw get item <id> | jq '.login.password="newpass"' | bw encode | bw edit item <id>

# 编辑 folder
bw get folder <id> | jq '.name="New Name"' | bw encode | bw edit folder <id>

# 编辑 item collections
 echo '["collection-uuid"]' | bw encode | bw edit item-collections <item-id> --organizationid <id>

# 编辑 org collection
bw get org-collection <id> --organizationid <id> | jq '.name="New Name"' | bw encode | bw edit org-collection <id> --organizationid <id>
```

### delete

移除对象：

```bash
# 移入回收站（30 天内可恢复）
bw delete item <id>
bw delete folder <id>
bw delete attachment <id> --itemid <id>
bw delete org-collection <id> --organizationid <id>

# 永久删除（不可逆！）
bw delete item <id> --permanent
```

### restore

从回收站恢复：

```bash
bw restore item <id>
```

## 密码生成

### generate

生成密码或口令：

```bash
# 密码（默认：14 个字符）
bw generate
bw generate --uppercase --lowercase --number --special --length 20
bw generate -ulns --length 32

# 口令
bw generate --passphrase --words 4 --separator "-" --capitalize --includeNumber
```

## Send 命令（安全分享）

### send

创建临时分享：

```bash
# Text Send
bw send -n "Secret" -d 7 --hidden "This text vanishes in 7 days"

# File Send
bw send -n "Doc" -d 14 -f /path/to/file.pdf

# 高级选项
bw send --password accesspass -f file.txt
```

### receive

访问收到的 Sends：

```bash
bw receive <url> --password <pass>
```

## 组织命令

### move

将 items 分享到组织：

```bash
echo '["collection-uuid"]' | bw encode | bw move <item-id> <organization-id>
```

### confirm

确认已邀请的成员：

```bash
bw get fingerprint <user-id>
bw confirm org-member <user-id> --organizationid <id>
```

### device-approval

管理设备审批：

```bash
bw device-approval list --organizationid <id>
bw device-approval approve <request-id> --organizationid <id>
bw device-approval approve-all --organizationid <id>
bw device-approval deny <request-id> --organizationid <id>
bw device-approval deny-all --organizationid <id>
```

## 导入与导出

### import

从其他密码管理器导入：

```bash
bw import --formats                          # 列出支持的格式
bw import lastpasscsv ./export.csv
bw import bitwardencsv ./import.csv --organizationid <id>
```

### export

导出 vault 数据：

```bash
bw export                                    # CSV 格式
bw export --format json
bw export --format encrypted_json
bw export --format encrypted_json --password <custom-pass>
bw export --format zip                       # 包含 attachments
bw export --output /path/ --raw              # 输出到文件或 stdout
bw export --organizationid <id> --format json
```

## 工具

### encode

对 create/edit 操作的 JSON 进行 Base64 编码：

```bash
bw get template folder | jq '.name="Test"' | bw encode | bw create folder
```

### generate（密码）

参见 [密码生成](#密码生成)。

### 全局选项

所有命令都可用：

```bash
--pretty                     # 使用制表符格式化 JSON 输出
--raw                        # 返回原始输出
--response                   # JSON 格式化响应
--quiet                      # 无 stdout（用于管道传输 secrets）
--nointeraction             # 不提示输入
--session <key>             # 直接传递 session key
--version                   # CLI 版本
--help                      # 命令帮助
```

## 安全参考

### 安全密码存储（Workspace .secrets）

将主密码存储在工作区根目录的 `.secrets` 文件中并自动加载：

```bash
# 创建 .secrets 文件
mkdir -p ~/.openclaw/workspace
echo "BW_PASSWORD=your_master_password" > ~/.openclaw/workspace/.secrets
chmod 600 ~/.openclaw/workspace/.secrets

# 添加到 .gitignore
echo ".secrets" >> ~/.openclaw/workspace/.gitignore

# 在 shell 配置中自动 source（运行一次）
echo 'source ~/.openclaw/workspace/.secrets 2>/dev/null' >> ~/.bashrc
# 或对于 zsh：
echo 'source ~/.openclaw/workspace/.secrets 2>/dev/null' >> ~/.zshrc
```

**现在 BW_PASSWORD 始终可用：**

```bash
bw unlock --passwordenv BW_PASSWORD
```

**安全要求：**
- 文件权限必须为 `600`（仅用户可读/写）
- 必须将 `.secrets` 添加到 `.gitignore`
- 永远不要提交 .secrets 文件
- 自动 source 发生在新的 shell 会话中；对当前会话运行 `source ~/.openclaw/workspace/.secrets`

### API Key 认证（Workspace .secrets）

对于自动化/API key login，将凭证存储在同一个 `.secrets` 文件中：

```bash
# 将 API 凭证添加到 .secrets
echo "BW_CLIENTID=user.xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" >> ~/.openclaw/workspace/.secrets
echo "BW_CLIENTSECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" >> ~/.openclaw/workspace/.secrets
chmod 600 ~/.openclaw/workspace/.secrets
```

**使用 API key login：**

```bash
bw login --apikey
```

**⚠️ 已知问题 / 变通方法**

在某些自托管的 Vaultwarden 实例上，`bw login --apikey` 可能会失败并显示：
```
User Decryption Options are required for client initialization
```

**变通方法 - 使用 Email/Password Login：**

```bash
# 将 EMAIL 添加到 .secrets
echo "BW_EMAIL=your@email.com" >> ~/.openclaw/workspace/.secrets

# 使用 email + password login（而不是 --apikey）
bw login "$BW_EMAIL" "$BW_PASSWORD"

# 或作为一行命令
set -a && source ~/.openclaw/workspace/.secrets && set +a && bw login "$BW_EMAIL" "$BW_PASSWORD"

# 然后像往常一样 unlock
bw unlock --passwordenv BW_PASSWORD
```

**完整工作流（自托管推荐）：**

```bash
# Source .secrets 文件
set -a && source ~/.openclaw/workspace/.secrets && set +a

# 先尝试 unlock（更快，如果已经 login 则有效）
export BW_SESSION=$(bw unlock --passwordenv BW_PASSWORD --raw 2>/dev/null)

# 只有 unlock 失败时才 login（vault 未初始化）
if [ -z "$BW_SESSION" ]; then
  bw login "$BW_EMAIL" "$BW_PASSWORD"
  export BW_SESSION=$(bw unlock --passwordenv BW_PASSWORD --raw)
fi

# 准备就绪
bw sync
bw list items
```

**获取你的 API key：** https://bitwarden.com/help/personal-api-key/

### 环境变量

```bash
BW_CLIENTID                  # API key client_id
BW_CLIENTSECRET              # API key client_secret
BW_PASSWORD                  # 用于 unlock 的主密码
BW_SESSION                   # Session key（CLI 自动使用）
BITWARDENCLI_DEBUG=true      # 启用调试输出
NODE_EXTRA_CA_CERTS          # 自签名证书路径
BITWARDENCLI_APPDATA_DIR     # 自定义配置目录
```

### 两步登录方法

方法值：`0=Authenticator`、`1=Email`、`3=YubiKey`。

```bash
bw login user@example.com password --method 0 --code 123456
```

### URI 匹配类型

值：`0=Domain`、`1=Host`、`2=Starts With`、`3=Exact`、`4=Regex`、`5=Never`。

### 字段类型

值：`0=Text`、`1=Hidden`、`2=Boolean`。

### 组织用户类型

`0=Owner`、`1=Admin`、`2=User`、`3=Manager`、`4=Custom`。

### 组织用户状态

`0=Invited`、`1=Accepted`、`2=Confirmed`、`-1=Revoked`。

## 最佳实践

1. **先 unlock，仅在需要时 login**：先尝试 `bw unlock`，因为它更快；只有 unlock 失败（vault 未初始化）时才运行 `bw login`
2. **始终 sync**：在任何 vault 操作之前运行 `bw sync`
3. **安全会话**：完成后使用 `bw lock`
4. **保护 secrets**：永远不要记录 BW_SESSION 或 BW_PASSWORD
5. **安全存储**：保持 .secrets 文件权限为 600，永远不要提交它
6. **自动 source**：添加到 ~/.bashrc 或 ~/.zshrc 以持久化环境变量
7. **验证指纹**：在确认组织成员之前

## 故障排除

| 问题 | 解决方案 |
|-------|----------|
| "Bot detected" | 使用 `--apikey` 或提供 `client_secret` |
| "Vault is locked" | 运行 `bw unlock` 并 export BW_SESSION |
| 自签名证书错误 | 设置 `NODE_EXTRA_CA_CERTS` |
| 需要调试信息 | `export BITWARDENCLI_DEBUG=true` |

---

**参考：**
- HTML 文档：https://bitwarden.com/help/cli/
- Markdown（可获取）：https://bitwarden.com/help/cli.md
- 个人 API Key：https://bitwarden.com/help/personal-api-key/
