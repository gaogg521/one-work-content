---
name: headless-vault-cli
description: 通过 SSH 隧道在您的个人计算机上读取和编辑 Markdown 笔记。当用户要求在他们的 vault 中读取、创建或追加笔记时使用。
homepage: https://github.com/logancyang/headless-vault-cli
metadata: None
moltbot: None
emoji: 🗄️
tags:
- Vault
---

# Headless Vault CLI

通过 SSH 隧道从这台 VPS 托管的 Moltbot 访问您个人计算机上的 Markdown 笔记。

**术语**："Local machine" = 您的个人计算机（macOS 或 Linux），您的笔记所在的位置。此 skill 在 VPS 上运行，并通过反向 SSH 隧道连接到您的机器。

## 可用命令

您只能访问这些命令。不要尝试此处未列出的命令（不存在 rename、delete、move 或 edit 命令）。

| Command | Description |
|---------|-------------|
| `tree` | 列出 vault 目录结构 |
| `resolve` | 通过路径或标题查找笔记 |
| `info` | 获取文件元数据（行数、字节数、sha256、mtime） |
| `read` | 读取笔记内容 |
| `create` | 创建新笔记（如果文件已存在则失败） |
| `append` | 向现有笔记追加内容 |
| `set-root` | 设置 vault 根目录 |

## 如何运行命令

所有命令都通过 SSH 执行：
```bash
ssh -4 -p ${VAULT_SSH_PORT:-2222} ${VAULT_SSH_USER}@${VAULT_SSH_HOST:-localhost} vaultctl <command> [args]
```

始终使用 `-4` 强制使用 IPv4（避免 IPv6 超时问题）。

## 命令参考

### tree - 列出 vault 结构
```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl tree
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl tree --depth 2
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl tree --all
```
Options:
- `--depth N` - 遍历的最大深度
- `--all` - 包含所有文件，不仅仅是 .md

### resolve - 通过路径或标题查找笔记
```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl resolve --title "Meeting Notes"
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl resolve --path "Projects/Plan.md"
```

**对于包含空格的路径/标题**，使用 `--base64`：
```bash
# echo -n "My Meeting Notes" | base64 → TXkgTWVldGluZyBOb3Rlcw==
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl resolve --title TXkgTWVldGluZyBOb3Rlcw== --base64
```

### info - 获取文件元数据
```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl info "Projects/Plan.md"
```
返回 JSON: `{"path": "...", "lines": N, "bytes": N, "sha256": "...", "mtime": N}`

**对于包含空格的路径**，使用 `--base64`：
```bash
# echo -n "Notes/My File.md" | base64 → Tm90ZXMvTXkgRmlsZS5tZA==
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl info Tm90ZXMvTXkgRmlsZS5tZA== --base64
```

### read - 读取笔记内容
```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl read "Projects/Plan.md"
```
返回 JSON: `{"path": "...", "content": "..."}`

**对于包含空格的路径**，使用 `--base64`：
```bash
# echo -n "Notes/My File.md" | base64 → Tm90ZXMvTXkgRmlsZS5tZA==
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl read Tm90ZXMvTXkgRmlsZS5tZA== --base64
```

### create - 创建新笔记
**IMPORTANT**: 对路径和内容都使用 `--base64` flag 进行 base64 编码。这对于包含空格或特殊字符的路径/内容是必需的。

```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl create <base64_path> <base64_content> --base64
```

示例：创建 "Notes/Morning Brief.md" 并设置内容为 "# Hello\n\nWorld"：
```bash
# Encode path: echo -n "Notes/Morning Brief.md" | base64 → Tm90ZXMvTW9ybmluZyBCcmllZi5tZA==
# Encode content: echo -n "# Hello\n\nWorld" | base64 → IyBIZWxsbwoKV29ybGQ=
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl create Tm90ZXMvTW9ybmluZyBCcmllZi5tZA== IyBIZWxsbwoKV29ybGQ= --base64
```

- 自动创建父目录
- 如果文件已存在则失败（使用 `append` 添加到现有文件）
- 文件必须具有 `.md` 扩展名
- **切勿在笔记内容中重复标题作为标题**（例如，对于 "My Note.md"，不要以 "# My Note" 开头内容）

### append - 追加到现有笔记
```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl append <base64_path> <base64_content> --base64
```

- 如果文件不存在则失败（对新文件使用 `create`）

### set-root - 设置 vault 根目录
```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl set-root /path/to/vault
```

## 您不能做什么

不支持这些操作：
- **重命名**文件或文件夹
- **删除**文件或文件夹
- **移动**文件到不同文件夹
- **编辑**文件的特定部分（仅追加到末尾）
- **创建**没有文件的文件夹（文件夹随 `create` 自动创建）

## 环境变量

由 tunnel-setup.sh 自动配置：
- `VAULT_SSH_USER` - 本地机器用户名（自动检测）
- `VAULT_SSH_PORT` - 隧道端口（默认：2222）
- `VAULT_SSH_HOST` - 隧道主机（默认：localhost）

## 提示

- 首先运行 `vaultctl tree` 以查看存在哪些笔记
- 使用 `vaultctl resolve --title "..."` 按名称查找笔记
- 所有输出均为 JSON
- 本地机器必须在线且隧道正在运行
- **对于包含空格的路径**：使用 `--base64` flag 并附带 base64 编码的路径（适用于 `read`、`info`、`create`、`append`）

## 示例

**Important**: 如果不确定存在哪些笔记，始终先运行 `tree`。这可以防止因路径错误或重复名称而导致的错误。

### 示例 1：用户要求读取笔记（先检查）
User: "Show me my project plan"

Step 1 - 检查存在什么：
```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl tree
```
Output:
```json
{"tree": [{"path": "Projects", "type": "dir"}, {"path": "Projects/Plan.md", "type": "file"}]}
```

Step 2 - 现在读取正确的路径：
```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl read "Projects/Plan.md"
```
Output:
```json
{"path": "Projects/Plan.md", "content": "# Project Plan\n\n## Goals\n..."}
```

### 示例 2：用户要求创建笔记（先检查以避免重复）
User: "Create a meeting notes file"

Step 1 - 检查已存在什么：
```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl tree
```
Output:
```json
{"tree": [{"path": "Projects", "type": "dir"}, {"path": "Projects/Plan.md", "type": "file"}]}
```

Step 2 - 不存在 "Meeting Notes"，可以安全创建（不要重复标题作为标题）：
```bash
# echo -n "Meeting Notes.md" | base64 → TWVldGluZyBOb3Rlcy5tZA==
# echo -n "## Agenda\n\n- Item 1\n- Item 2\n" | base64 → IyMgQWdlbmRhCgotIEl0ZW0gMQotIEl0ZW0gMgo=
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl create TWVldGluZyBOb3Rlcy5tZA== IyMgQWdlbmRhCgotIEl0ZW0gMQotIEl0ZW0gMgo= --base64
```
Output:
```json
{"status": "ok", "path": "Meeting Notes.md"}
```

### 示例 3：用户询问 vault 内容
User: "What's in my notes?"

```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl tree --depth 2
```
Output:
```json
{"tree": [{"path": "Projects", "type": "dir"}, {"path": "Projects/Plan.md", "type": "file"}, {"path": "Ideas.md", "type": "file"}]}
```

Then summarize for user: "You have a Projects folder with Plan.md, and an Ideas.md file at the root."

### 示例 4：带有源笔记和输出笔记的复杂工作流
User: "According to the source note 'AI Digest Sources.md', browse the sources and output the digest to 'digest/2025-01-28-digest.md'"

Step 1 - 检查存在什么：
```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl tree
```
Output:
```json
{"tree": [{"path": "AI Digest Sources.md", "type": "file"}, {"path": "digest", "type": "dir"}, {"path": "digest/2025-01-27-digest.md", "type": "file"}]}
```

Step 2 - 验证：
- 源 "AI Digest Sources.md" 存在 ✓
- 输出 "digest/2025-01-28-digest.md" 不存在 → 将使用 `create`

(If source didn't exist: STOP and ask user "I couldn't find 'AI Digest Sources.md'. Did you mean one of these: [list alternatives]?")

(If output already existed: use `append` instead of `create`)

Step 3 - 读取源笔记：
```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl read "AI Digest Sources.md"
```
Output:
```json
{"path": "AI Digest Sources.md", "content": "# AI Digest Sources\n\n- https://example.com/article1\n- https://example.com/article2\n"}
```

Step 4 - 浏览源并生成摘要内容（由 bot 在此 skill 外部完成）

Step 5 - 将输出写入 vault（不要重复标题作为标题）：
```bash
# echo -n "digest/2025-01-28-digest.md" | base64 → ZGlnZXN0LzIwMjUtMDEtMjgtZGlnZXN0Lm1k
# echo -n "## Summary\n\nKey points from today's sources...\n" | base64 → IyMgU3VtbWFyeQoKS2V5IHBvaW50cyBmcm9tIHRvZGF5J3Mgc291cmNlcy4uLgo=
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl create ZGlnZXN0LzIwMjUtMDEtMjgtZGlnZXN0Lm1k IyMgU3VtbWFyeQoKS2V5IHBvaW50cyBmcm9tIHRvZGF5J3Mgc291cmNlcy4uLgo= --base64
```

(If output already existed, use `append` instead:)
```bash
ssh -4 -p 2222 ${VAULT_SSH_USER}@localhost vaultctl append ZGlnZXN0LzIwMjUtMDEtMjgtZGlnZXN0Lm1k IyMgVXBkYXRlCi4uLg== --base64
```
