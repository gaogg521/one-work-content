---
name: gtasks-cli
description: 从命令行管理 Google Tasks - 查看、创建、更新、删除任务和任务列表。当用户要求与 Google Tasks 交互、管理待办事项、创建任务列表、标记任务完成或检查其 Google Tasks 时使用。
license: MIT
compatibility: 需要安装并认证 gtasks CLI 工具
metadata:
  author: BRO3886
  version: '1.0'
allowed-tools: Bash(gtasks:*)
tags:
- 效率
- CLI
- 管理
---

# Google Tasks CLI Skill

此 skill 使您能够使用 `gtasks` CLI 工具直接从命令行管理 Google Tasks。

## 先决条件

在使用任何命令之前，请确保满足以下要求：

### 1. GTasks 安装

检查系统上是否安装了 gtasks：

```bash
# Cross-platform check (works on macOS, Linux, Windows Git Bash)
gtasks --version 2>/dev/null || gtasks.exe --version 2>/dev/null || echo "gtasks not found"

# Or use which/where commands
# macOS/Linux:
which gtasks

# Windows (Command Prompt):
where gtasks

# Windows (PowerShell):
Get-Command gtasks
```

**如果未安装 gtasks：**

1. 从 [GitHub Releases](https://github.com/BRO3886/gtasks/releases) 下载适合您系统的二进制文件
2. 安装它：
   - **macOS/Linux**：移动到 `/usr/local/bin` 或添加到 PATH
   - **Windows**：添加到 PATH 环境变量中的文件夹
3. 验证安装：`gtasks --version`

**IMPORTANT for Agents:** 在尝试使用之前始终检查 gtasks 是否已安装。如果找不到命令，请告知用户并提供安装说明。

### 2. 环境变量

将 Google OAuth2 凭据设置为环境变量：

```bash
export GTASKS_CLIENT_ID="your-client-id.apps.googleusercontent.com"
export GTASKS_CLIENT_SECRET="your-client-secret"
```

**如何获取凭据：**
1. 前往 [Google Cloud Console](https://console.cloud.google.com/)
2. 创建新项目或选择现有项目
3. 启用 Google Tasks API
4. 创建 OAuth2 凭据（应用类型："Web application"）
5. 添加授权的 redirect URI：
   - `http://localhost:8080/callback`
   - `http://localhost:8081/callback`
   - `http://localhost:8082/callback`
   - `http://localhost:9090/callback`
   - `http://localhost:9091/callback`

**对于持久化设置**，将这些添加到您的 shell profile（`~/.bashrc`, `~/.zshrc` 等）：

```bash
echo 'export GTASKS_CLIENT_ID="your-client-id"' >> ~/.bashrc
echo 'export GTASKS_CLIENT_SECRET="your-client-secret"' >> ~/.bashrc
source ~/.bashrc
```

### 2. 认证

设置环境变量后，使用 Google 进行认证：

```bash
gtasks login
```

这将打开浏览器进行 OAuth2 认证。Token 存储在 `~/.gtasks/token.json` 中。

## 核心概念

- **Task Lists**：容纳任务的容器（如 "Work", "Personal", "Shopping"）
- **Tasks**：任务列表中的单个待办事项
- **Task Properties**：标题（必需）、备注/描述（可选）、截止日期（可选）、状态（pending/completed）

## 命令结构

所有命令遵循以下模式：
```
gtasks [command] [subcommand] [flags] [arguments]
```

## 认证

### 登录
```bash
gtasks login
```
打开浏览器进行 Google OAuth2 认证。在使用任何其他命令之前是必需的。

### 登出
```bash
gtasks logout
```
从 `~/.gtasks/token.json` 中移除已存储的凭据。

## 任务列表管理

### 查看所有任务列表
```bash
gtasks tasklists view
```
显示所有带有编号索引的任务列表。

**输出示例：**
```
[1] My Tasks
[2] Work
[3] Personal
```

### 创建任务列表
```bash
gtasks tasklists add -t "Work Projects"
gtasks tasklists add --title "Shopping List"
```
创建具有指定标题的新任务列表。

**Flags:**
- `-t, --title`: 任务列表标题（必需）

### 删除任务列表
```bash
gtasks tasklists rm
```
交互式提示以选择并删除任务列表。

### 更新任务列表标题
```bash
gtasks tasklists update -t "New Title"
```
交互式提示以选择任务列表并更新其标题。

**Flags:**
- `-t, --title`: 任务列表的新标题（必需）

## 任务管理

所有任务命令都可以选择使用 `-l` flag 指定任务列表。如果省略，系统将提示您进行交互式选择。

### 查看任务

**基本查看：**
```bash
gtasks tasks view
gtasks tasks view -l "Work"
```

**包含已完成的任务：**
```bash
gtasks tasks view --include-completed
gtasks tasks view -i
```

**仅显示已完成的任务：**
```bash
gtasks tasks view --completed
```

**排序任务：**
```bash
gtasks tasks view --sort=due        # 按截止日期排序
gtasks tasks view --sort=title      # 按标题排序
gtasks tasks view --sort=position   # 按位置排序（默认）
```

**输出格式：**
```bash
gtasks tasks view --format=table    # 表格格式（默认）
gtasks tasks view --format=json     # JSON 输出
gtasks tasks view --format=csv      # CSV 输出
```

**表格输出示例：**
```
Tasks in Work:
No  Title              Description         Status     Due
1   Finish report      Q4 analysis         pending    25 December 2024
2   Team meeting       Weekly sync         pending    -
3   Code review        PR #123            completed  20 December 2024
```

**JSON 输出示例：**
```json
[
  {
    "number": 1,
    "title": "Finish report",
    "description": "Q4 analysis",
    "status": "pending",
    "due": "2024-12-25"
  }
]
```

### 创建任务

**交互模式：**
```bash
gtasks tasks add
gtasks tasks add -l "Work"
```
提示输入标题、备注和截止日期。

**Flag 模式：**
```bash
gtasks tasks add -t "Buy groceries"
gtasks tasks add -t "Finish report" -n "Q4 analysis" -d "2024-12-25"
gtasks tasks add -t "Call dentist" -d "tomorrow"
gtasks tasks add -t "Team meeting" -d "Dec 25"
```

**Flags:**
- `-t, --title`: 任务标题（非交互模式下必需）
- `-n, --note`: 任务备注/描述（可选）
- `-d, --due`: 截止日期（可选，灵活格式）

**日期格式示例：**
日期解析器支持多种格式：
- `2024-12-25` (ISO format)
- `Dec 25, 2024`
- `December 25`
- `tomorrow`
- `next Friday`
- `12/25/2024`

有关所有支持的格式，请参见 [dateparse examples](https://github.com/araddon/dateparse#extended-example)。

### 标记任务为完成

**使用任务编号：**
```bash
gtasks tasks done 1
gtasks tasks done 3 -l "Work"
```

**交互模式：**
```bash
gtasks tasks done
gtasks tasks done -l "Personal"
```
提示从列表中选择一个任务。

### 删除任务

**使用任务编号：**
```bash
gtasks tasks rm 2
gtasks tasks rm 1 -l "Shopping"
```

**交互模式：**
```bash
gtasks tasks rm
gtasks tasks rm -l "Work"
```
提示选择一个任务进行删除。

### 查看任务详情

**使用任务编号：**
```bash
gtasks tasks info 1
gtasks tasks info 3 -l "Work"
```

**交互模式：**
```bash
gtasks tasks info
gtasks tasks info -l "Personal"
```

**输出示例：**
```
Task: Finish report
Status: Needs action
Due: 25 December 2024
Notes: Complete Q4 analysis and submit to manager

Links:
  - https://docs.google.com/document/d/...

View in Google Tasks: https://tasks.google.com/...
```

## 常见工作流

### 快速创建任务
当用户说 "add a task to my work list"：
```bash
gtasks tasks add -l "Work" -t "Task title"
```

### 检查今日任务
```bash
gtasks tasks view --sort=due
```

### 完成多个任务
```bash
gtasks tasks done -l "Work"
# Interactive prompt appears, select task
gtasks tasks done -l "Work"
# Repeat as needed
```

### 查看所有列表中的任务
为每个列表多次运行 view 命令，或首先列出所有任务列表：
```bash
gtasks tasklists view
gtasks tasks view -l "Work"
gtasks tasks view -l "Personal"
```

### 导出任务
```bash
gtasks tasks view --format=json > tasks.json
gtasks tasks view --format=csv > tasks.csv
```

## 最佳实践

1. **始终首先检查认证**：如果命令因认证错误而失败，请运行 `gtasks login`

2. **为自动化使用任务列表 flag**：在脚本编写或用户指定列表名称时，使用 `-l` flag 以避免交互式提示

3. **利用灵活的日期解析**：`--due` flag 接受自然语言日期，如 "tomorrow", "next week" 等

4. **使用适当的输出格式**：
   - 表格格式用于人类可读输出
   - JSON 用于解析/与其他工具集成
   - CSV 用于电子表格导入

5. **任务编号是临时的**：添加、完成或删除任务时，任务编号会改变。始终先查看列表以获取当前编号。

6. **优雅地处理缺失的列表**：如果用户指定了不存在的列表名称，命令将报错。始终先使用 `gtasks tasklists view` 验证列表名称。

## 错误处理

常见错误和解决方案：

- **"Failed to get service"** 或 **认证错误**：
  - 首先，确保环境变量已设置：`echo $GTASKS_CLIENT_ID`
  - 如果变量未设置，请导出它们（参见先决条件部分）
  - 然后运行 `gtasks login` 进行认证
- **"incorrect task-list name"**：指定的列表名称不存在。使用 `gtasks tasklists view` 查看可用列表
- **"Incorrect task number"**：任务编号无效。使用 `gtasks tasks view` 查看当前任务编号
- **"Date format incorrect"**：日期字符串无法解析。使用格式如 "2024-12-25", "tomorrow", 或 "Dec 25"

## 示例

### 示例 1：创建购物清单并添加项目
```bash
gtasks tasklists add -t "Shopping"
gtasks tasks add -l "Shopping" -t "Milk"
gtasks tasks add -l "Shopping" -t "Bread"
gtasks tasks add -l "Shopping" -t "Eggs"
```

### 示例 2：查看并完成工作任务
```bash
gtasks tasks view -l "Work" --sort=due
gtasks tasks done 1 -l "Work"
```

### 示例 3：添加带截止日期的任务
```bash
gtasks tasks add -l "Work" -t "Submit proposal" -n "Include budget and timeline" -d "next Friday"
```

### 示例 4：导出已完成的任务
```bash
gtasks tasks view --completed --format=json -l "Work" > completed_work.json
```

## Agents 提示

### 运行任何命令之前

1. **首先检查 gtasks 安装**：
   ```bash
   # Try to run gtasks version check
   gtasks --version 2>/dev/null || gtasks.exe --version 2>/dev/null
   ```
   如果失败，请告知用户 gtasks 未安装，并提供先决条件部分中的安装说明。

2. **验证环境变量是否已设置**：
   ```bash
   # Check if variables exist (macOS/Linux)
   [ -n "$GTASKS_CLIENT_ID" ] && echo "GTASKS_CLIENT_ID is set" || echo "GTASKS_CLIENT_ID is not set"
   [ -n "$GTASKS_CLIENT_SECRET" ] && echo "GTASKS_CLIENT_SECRET is set" || echo "GTASKS_CLIENT_SECRET is not set"

   # Windows PowerShell
   if ($env:GTASKS_CLIENT_ID) { "GTASKS_CLIENT_ID is set" } else { "GTASKS_CLIENT_ID is not set" }
   if ($env:GTASKS_CLIENT_SECRET) { "GTASKS_CLIENT_SECRET is set" } else { "GTASKS_CLIENT_SECRET is not set" }
   ```

3. **检查认证状态**：
   ```bash
   gtasks tasklists view &>/dev/null && echo "Authenticated" || echo "Not authenticated - run 'gtasks login'"
   ```

### 通用提示

- 当用户提到 "tasks" 而没有指定工具时，询问他们是否想使用 Google Tasks
- 如果用户询问他们的任务，首先运行 `gtasks tasklists view` 查看可用列表
- 如果用户未指定，始终确认要使用哪个任务列表
- 创建带有日期的任务时，优先使用显式日期格式 (YYYY-MM-DD) 而不是相对术语，以确保清晰
- 记住任务编号是从 1 开始的，并且在修改后会发生变化
- 如果命令需要交互但您正在非交互式运行，请使用 flags 提供所有必需的信息
