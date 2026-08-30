---
name: agent-browser-clawdbot
description: AI Agent 专用无头浏览器自动化工具，支持无障碍树快照和基于引用的元素选择
---

# Agent Browser Skill

使用带有 refs 的 accessibility tree snapshots 进行快速的浏览器自动化。

## 为什么使用这个而不是内置的 Browser Tool

**在以下情况使用 agent-browser:**
- 自动化多步骤工作流
- 需要确定性的元素选择
- 性能至关重要
- 处理复杂的 SPA
- 需要会话隔离

**在以下情况使用内置的 browser tool:**
- 需要截图/PDF 进行分析
- 需要视觉检查
- 需要浏览器扩展集成

## Core Workflow

```bash
# 1. 导航并截图
agent-browser open https://example.com
agent-browser snapshot -i --json

# 2. 从 JSON 中解析 refs，然后交互
agent-browser click @e2
agent-browser fill @e3 "text"

# 3. 页面变化后重新截图
agent-browser snapshot -i --json
```

## Key Commands

### Navigation
```bash
agent-browser open <url>
agent-browser back | forward | reload | close
```

### Snapshot (始终使用 -i --json)
```bash
agent-browser snapshot -i --json          # 交互元素，JSON 输出
agent-browser snapshot -i -c -d 5 --json  # + 紧凑，深度限制
agent-browser snapshot -s "#main" -i      # 限定到选择器
```

### Interactions (基于 Ref)
```bash
agent-browser click @e2
agent-browser fill @e3 "text"
agent-browser type @e3 "text"
agent-browser hover @e4
agent-browser check @e5 | uncheck @e5
agent-browser select @e6 "value"
agent-browser press "Enter"
agent-browser scroll down 500
agent-browser drag @e7 @e8
```

### Get Information
```bash
agent-browser get text @e1 --json
agent-browser get html @e2 --json
agent-browser get value @e3 --json
agent-browser get attr @e4 "href" --json
agent-browser get title --json
agent-browser get url --json
agent-browser get count ".item" --json
```

### Check State
```bash
agent-browser is visible @e2 --json
agent-browser is enabled @e3 --json
agent-browser is checked @e4 --json
```

### Wait
```bash
agent-browser wait @e2                    # 等待元素
agent-browser wait 1000                   # 等待毫秒
agent-browser wait --text "Welcome"       # 等待文本
agent-browser wait --url "**/dashboard"   # 等待 URL
agent-browser wait --load networkidle     # 等待网络
agent-browser wait --fn "window.ready === true"
```

### Sessions (隔离的浏览器)
```bash
agent-browser --session admin open site.com
agent-browser --session user open site.com
agent-browser session list
# 或通过环境变量: AGENT_BROWSER_SESSION=admin agent-browser ...
```

### State Persistence
```bash
agent-browser state save auth.json        # 保存 cookies/storage
agent-browser state load auth.json        # 加载（跳过登录）
```

### Screenshots & PDFs
```bash
agent-browser screenshot page.png
agent-browser screenshot --full page.png
agent-browser pdf page.pdf
```

### Network Control
```bash
agent-browser network route "**/ads/*" --abort           # 拦截
agent-browser network route "**/api/*" --body '{"x":1}'  # Mock
agent-browser network requests --filter api              # 查看
```

### Cookies & Storage
```bash
agent-browser cookies                     # 获取全部
agent-browser cookies set name value
agent-browser storage local key           # 获取 localStorage
agent-browser storage local set key val
```

### Tabs & Frames
```bash
agent-browser tab new https://example.com
agent-browser tab 2                       # 切换到标签页
agent-browser frame @e5                   # 切换到 iframe
agent-browser frame main                  # 返回主页面
```

## Snapshot Output Format

```json
{
  "success": true,
  "data": {
    "snapshot": "...",
    "refs": {
      "e1": {"role": "heading", "name": "Example Domain"},
      "e2": {"role": "button", "name": "Submit"},
      "e3": {"role": "textbox", "name": "Email"}
    }
  }
}
```

## Best Practices

1. **始终使用 `-i` flag** - 聚焦交互元素
2. **始终使用 `--json`** - 更易于解析
3. **等待稳定性** - `agent-browser wait --load networkidle`
4. **保存认证状态** - 使用 `state save/load` 跳过登录流程
5. **使用 sessions** - 隔离不同的浏览器上下文
6. **使用 `--headed` 进行调试** - 查看正在发生什么

## Example: Search and Extract

```bash
agent-browser open https://www.google.com
agent-browser snapshot -i --json
# AI 识别搜索框 @e1
agent-browser fill @e1 "AI agents"
agent-browser press Enter
agent-browser wait --load networkidle
agent-browser snapshot -i --json
# AI 识别结果 refs
agent-browser get text @e3 --json
agent-browser get attr @e4 "href" --json
```

## Example: Multi-Session Testing

```bash
# Admin session
agent-browser --session admin open app.com
agent-browser --session admin state load admin-auth.json
agent-browser --session admin snapshot -i --json

# User session (同时)
agent-browser --session user open app.com
agent-browser --session user state load user-auth.json
agent-browser --session user snapshot -i --json
```

## Installation

```bash
npm install -g agent-browser
agent-browser install                     # 下载 Chromium
agent-browser install --with-deps         # Linux: + 系统依赖
```

## Credits

Skill created by Yossi Elkrief ([@MaTriXy](https://github.com/MaTriXy))

agent-browser CLI by [Vercel Labs](https://github.com/vercel-labs/agent-browser)
