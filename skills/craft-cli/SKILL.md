---
name: craft-cli
description: 通过 craft CLI 工具与 Craft Documents 交互，支持列出、搜索、获取、创建和更新文档
tags:
- MongoDB
---

# Craft CLI Skill

通过 `craft` CLI 工具与 Craft Documents 交互。快速、节省令牌、为 LLM 准备。

## 安装

`craft` CLI 二进制文件应安装在 `/usr/local/bin/craft`。

如果未安装：
```bash
curl -L https://github.com/nerveband/craft-cli/releases/download/v1.0.0/craft-darwin-arm64 -o craft
chmod +x craft
sudo mv craft /usr/local/bin/
```

## 配置

两个 Craft 空间可用：

### wavedepth Space (Business)
```bash
~/clawd/skills/craft-cli/craft config set-api https://connect.craft.do/links/5VruASgpXo0/api/v1
```

### Personal Space
```bash
~/clawd/skills/craft-cli/craft config set-api https://connect.craft.do/links/HHRuPxZZTJ6/api/v1
```

### 快速切换 (Helper Script)
```bash
# 切换到 wavedepth space
~/clawd/skills/craft-cli/craft-helper.sh wavedepth

# 切换到 personal space
~/clawd/skills/craft-cli/craft-helper.sh personal

# 检查当前 space
~/clawd/skills/craft-cli/craft-helper.sh current
```

**检查当前配置：**
```bash
~/clawd/skills/craft-cli/craft config get-api
```

## 命令

### 列出文档
```bash
# JSON 格式（默认 - 对 LLM 友好）
~/clawd/skills/craft-cli/craft list

# 人类可读的表格
~/clawd/skills/craft-cli/craft list --format table

# Markdown 格式
~/clawd/skills/craft-cli/craft list --format markdown
```

### 搜索文档
```bash
# 搜索文档
~/clawd/skills/craft-cli/craft search "query terms"

# 使用表格输出
~/clawd/skills/craft-cli/craft search "query" --format table
```

### 获取文档
```bash
# 通过 ID 获取文档 (JSON)
~/clawd/skills/craft-cli/craft get <document-id>

# 保存到文件
~/clawd/skills/craft-cli/craft get <document-id> --output document.md

# 不同格式
~/clawd/skills/craft-cli/craft get <document-id> --format markdown
```

### 创建文档
```bash
# 仅使用标题创建
~/clawd/skills/craft-cli/craft create --title "My New Document"

# 从文件创建
~/clawd/skills/craft-cli/craft create --title "My Document" --file content.md

# 使用内联 markdown 创建
~/clawd/skills/craft-cli/craft create --title "Quick Note" --markdown "# Hello\nThis is content"

# 作为另一个文档的子文档创建
~/clawd/skills/craft-cli/craft create --title "Child Doc" --parent <parent-id>
```

### 更新文档
```bash
# 更新标题
~/clawd/skills/craft-cli/craft update <document-id> --title "New Title"

# 从文件更新
~/clawd/skills/craft-cli/craft update <document-id> --file updated-content.md

# 使用内联 markdown 更新
~/clawd/skills/craft-cli/craft update <document-id> --markdown "# Updated\nNew content"

# 同时更新标题和内容
~/clawd/skills/craft-cli/craft update <document-id> --title "New Title" --file content.md
```

### 删除文档
```bash
~/clawd/skills/craft-cli/craft delete <document-id>
```

### 信息命令
```bash
# 显示 API 信息和最近文档
~/clawd/skills/craft-cli/craft info

# 列出所有可用文档
~/clawd/skills/craft-cli/craft docs
```

### 版本
```bash
~/clawd/skills/craft-cli/craft version
```

## 输出格式

- **json** (默认): 机器可读的 JSON，非常适合 LLM 和脚本
- **table**: 人类可读的表格格式
- **markdown**: Markdown 格式化输出

在配置中设置默认格式，或按命令使用 `--format` 标志。

## API URL 覆盖

为任何命令覆盖配置的 API URL：
```bash
~/clawd/skills/craft-cli/craft list --api-url https://connect.craft.do/links/ANOTHER_LINK/api/v1
```

## 错误处理

CLI 提供清晰的错误消息和退出代码：

- **退出代码 0**: 成功
- **退出代码 1**: 用户错误（无效输入、缺少参数）
- **退出代码 2**: API 错误（服务器端问题）
- **退出代码 3**: 配置错误

常见错误：
- `authentication failed. Check API URL` - 无效/未授权的 API URL
- `resource not found` - 文档 ID 不存在
- `rate limit exceeded. Retry later` - 请求过多
- `no API URL configured. Run 'craft config set-api <url>' first` - 缺少配置

## 使用示例

### 工作流：列出和搜索
```bash
# 列出 wavedepth space 中的所有文档
~/clawd/skills/craft-cli/craft config set-api https://connect.craft.do/links/5VruASgpXo0/api/v1
~/clawd/skills/craft-cli/craft list --format table

# 搜索特定文档
~/clawd/skills/craft-cli/craft search "proposal" --format table
```

### 工作流：创建和更新
```bash
# 创建新文档
~/clawd/skills/craft-cli/craft create --title "Project Notes" --markdown "# Initial notes\n\nStart here."

# 从输出中获取文档 ID，然后更新
~/clawd/skills/craft-cli/craft update <doc-id> --title "Updated Project Notes"

# 验证更新
~/clawd/skills/craft-cli/craft get <doc-id> --format markdown
```

### 工作流：导出文档
```bash
# 获取特定文档并保存到文件
~/clawd/skills/craft-cli/craft get <doc-id> --output exported-notes.md
```

### LLM 集成
```bash
# 获取所有文档为 JSON（通过管道处理）
~/clawd/skills/craft-cli/craft list | jq '.[] | {id, title}'

# 搜索并提取特定字段
~/clawd/skills/craft-cli/craft search "meeting" | jq '.[].title'
```

## 提示

1. **默认为 JSON 格式** 以供 LLM 消费（这是默认设置）
2. **向人类展示结果时使用表格格式**
3. **操作前检查配置**：`craft config get-api`
4. **使用 `craft config set-api <url>` 轻松切换 space**
5. **临时使用 `--api-url` 标志覆盖 API URL**，而不是更改配置

## GitHub 仓库

源代码和文档：https://github.com/nerveband/craft-cli

## 版本

当前版本：1.6.0
