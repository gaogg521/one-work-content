---
name: clinkding
description: 管理 linkding 书签 - 保存 URL、搜索、标记、组织和检索你的个人书签集合。当用户想要保存链接、搜索书签、管理标签或组织他们的阅读列表时使用。
homepage: https://github.com/daveonkels/clinkding
metadata:
  clawdis:
    emoji: 🔖
    requires:
      bins:
      - clinkding
    install:
    - id: homebrew
      kind: brew
      formula: daveonkels/tap/clinkding
      bins:
      - clinkding
      label: Install clinkding (Homebrew)
    - id: go
      kind: go
      module: github.com/daveonkels/clinkding@latest
      bins:
      - clinkding
      label: Install clinkding (Go)
tags:
- 管理
---

# clinkding - Linkding 书签管理器 CLI

一个用于管理 [linkding](https://github.com/sissbruecker/linkding)（一个自托管的书签管理器）中书签的现代 Go CLI。

## 功能

**Linkding** 是一个自托管的书签管理器（类似于 Pocket、Instapaper）。**clinkding** 是允许你从终端或通过 AI 代理管理书签的 CLI。

可以将其视为：
- **稍后阅读** - 捕获你想要阅读的 URL
- **可搜索的库** - 跨标题、描述、标签的全文搜索
- **有序的集合** - 标记和捆绑相关的书签
- **个人档案** - 保留带有注释和元数据的重要链接

## 快速开始

### 初始设置

```bash
# 交互式配置
clinkding config init

# 或手动配置
clinkding config set url https://your-linkding-instance.com
clinkding config set token YOUR_API_TOKEN

# 测试连接
clinkding config test
```

### 配置文件

位置：`~/.config/clinkding/config.yaml`

```yaml
url: https://linkding.example.com
token: your-api-token-here

defaults:
  bookmark_limit: 100
  output_format: auto
```

### 环境变量

```bash
export LINKDING_URL="https://linkding.example.com"
export LINKDING_TOKEN="your-api-token-here"
```

## 核心命令

### 书签

#### 列出与搜索

```bash
# 列出最近的书签
clinkding bookmarks list

# 按关键字搜索
clinkding bookmarks list --query "golang tutorial"

# 按标签过滤
clinkding bookmarks list --query "tag:programming"

# 最近的书签（最近 7 天）
clinkding bookmarks list --added-since "7d"

# 未读的书签
clinkding bookmarks list --query "unread:yes"

# 用于脚本的 JSON 输出
clinkding bookmarks list --json

# 纯文本（制表符分隔）
clinkding bookmarks list --plain
```

#### 创建书签

```bash
# 简单的书签
clinkding bookmarks create https://go.dev

# 带有元数据
clinkding bookmarks create https://go.dev \
  --title "Go Programming Language" \
  --tags "golang,programming,reference" \
  --description "Official Go website" \
  --unread

# 在创建之前检查 URL 是否已存在
clinkding bookmarks check https://go.dev
```

#### 更新书签

```bash
# 更新标题
clinkding bookmarks update 42 --title "New Title"

# 添加标签
clinkding bookmarks update 42 --add-tags "important,work"

# 移除标签
clinkding bookmarks update 42 --remove-tags "old-tag"

# 标记为已读
clinkding bookmarks update 42 --read

# 更新描述
clinkding bookmarks update 42 --description "Updated notes"
```

#### 获取书签详情

```bash
# 完整详情
clinkding bookmarks get 42

# JSON 输出
clinkding bookmarks get 42 --json
```

#### 归档与删除

```bash
# 归档（从主列表中隐藏）
clinkding bookmarks archive 42

# 取消归档
clinkding bookmarks unarchive 42

# 永久删除
clinkding bookmarks delete 42
```

### 标签

```bash
# 列出所有标签
clinkding tags list

# 创建标签
clinkding tags create "golang"

# 获取标签详情
clinkding tags get 1

# 纯文本输出
clinkding tags list --plain
```

### 捆绑包

捆绑包是相关书签的集合。

```bash
# 列出捆绑包
clinkding bundles list

# 创建捆绑包
clinkding bundles create "Go Resources" \
  --description "Everything related to Go programming"

# 更新捆绑包
clinkding bundles update 1 --name "Go Lang Resources"

# 获取捆绑包详情
clinkding bundles get 1

# 删除捆绑包
clinkding bundles delete 1
```

### 资源

上传和管理书签的文件附件。

```bash
# 列出书签的资源
clinkding assets list 42

# 上传文件
clinkding assets upload 42 ~/Documents/screenshot.png

# 下载资源
clinkding assets download 42 1 -o ./downloaded-file.png

# 删除资源
clinkding assets delete 42 1
```

### 用户资料

```bash
# 获取用户资料信息
clinkding user profile
```

## 代理使用模式

### 从对话中保存 URL

```bash
# 用户："Save this for later: https://example.com"
clinkding bookmarks create https://example.com \
  --title "Article Title" \
  --description "Context from conversation" \
  --tags "topic,context"
```

### 搜索书签

```bash
# 用户："Find my golang bookmarks"
clinkding bookmarks list --query "golang"

# 用户："Show me unread programming articles"
clinkding bookmarks list --query "tag:programming unread:yes"

# 用户："What did I save last week?"
clinkding bookmarks list --added-since "7d"
```

### 组织与标记

```bash
# 用户："Tag bookmark 42 as important"
clinkding bookmarks update 42 --add-tags "important"

# 用户："Create a bundle for my AI research links"
clinkding bundles create "AI Research" \
  --description "Machine learning and AI papers"
```

### 检索以供阅读

```bash
# 用户："Give me something to read"
clinkding bookmarks list --query "unread:yes" --limit 5

# 用户："Show me my golang tutorials"
clinkding bookmarks list --query "tag:golang tag:tutorial"
```

## 输出格式

### 自动（默认）
用于终端显示的人性化表格和颜色。

### JSON
```bash
clinkding bookmarks list --json
```
用于脚本和代理解析的机器可读格式。

### 纯文本
```bash
clinkding bookmarks list --plain
```
用于管道友好解析的制表符分隔值。

## 相对日期过滤

支持人性化的时间范围：

```bash
# 最近 24 小时
clinkding bookmarks list --added-since "24h"

# 最近 7 天
clinkding bookmarks list --added-since "7d"

# 最近 6 个月
clinkding bookmarks list --modified-since "180d"
```

**支持的单位：** `h`（小时）、`d`（天）、`y`（年）

## 常见工作流

### 晨读例程

```bash
# 检查未读书签
clinkding bookmarks list --query "unread:yes"

# 获取最近的 5 个
clinkding bookmarks list --limit 5
```

### 从剪贴板保存

```bash
# macOS
pbpaste | xargs -I {} clinkding bookmarks create {}

# Linux
xclip -o | xargs -I {} clinkding bookmarks create {}
```

### 批量操作

```bash
# 标记多个书签
for id in 42 43 44; do
  clinkding bookmarks update $id --add-tags "important"
done

# 归档旧的未读书签
clinkding bookmarks list --query "unread:yes" --added-since "30d" --plain | \
  while read id _; do
    clinkding bookmarks archive "$id"
  done
```

### 备份书签

```bash
# 将所有书签导出为 JSON
clinkding bookmarks list --json > bookmarks-backup-$(date +%Y%m%d).json

# 导出特定标签
clinkding bookmarks list --query "tag:important" --json > important.json
```

## 全局标志

所有命令均可用：

| 标志 | 描述 |
|------|-------------|
| `-c, --config <file>` | 配置文件路径 |
| `-u, --url <url>` | Linkding 实例 URL |
| `-t, --token <token>` | API 令牌 |
| `--json` | 输出为 JSON |
| `--plain` | 输出为纯文本 |
| `--no-color` | 禁用颜色 |
| `-q, --quiet` | 最小输出 |
| `-v, --verbose` | 详细输出 |

## 退出代码

| 代码 | 含义 |
|------|---------|
| 0 | 成功 |
| 1 | 一般错误（API/网络） |
| 2 | 无效用法（错误的标志/参数） |
| 3 | 认证错误 |
| 4 | 未找到 |
| 130 | 中断（Ctrl-C） |

## 故障排除

### 测试配置

```bash
# 验证设置
clinkding config show

# 测试连接
clinkding config test
```

### 常见问题

**认证错误：**
- 在 linkding Web 界面中验证 API 令牌
- 检查 URL 是否包含协议（`https://`）
- 从 URL 中删除尾部斜杠

**特定命令的帮助：**
```bash
clinkding bookmarks --help
clinkding bookmarks create --help
```

## 链接

- **GitHub：** https://github.com/daveonkels/clinkding
- **Linkding：** https://github.com/sissbruecker/linkding
- **Homebrew：** `brew install daveonkels/tap/clinkding`

## 安装

### Homebrew (macOS/Linux)

```bash
brew install daveonkels/tap/clinkding
```

### Go Install

```bash
go install github.com/daveonkels/clinkding@latest
```

### 二进制下载

从 [releases](https://github.com/daveonkels/clinkding/releases) 下载适用于你平台的版本。

## Shell 补全

```bash
# Bash
clinkding completion bash > /etc/bash_completion.d/clinkding

# Zsh
clinkding completion zsh > "${fpath[1]}/_clinkding"

# Fish
clinkding completion fish > ~/.config/fish/completions/clinkding.fish
```

---

**Built by:** [@daveonkels](https://github.com/daveonkels)  
**License:** MIT

## 用于智能书签创建的代理工作流

### 添加带有自动元数据的 URL

当用户说 "Add this to linkding" 或 "Save this URL" 时，请遵循此工作流：

**1. 从 URL 提取元数据**

使用 `summarize` 技能获取标题和描述：

```bash
# 获取页面元数据
summarize url https://example.com --format json
```

这将返回包含以下内容的结构化数据：
- 标题
- 描述/摘要
- 主要内容

**2. 从内容推断适当的标签**

仅映射到**现有的规范标签**。不要创建新标签。

使用此规范标签列表（共 263 个标签）：
- **Tech:** webdev, design, programming, ai, cloud, devops, docker, linux, networking, security, privacy
- **Content:** content, media, photography, video, audio, books, podcasting
- **Business:** business, marketing, ecommerce, finance, career, productivity
- **Home:** smart-home, home-assistant, esphome, iot, home-improvement
- **Tools:** tools, cli, git, github, editor, reference, documentation
- **Data:** data, analytics, mysql, nosql
- **Communication:** communication, email, messaging, slack
- **Education:** education, guide, howto, research, testing
- **Locations:** texas, seattle, dallas (use sparingly)

**标签选择规则：**
- 最多使用 2-5 个标签
- 选择最具体的适用标签
- 如果不确定，默认为更广泛的类别（例如，`tools` 优于 `generator`）
- 首先检查现有标签：`clinkding tags list --plain | grep -i <keyword>`
- 永远不要创建诸如：awesome、cool、interesting、resources、tips 之类的标签

**3. 使用元数据创建书签**

```bash
clinkding bookmarks create "https://example.com" \
  --title "Title from summarize" \
  --description "Summary from summarize (1-2 sentences)" \
  --tags "webdev,tools,reference"
```

### 示例工作流

**用户：** "Save this to linkding: https://github.com/awesome/project"

**代理操作：**

```bash
# 1. 检查是否已添加书签
clinkding bookmarks check https://github.com/awesome/project

# 2. 获取元数据（使用 summarize 技能）
summarize url https://github.com/awesome/project --format json

# 3. 分析内容并推断标签
# 从摘要中："A CLI tool for Docker container management"
# 规范标签：docker, devops, cli, tools

# 4. 创建书签
clinkding bookmarks create https://github.com/awesome/project \
  --title "Awesome Project - Docker Container CLI" \
  --description "Command-line tool for managing Docker containers with enhanced features" \
  --tags "docker,devops,cli"
```

### 标签映射启发式

使用这些规则将内容映射到规范标签：

| 内容类型 | 规范标签 |
|--------------|----------------|
| Web 开发, HTML, CSS, JavaScript | `webdev`, `css`, `javascript` |
| React, 框架, 前端 | `webdev`, `react` |
| 设计, UI/UX, 模型 | `design` |
| Python, Go, Ruby 代码 | `programming`, `python`/`ruby` |
| Docker, K8s, DevOps | `docker`, `devops`, `cloud` |
| 家庭自动化, ESP32, 传感器 | `smart-home`, `esphome`, `iot` |
| AI, ML, LLMs | `ai`, `llm` |
| 生产力工具, 工作流 | `productivity`, `tools` |
| 金融, 投资, 加密 | `finance` |
| 营销, SEO, 广告 | `marketing` |
| 购物, 优惠, 商店 | `ecommerce` |
| 教程, 指南, 文档 | `guide`, `howto`, `documentation` |
| 安全, 隐私, 加密 | `security`, `privacy` |
| 本地 (DFW/Seattle) | `texas`, `seattle` |

### 创建前验证

始终运行这些检查：

```bash
# 1. URL 是否已存在？
clinkding bookmarks check <url>

# 2. 标签是否存在？
clinkding tags list --plain | grep -iE "^(tag1|tag2|tag3)$"

# 3. 我们是否在使用规范标签？
# 对照 263 个规范标签进行交叉引用
# 未经明确用户请求，永远不要创建新标签
```

### 用户请求保存多个链接

如果用户提供多个 URL：

```bash
# 分别处理每个 URL 并提取元数据
for url in url1 url2 url3; do
  # 获取元数据
  # 推断标签
  # 创建书签
done
```

### 更新现有书签

如果用户说 "Update that bookmark" 或 "Add tags to my last save"：

```bash
# 获取最近的书签
recent_id=$(clinkding bookmarks list --limit 1 --plain | cut -f1)

# 添加标签（除非要求，否则不要移除现有的）
clinkding bookmarks update $recent_id --add-tags "new-tag"

# 更新描述
clinkding bookmarks update $recent_id --description "Updated notes"
```

### 关键原则

1. **始终获取元数据** - 使用 `summarize` 获取好的标题/描述
2. **使用现有标签** - 未经检查规范列表，永远不要创建新标签
3. **有选择性** - 最多 2-5 个标签，选择最具体的适用标签
4. **首先验证** - 在创建之前检查重复项
5. **提供上下文** - 包含简要描述，解释为什么它有用

---

## 当前规范标签结构

Dave 的 linkding 实例在从 17,189 个重复项合并后有 **263 个规范标签**。

顶级类别（按书签数量）：
- `pinboard` (4,987) - 旧版导入标签
- `ifttt` (2,639) - 旧版导入标签  
- `webdev` (1,679) - Web 开发
- `design` (561) - 设计/UI/UX
- `content` (416) - 内容/写作
- `cloud` (383) - 云/托管/SaaS
- `business` (364) - 商业/战略
- `ecommerce` (308) - 购物/市场
- `smart-home` (295) - 家庭自动化
- `productivity` (291) - 生产力工具

**黄金法则：** 如有疑问，请使用更广泛的现有标签，而不是创建新的特定标签。
