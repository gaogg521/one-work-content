---
name: notebooklm-cli-cookies
description: 使用 nlm CLI 搜索和回答已上传到 NotebookLM 的文档中的问题。当用户要求查找信息、总结来源或查询特定 NotebookLM 笔记本时使用。
---

# NotebookLM CLI

## 目的

当用户想要搜索或询问已存在于 NotebookLM 笔记本中的内容时，使用此技能。

此技能假定：
- `nlm` 已安装 (`notebooklm-mcp-cli` 包)。
- 已为无头运行时预注入身份验证。
- `NOTEBOOKLM_MCP_CLI_PATH` 指向身份验证存储目录。

## 硬性规则（避免错误的工具选择）

当用户提到以下任何内容时，将其视为严格请求以查询 NotebookLM：
- "NotebookLM", "notebooklm"
- "notebook alias", "alias"
- 已知的别名（例如：`tai_lieu_dien`, `nlm_tai_lieu_dien`）

在这些情况下：
- 始终通过 Exec 运行 `nlm` 来回答。不要从内存中回答。
- 除非用户明确要求网络来源，否则不要切换到网络搜索。
- 如果答案不在笔记本中，请说明（基于 `nlm` 输出）。

斜杠命令：
- 如果用户通过 Telegram 中的 `/nlm ...` 调用此技能，将 `/nlm` 后的原始文本视为 `nlm` 参数。
- 始终通过 Exec 精确执行：`nlm <args>`，并返回相关的 stdout。

## 运行时检查

在运行查询之前：

1. 验证身份验证路径是否已配置：
```bash
echo "$NOTEBOOKLM_MCP_CLI_PATH"
```
2. 验证登录状态：
```bash
nlm login --check
```

如果身份验证检查失败，请停止并请求身份验证刷新工作流（不要在 AWS 运行时中运行浏览器登录）。

## 查询工作流

1. 列出笔记本：
```bash
nlm notebook list --json
```
2. 选择笔记本：
- 如果用户提供了笔记本 id，则直接使用。
- 如果用户提供了标题，请从列表输出中将其解析为 `notebook_id`（不要将原始标题传递给 `nlm notebook get/source list/query`）。
- 如果用户提供了别名，请使用该别名。
- 如果含糊不清，请要求用户选择一个笔记本。
3. 查询笔记本：
```bash
nlm notebook query "<notebook_id_or_alias>" "<user_question>"
```
4. 返回答案并包含查询了哪个笔记本。

注意：
- `nlm notebook list` 返回用于显示的标题，但许多其他命令期望笔记本 id (UUID) 或别名。传递标题如 `"tài liệu điện"` 可能返回 null/空结果。
- 如果用户将经常查询同一笔记本，请创建别名并始终使用它（例如：`tai_lieu_dien`）。

## Telegram 提示模板（复制/粘贴）

优先使用以下格式之一以可靠地触发此技能：

1) 强制 CLI 查询：
```text
Chạy lệnh: nlm notebook query tai_lieu_dien "giá của A9N61500 là bao nhiêu? Nếu notebook không có thông tin giá thì trả lờ i: không thấy trong NotebookLM."
```

2) 自然语言但明确：
```text
Trong NotebookLM notebook alias tai_lieu_dien: trả lờ i câu hỏi "giá của A9N61500 là bao nhiêu?". Bắt buộc dùng nlm để truy vấn, không tìm web, không đọc file local.
```

## 输出指南

- 明确说明笔记本身份（标题 + id，如果可用）。
- 如果查询结果为空或含糊，建议改进的后续查询。
- 优先选择基于 NotebookLM 响应的简洁、事实性答案。

## 常见错误

- `Authentication expired` / `401` / `403`：
  - 检查 `NOTEBOOKLM_MCP_CLI_PATH`。
  - 确保 `profiles/default/cookies.json` 和 `profiles/default/metadata.json` 存在。
  - 在 AWS 外部（带浏览器的机器）刷新 cookie，然后重新部署密钥。
- `nlm: command not found`：
  - 安装包：`pipx install notebooklm-mcp-cli`（推荐），或 `uv tool install notebooklm-mcp-cli`。

## 命令参考

```bash
# 列出笔记本
nlm notebook list --json

# 按 id 或别名查询笔记本
nlm notebook query "<notebook_id_or_alias>" "<question>"

# 检查身份验证状态
nlm login --check
```
