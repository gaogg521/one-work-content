---
name: codebuddy-cli
description: 腾讯 CodeBuddy Code CLI 的安装、配置和使用指南。AI 驱动的终端编程助手，支持自然语言驱动的开发，适用于安装、配置、命令使用和问题排查。
tags:
- AI
---

# CodeBuddy CLI Skill

来自腾讯的 AI 驱动终端编程助手。

## 安装

```bash
# 检查先决条件
node -v  # 需要 Node.js 18+
npm -v

# 全局安装
npm install -g @tencent-ai/codebuddy-code

# 验证
codebuddy --version
```

## 快速开始

1. 导航到项目目录
2. 运行 `codebuddy` 启动交互式会话
3. 选择登录方式：
   - **Google/GitHub**：国际版（Gemini、GPT 模型）
   - **微信**：中国版（DeepSeek 模型）

## CLI 参数

| 参数 | 描述 |
|----------|-------------|
| `codebuddy "<prompt>"` | 执行单个任务 |
| `-y` / `--dangerously-skip-permissions` | 跳过权限确认（仅限沙盒） |
| `-p` / `--print` | 单次执行模式（文件操作需要 `-y`） |
| `--permission-mode <mode>` | `acceptEdits`, `bypassPermissions`, `default`, `plan` |
| `--version` | 显示版本 |

### 示例

```bash
# 交互模式
codebuddy

# 单个任务
codebuddy "帮我优化这个函数的性能"
codebuddy "生成这个 API 的单元测试"
codebuddy "检查这次提交的代码质量"

# 跳过权限（仅限沙盒）
codebuddy -p "Review code quality" -y
```

## 斜杠命令

| 命令 | 描述 |
|---------|-------------|
| `/help` | 显示可用命令 |
| `/status` | 显示账户信息和当前模型 |
| `/login` | 切换账户 |
| `/logout` | 登出 |
| `/clear` | 重置对话历史 |
| `/exit` | 结束会话 |
| `/config` | 打开配置 |
| `/doctor` | 诊断问题 |
| `/cost` | Token 使用统计 |
| `/init` | 生成 CODEBUDDY.md 项目指南 |
| `/memory` | 编辑项目记忆文件 |

在会话期间键入 `?` 以获取键盘快捷键。

## 自定义命令

在以下位置创建 `.md` 文件：
- **项目**：`.codebuddy/commands/`
- **全局**：`~/.codebuddy/commands/`

## 更新

```bash
npm install -g @tencent-ai/codebuddy-code
```

## 安全说明

`--dangerously-skip-permissions` 风险：文件删除、范围蔓延、数据丢失。**切勿在生产环境中使用。**
