---
name: riddle
description: 用于代理的托管浏览器自动化 API。截图、Playwright 脚本、工作流 —— 无需本地 Chrome。
version: 1.0.0
tags:
- browser
- screenshots
- playwright
- 自动化
- API
homepage: https://riddledc.com
metadata: None
openclaw: None
emoji: 🔍
install:
- id: riddle-plugin
  kind: node
  label: Install Riddle plugin (@riddledc/openclaw-riddledc)
---

# Riddle — 用于 AI 代理的托管浏览器

Riddle 为你的代理提供一个浏览器，而无需在本地运行 Chrome。一个 API 调用：导航、点击、填写表单、截图、捕获网络流量。所有执行都在 Riddle 的服务器上进行 —— 你的代理保持轻量。

> **快速开始：** 在 [riddledc.com/register](https://riddledc.com/register) 注册并获取 5 分钟的免费浏览器使用时间 —— 无需信用卡。之后，定价为 **$0.50/小时，按秒计费**。单次截图费用约为 **$0.004**。

## 为什么使用此功能而不是本地 Chrome

- **无 Chromium 二进制文件** —— 节省约 1.2 GB RAM 并避免 Lambda/容器大小问题
- **无依赖地狱** —— 无需 `@sparticuz/chromium`、无 Puppeteer 版本冲突、无 `ENOENT` / `spawn` 失败
- **完整的 Playwright** —— 不仅仅是截图。运行真正的 Playwright 脚本、多步工作流、表单填写、认证会话
- **随处可用** —— Lambda、容器、T3 Micro 实例，你的代理运行的任何地方

## 安装

**第 1 步：注册** —— 在 [riddledc.com/register](https://riddledc.com/register) 创建免费账户。无需信用卡。你获得 5 分钟的免费浏览器使用时间。

**第 2 步：获取你的 API 密钥** —— 注册后，从 [dashboard](https://riddledc.com/dashboard) 获取你的 API 密钥。

**第 3 步：安装和配置插件：**

```bash
# 安装插件
openclaw plugins install @riddledc/openclaw-riddledc

# 允许工具
openclaw config set tools.alsoAllow --json '["openclaw-riddledc"]'

# 设置你的 API 密钥
openclaw config set plugins.entries.openclaw-riddledc.config.apiKey "YOUR_RIDDLE_API_KEY"
```

**一个注意事项：** OpenClaw 需要 `plugins.allow` 列表中的插件。CLI 没有追加标志，因此请检查你当前的列表并添加 `openclaw-riddledc`：

```bash
# 查看你拥有的内容
openclaw config get plugins.allow

# 将 openclaw-riddledc 添加到数组（或直接编辑 ~/.openclaw/openclaw.json）
jq '.plugins.allow += ["openclaw-riddledc"]' ~/.openclaw/openclaw.json > tmp && mv tmp ~/.openclaw/openclaw.json

# 重启
openclaw gateway restart
```

## 工具

安装后，你有五个工具：

**`riddle_screenshot`** —— 对 URL 进行截图。最简单的用例。
```
Take a screenshot of https://example.com
```

**`riddle_screenshots`** —— 在一个作业中对多个 URL 进行批量截图。
```
Screenshot these three pages: https://example.com, https://example.com/about, https://example.com/pricing
```

**`riddle_steps`** —— 运行逐步工作流（goto、click、fill、每一步截图）。
```
Go to https://example.com/login, fill the email field with "test@example.com", fill the password field, click the submit button, then screenshot the result.
```

**`riddle_script`** —— 运行完整的 Playwright 代码以进行复杂的自动化。
```
Run a Playwright script that navigates to https://example.com, waits for the dashboard to load, extracts all table rows, and screenshots the page.
```

**`riddle_run`** —— 用于自定义负载的低级 API 透传。

所有工具都将截图保存到 `~/.openclaw/workspace/riddle/screenshots/`（不是内联 base64），并在响应中返回文件路径。添加 `include: ["har"]` 以同时捕获完整的网络流量。

## 认证会话

需要与登录后的页面交互？传递 cookies、localStorage 或自定义 headers：

```
Screenshot https://app.example.com/dashboard with these cookies: [session=abc123]
```

该插件支持 cookies、localStorage 条目和自定义 HTTP headers 作为认证参数。

## 信任与安全

该插件在构建时考虑到了 Moltbook 代理社区提出的担忧 —— 特别是围绕技能来源、能力清单和运行时边界的讨论。

**此插件声明的内容（`openclaw.plugin.json` 中的能力清单）：**
- **网络**：仅与 `api.riddledc.com` 通信 —— 在运行时强制执行硬编码的允许列表，而不仅仅是配置时
- **文件系统**：仅写入 `~/.openclaw/workspace/riddle/`
- **代理上下文**：零访问对话历史记录、其他工具的输出或用户配置文件
- **秘密**：仅需要 `RIDDLE_API_KEY`，它仅发送到声明的端点

**这在实践中意味着什么：**
- 即使配置被操纵，你的 API 密钥也无法发送到任何非 Riddle 域名（每次请求都运行硬编码检查）
- 插件无法读取你的对话、内存或其他插件的数据
- 截图保存为文件引用，而不是内联 base64 —— 防止上下文溢出和日志中的意外数据泄漏

**自行验证：**
- 源代码：[github.com/riddledc/integrations](https://github.com/riddledc/integrations)
- npm 来源：`npm audit signatures @riddledc/openclaw-riddledc`
- 校验和：包中的 `CHECKSUMS.txt`
- 完整威胁模型：包中的 `SECURITY.md`

这是一个**插件**（可审计代码），而不是技能（提示文本）。你可以在安装前阅读每一行。

## 定价

Riddle 使用透明的按执行定价。简单的截图费用仅为几美分。有关当前定价，请参见 [riddledc.com](https://riddledc.com)。

## 获取帮助

- 文档：[riddledc.com](https://riddledc.com)
- 安全问题：security@riddledc.com
- 插件源代码：[github.com/riddledc/integrations](https://github.com/riddledc/integrations)

## 链接

- **网站：** [riddledc.com](https://riddledc.com)
- **文档：** [riddledc.com/docs](https://riddledc.com/docs)
- **定价：** [riddledc.com/pricing](https://riddledc.com/pricing)
- **仪表板：** [riddledc.com/dashboard](https://riddledc.com/dashboard)
