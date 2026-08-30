---
name: ultra-agent-stinct
description: 内部调试与编码技能。当你在任务中遇到代码错误、构建失败、堆栈跟踪或意外行为时自动激活。这是修复问题的本能，非用户触发。命令失败、脚本报错或工作出错时，自行调试并修复。
version: 1.2.0
author: grimmjoww
homepage: https://github.com/grimmjoww/ultra-agent-stinct
metadata: None
openclaw: None
emoji: ⚡
os:
- darwin
- linux
- win32
tags:
- AI
---

# Ultra Agent Stinct

你的内部调试本能。当你工作时出现问题，这是你修复它的方式。

## 始终遵循这些（任何你接触代码的时候）

这些规则每次都适用，即使是快速修复。没有例外。

### 安全性
1. **编辑前先读取。** 永远不要不 `read` 就 `edit` —— 需要精确文本匹配否则会失败
2. **`write` 完全覆盖。** 对现有文件的更改使用 `edit`
3. **永远不要不询问就删除。** 优先使用安全删除而非 `rm -rf`
4. **永远不要不询问就 push。** 仅在用户明确说时执行 `git push`
5. **永远不要不询问就 commit。** 仅在请求时 stage 和 commit
6. **备份意识。** 大型重构前，建议 branch 或 stash

### 良好实践
7. **始终验证你的修复。** 每次更改后，重新运行失败的命令或测试。永远不要假设它有效
8. **告诉用户发生了什么。** 修复后，简要解释什么坏了以及你更改了什么
9. **先读取错误。** 不要猜测修复——在接触代码前读取实际的错误消息、堆栈跟踪或测试输出
10. **最小化更改。** 修复 bug，不要重构整个区域。保持 diff 小而聚焦

## 何时激活完整工作流

如果你在任务期间遇到错误，先尝试快速修复，同时遵循上述规则。但如果你：
- **卡住了** —— 你的第一次修复没起作用，同样的错误或新的错误
- **遇到复杂问题** —— 跨多个文件的错误、不熟悉的代码、架构问题
- **需要结构** —— 不确定 bug 在哪里或从哪里开始

然后**激活 Ultra Agent Stinct** —— 逐步遵循下面完整的结构化工作流。

---

## 调试工作流

当你遇到错误或出现问题时：

**1. 复现** —— 运行失败的命令：
```
exec command:"<failing command>" workdir:"<project dir>"
```

**2. 读取错误** —— 解析堆栈跟踪。识别文件 + 行号。

**3. 读取代码** —— 读取相关文件：
```
read path:"<file from stack trace>"
```

**4. 追踪原因** —— 跟踪调用链。读取 imports、依赖、配置。检查：
- 拼写错误、错误的变量名
- 缺少 imports 或依赖
- 类型不匹配、null/undefined 访问
- 错误路径、缺少 env vars
- 条件语句中的逻辑错误

**5. 修复** —— 应用最小且正确的修复：
```
read path:"<file>"
edit path:"<file>" old:"<exact broken code>" new:"<fixed code>"
```

**6. 验证** —— 重新运行原始失败命令。确认修复有效。

**7. 报告** —— 告诉用户什么坏了以及你修复了什么（简要）。然后继续你的原始任务。

## 编写新代码

当你需要创建或修改代码作为任务的一部分时：

**1. 理解项目** —— 检查现有模式：
```
exec command:"ls -la" workdir:"<project dir>"
```
读取 `package.json`、`pyproject.toml`、`Cargo.toml` 或等效文件。匹配现有风格和约定。

**2. 先计划** —— 编写前，概述你要创建的内容。思考结构、依赖、边界情况。

**3. 编写** —— 创建文件：
```
write path:"<new file path>" content:"<complete file content>"
```

**4. 验证** —— 运行它、测试它、确保它在继续前实际有效。

## 运行测试

**1. 找到 test runner：**
- **Node.js**: `npm test` / `npx jest` / `npx vitest`
- **Python**: `pytest` / `python -m unittest`
- **Rust**: `cargo test`
- **Go**: `go test ./...`

**2. 运行测试：**
```
exec command:"<test command>" workdir:"<project>" timeout:120
```

**3. 失败时：** 读取失败测试、读取被测源代码、应用调试工作流。

**4. 成功时：** 报告摘要并继续。

## Git 集成

仅在用户要求 commit、stage 或检查 git 状态时。

```
exec command:"git status" workdir:"<project>"
exec command:"git diff --stat" workdir:"<project>"
exec command:"git add <specific files>" workdir:"<project>"
exec command:"git commit -m '<message>'" workdir:"<project>"
```

详细的 git 工作流请参阅 [references/git-workflow.md](references/git-workflow.md)。

## 生成编码 Agents（繁重任务）

对于大型任务（多文件重构、完整功能、长时间构建），生成后台 agent：

```
exec pty:true workdir:"<project>" background:true command:"claude '<detailed task>'"
```

监控：
```
process action:list
process action:log sessionId:<id>
process action:poll sessionId:<id>
```

何时自行处理 vs 委托，请参阅 [references/escalation-guide.md](references/escalation-guide.md)。

## 跨平台快速参考

| 任务 | macOS/Linux | Windows (Git Bash) |
|------|-------------|-------------------|
| 查找文件 | `find . -name "*.ts" -not -path "*/node_modules/*"` | Same |
| 搜索代码 | `grep -rn "pattern" --include="*.ts" .` | Same |
| 进程列表 | `ps aux \| grep node` | `tasklist \| findstr node` |
| 终止进程 | `kill -9 <PID>` | `taskkill //f //pid <PID>` |
| Python | `python3` (或 `python`) | `python` |
| 打开文件 | `open <file>` | `start <file>` |

## 上下文管理

- 保持 tool calls 聚焦——每个链一个任务
- 不要读取已经在系统 prompt 中的文件
- 对于大文件，读取目标部分而非整个内容
- 如果上下文变重，在继续前总结发现
