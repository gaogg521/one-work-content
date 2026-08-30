---
name: homebrew
description: 用于 macOS 的 Homebrew 包管理器。搜索、安装、管理和排查软件包与 cask 的问题。
metadata:
  clawdbot:
    emoji: 🍺
    requires:
      bins:
      - brew
---

# Homebrew 包管理器

完整的 Homebrew 命令参考和使用指南，用于在 macOS 上安装、管理和排查软件包问题。

## 何时使用
- 安装软件包或应用程序 (`brew install X`)
- 搜索可用软件包 (`brew search X`)
- 更新和升级现有软件包
- 检查软件包信息和依赖项
- 排查安装问题
- 管理已安装的软件包

## 命令参考

### 软件包搜索与信息

#### `brew search TEXT|/REGEX/`
**用法:** 按名称或正则表达式模式查找软件包
**何时使用:** 当用户要求查找或搜索某个软件包时
**示例:**
```bash
brew search python
brew search /^node/
```

#### `brew info [FORMULA|CASK...]`
**用法:** 显示一个或多个软件包的详细信息
**何时使用:** 在安装之前查看依赖项、选项和详情
**示例:**
```bash
brew info python
brew info chrome google-chrome
```

### 安装与升级

#### `brew install FORMULA|CASK...`
**用法:** 安装一个或多个软件包或应用程序
**何时使用:** 当用户说 "install X" 或 "use brew to install X" 时
**说明:**
- FORMULA = 命令行工具 (安装到 /usr/local/bin)
- CASK = GUI 应用程序 (安装到 /Applications)
- 可以一次安装多个: `brew install git python nodejs`
**示例:**
```bash
brew install python
brew install google-chrome  # 作为 cask 安装
brew install git python nodejs
```

#### `brew update`
**用法:** 获取 Homebrew 和所有 formula 的最新版本
**何时使用:** 当 brew 看起来过时时，或在执行主要操作之前
**说明:** 不会升级软件包，只会更新软件包列表
**示例:**
```bash
brew update
```

#### `brew upgrade [FORMULA|CASK...]`
**用法:** 升级已安装的软件包或特定软件包
**何时使用:** 当用户想要更新到较新版本时
**说明:**
- 不带参数：升级所有过时的软件包
- 带参数：仅升级指定的软件包
**示例:**
```bash
brew upgrade              # 升级所有过时的软件包
brew upgrade python       # 仅升级 python
brew upgrade python git   # 升级多个
```

### 软件包管理

#### `brew uninstall FORMULA|CASK...`
**用法:** 移除已安装的软件包
**何时使用:** 当用户想要移除/删除某个软件包时
**说明:** 可以一次卸载多个
**示例:**
```bash
brew uninstall python
brew uninstall google-chrome
```

#### `brew list [FORMULA|CASK...]`
**用法:** 列出已安装的软件包或特定软件包的文件
**何时使用:** 当用户想要查看已安装的内容或某个软件包包含哪些文件时
**示例:**
```bash
brew list                 # 显示所有已安装的软件包
brew list python          # 显示 python 安装的文件
```

### 配置与故障排查

#### `brew config`
**用法:** 显示 Homebrew 配置和环境信息
**何时使用:** 调试安装问题或检查系统设置时
**显示:**
- 安装路径
- Xcode 位置
- Git 版本
- CPU 架构
**示例:**
```bash
brew config
```

#### `brew doctor`
**用法:** 检查 Homebrew 安装是否存在潜在问题
**何时使用:** 遇到安装问题或错误时
**返回:** 警告和修复建议
**示例:**
```bash
brew doctor
```

#### `brew install --verbose --debug FORMULA|CASK`
**用法:** 以详细输出和调试信息安装
**何时使用:** 标准安装失败且需要详细的错误信息时
**示例:**
```bash
brew install --verbose --debug python
```

### 高级用法

#### `brew create URL [--no-fetch]`
**用法:** 从源代码创建一个新的 formula
**何时使用:** 创建自定义软件包（高级用户）
**选项:**
- `--no-fetch` = 不立即下载源代码
**示例:**
```bash
brew create https://example.com/package.tar.gz
```

#### `brew edit [FORMULA|CASK...]`
**用法:** 编辑 formula 或 cask 定义
**何时使用:** 自定义软件包安装（高级用户）
**示例:**
```bash
brew edit python
```

#### `brew commands`
**用法:** 显示所有可用的 brew 命令
**何时使用:** 了解额外的 brew 功能时
**示例:**
```bash
brew commands
```

#### `brew help [COMMAND]`
**用法:** 获取特定命令的帮助信息
**何时使用:** 需要特定命令的详细帮助时
**示例:**
```bash
brew help install
brew help upgrade
```

## 快速参考

| 任务 | 命令 |
|------|---------|
| 搜索软件包 | `brew search TEXT` |
| 获取软件包信息 | `brew info FORMULA` |
| 安装软件包 | `brew install FORMULA` |
| 安装应用程序 | `brew install CASK` |
| 更新软件包列表 | `brew update` |
| 升级所有软件包 | `brew upgrade` |
| 升级特定软件包 | `brew upgrade FORMULA` |
| 移除软件包 | `brew uninstall FORMULA` |
| 列出已安装的 | `brew list` |
| 检查配置 | `brew config` |
| 故障排查 | `brew doctor` |

## 常见工作流

### 安装新软件包
1. 搜索: `brew search python`
2. 获取信息: `brew info python@3.11`
3. 安装: `brew install python@3.11`

### 排查安装问题
1. 检查配置: `brew config`
2. 运行 doctor: `brew doctor`
3. 使用调试重试: `brew install --verbose --debug FORMULA`

### 维护 Homebrew
1. 更新: `brew update`
2. 查看有哪些已过时: `brew upgrade` (显示将会升级的内容)
3. 全部升级: `brew upgrade`

## 关键概念

**FORMULA:** 命令行工具和库 (例如 python, git, node)
**CASK:** GUI 应用程序 (例如 google-chrome, vscode, slack)
**TAP:** 第三方 formula 仓库 (例如 `brew tap homebrew/cask-versions`)

## 说明
- 所有 brew 命令都需要 Homebrew 已安装
- 从源代码构建需要 Xcode Command Line Tools
- 某些软件包可能会提示输入 sudo 密码
- 不同软件包的安装时间不同
- 软件包名称不区分大小写，但按惯例显示为小写

## 资源
- 官方文档: https://docs.brew.sh
- Formula 文档: https://github.com/Homebrew/homebrew-core
- Cask 文档: https://github.com/Homebrew/homebrew-cask
