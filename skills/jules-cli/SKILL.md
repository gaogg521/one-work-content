---
name: jules-cli
description: 与Jules CLI交互以管理异步编码会话。对于受益于远程VM的复杂、孤立任务，请谨慎使用此技能。
binaries:
- jules
- python3
env:
- HOME
tags:
- CLI
- 效率
- 管理
---

# Jules CLI Skill

## 概述
此技能使代理能够与 `jules` CLI交互。它支持任务分配、会话监控和结果集成。

## 使用指南（关键）

为防止过度和不适当的会话创建，你**必须**遵循以下规则：

1.  **本地优先**: 如果你可以在当前环境中本地解决任务（例如，编辑文件、运行测试、小型重构），**不要**使用Jules。
2.  **复杂度阈值**: 仅将Jules用于以下任务：
    *   **大规模**: 涉及许多文件或需要重大架构更改。
    *   **孤立**: 受益于干净的远程环境以避免本地依赖问题。
    *   **探索性**: 解决方案不立即明显且需要在VM中迭代的任务。
3.  **禁止扩散（一次一个）**: 
    *   **切勿**为同一任务创建多个会话。
    *   **切勿**使用循环或并行执行来同时启动多个会话。
    *   等待会话完成并检查结果，然后再决定是否需要另一个会话。
4.  **禁止"小"任务**: 不要提交诸如"添加注释"、"更改变量名"或"修复拼写错误"之类的任务。

---

## 安全指南

为确保CLI命令的安全执行，你**必须**遵守以下安全实践：

1.  **输入验证**: 在运行任何命令之前，验证：
    *   **仓库名称**遵循 `owner/repo` 格式（字母数字、点、连字符和下划线）。
    *   **会话ID**是字母数字（通常也允许连字符和下划线）。
2.  **引用**: 始终将shell占位符用双引号包裹（例如，`"<repo>"`）。
3.  **禁止内联注入**: 切勿将用户提供的数据直接嵌入脚本字符串（如 `python3 -c`）。使用环境变量安全地传递此类数据。
4.  **清理**: 确保任务描述在直接传递给shell时不包含恶意shell字符。

---

## 安全控制
*   **需要批准（强制）**: 你**必须**在运行以下任何命令之前征求用户的明确批准：
    *   `jules remote new`: 因为这会创建远程会话/VM。
    *   `jules remote pull --apply`: 因为这会修改本地代码库。
    *   `jules teleport`: 因为这会克隆并修改环境。
*   **验证**: 在创建新会话之前，始终运行 `jules remote list --session` 以确保你没有同一仓库的待处理会话。
*   **凭据**: 如果需要 `jules login`，向用户解释*原因*并等待他们的确认后再继续。

---

## 核心工作流（手动控制）

优先直接使用CLI以保持态势感知。

### 1. 飞行前检查
验证仓库访问和格式。
```bash
jules remote list --repo
```
*注意：确保仓库格式为 `GITHUB_USERNAME/REPO`。*

### 2. 提交任务
创建会话并捕获会话ID。
```bash
# 捕获输出以获取ID
# 将 <repo> 和任务描述替换为验证后的输入
jules remote new --repo "<repo>" --session "Detailed task description" < /dev/null
```

### 3. 监控进度
列出会话并查找你的ID。使用这个健壮的一行命令检查状态（它处理带空格的状态，如"In Progress"）：

**检查状态（安全方法）:****
```bash
# 使用环境变量将会话ID安全地传递给Python
export JULES_SESSION_ID="<SESSION_ID>"
jules remote list --session | python3 -c "
import sys, re, os
session_id = os.environ.get('JULES_SESSION_ID', '')
if not session_id: sys.exit(0)
for line in sys.stdin:
    line = line.strip()
    if line.startswith(session_id):
        # 提取状态（多个空格后的最后一列）
        print(re.split(r'\s{2,}', line)[-1])
"
unset JULES_SESSION_ID
```

### 4. 集成结果
一旦状态为**已完成**，拉取并应用更改。
```bash
# 将 <SESSION_ID> 替换为验证后的会话ID
jules remote pull --session "<SESSION_ID>" --apply < /dev/null
```

---

## 错误处理和故障排除

*   **仓库未找到**: 使用 `jules remote list --repo` 验证格式。它必须与GitHub路径匹配。
*   **TTY错误**: 对于使用原始 `jules` 命令的非交互式自动化，始终使用 `< /dev/null`。
*   **凭据**: 如果你看到登录错误，确保 `HOME` 设置正确或运行 `jules login`。

---

## 命令参考

| 命令 | 用途 |
| :--- | :--- |
| `jules remote list --repo` | 验证可用仓库及其确切名称。 |
| `jules remote list --session` | 列出活动和过去的会话以检查状态。 |
| `jules remote new` | 创建新的编码任务。 |
| `jules remote pull` | 应用已完成会话的更改。 |
| `jules teleport "<id>"` | 克隆并应用更改（对新环境有用）。 |
