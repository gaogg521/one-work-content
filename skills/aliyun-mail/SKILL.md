---
name: aliyun-mail
description: 通过阿里云企业邮箱服务发送邮件，支持 Markdown、HTML、附件与代码块语法高亮。触发词：阿里云邮箱、邮件发送(email sending)、Markdown 邮件(Markdown email)、附件(attachment)、SMTP。
tags:
- 阿里云
---

# 阿里云邮件技能

此技能支持通过阿里云企业邮箱服务发送邮件，高级功能包括 Markdown 转换、HTML 样式、文件附件以及代码块的语法高亮。

## 功能
- **阿里云企业邮箱支持**：针对阿里云的 SMTP 服务 (smtp.mxhichina.com) 优化
- **多种内容类型**：发送纯文本、Markdown 或 HTML 邮件
- **带语法高亮的 Markdown**：Markdown 中代码块自动语法高亮
- **文件附件**：包含一个或多个文件作为附件
- **基于配置**：使用安全的配置文件存储 SMTP 凭据
- **错误处理**：包含重试逻辑和详细的错误报告

## 前提条件
- **SMTP 配置文件**：在 OpenClaw 配置目录 (`/root/.openclaw/`) 中创建 `aliyun-mail-config.json`

示例配置文件：
```json
{
  "server": "smtp.mxhichina.com",
  "port": 465,
  "username": "your-email@yourdomain.com",
  "password": "your-app-password",
  "emailFrom": "your-email@yourdomain.com",
  "useTLS": true
}
```

确保配置文件具有安全的权限：
```bash
chmod 600 /root/.openclaw/aliyun-mail-config.json
```

## 使用方法

### 基本文本邮件
```bash
aliyun-mail send --to "recipient@example.com" --subject "Hello" --body "This is a plain text email"
```

### 带语法高亮的 Markdown 邮件
```bash
aliyun-mail send \
  --to "recipient@example.com" \
  --subject "Code Report" \
  --body "**Check out this Python code:**\n\n```python\nprint('Hello World')\n```" \
  --markdown
```

### 带附件的 HTML 邮件
```bash
aliyun-mail send \
  --to "recipient@example.com" \
  --subject "Weekly Report" \
  --body "<h1>Weekly Report</h1><p>See attached file.</p>" \
  --html \
  --attachments "/path/to/report.pdf"
```

### 从文件使用正文
```bash
aliyun-mail send \
  --to "recipient@example.com" \
  --subject "Report from File" \
  --body-file "/path/to/report.md" \
  --markdown \
  --attachments "/path/to/data.csv"
```

## 命令行选项
- `--to`: 收件人邮箱地址 (必需)
- `--subject`: 邮件主题 (必需)
- `--body`: 邮件正文内容 (如果未提供 --body-file 则必需)
- `--body-file`: 包含邮件正文的文件路径
- `--html`: 作为 HTML 邮件发送 (默认: 纯文本)
- `--markdown`: 作为带语法高亮的 Markdown 邮件发送
- `--attachments`: 要附加的文件路径列表，以空格分隔

## 错误处理
该工具包含健壮的错误处理，失败时最多进行 3 次重试。网络问题、认证错误和无效邮箱地址都会报告详细的错误信息。

## 安全注意事项
- 始终使用应用专用密码而不是你的主邮箱密码
- 使用适当的文件权限保持配置文件安全
- 切勿将配置文件提交到版本控制

## 未来增强
- 支持 CC/BCC 收件人
- 邮件模板系统
- 定时邮件发送
- 富文本编辑器集成
