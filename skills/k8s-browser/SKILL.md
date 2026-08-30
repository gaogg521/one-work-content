---
name: k8s-browser
description: 用于 Kubernetes dashboards 和 web UIs 的 Browser automation。在以下情况下使用：与 Kubernetes Dashboard, Grafana, ArgoCD UI, 或其他 web 接口交互。需要 MCP_BROWSER_ENABLED=true。
license: Apache-2.0
metadata:
  author: rohitg00
  version: 1.0.0
  tools: 26
  category: automation
---

# Browser Automation for Kubernetes

使用 kubectl-mcp-server 的 browser 工具（26 个工具）自动化 Kubernetes web UIs。

## 何时应用

在以下情况下使用此技能：
- 用户提到："dashboard", "Grafana", "ArgoCD UI", "web interface", "screenshot"
- 操作：导航 K8s dashboards, 捕获 screenshots, 自动化 web UIs
- 关键词："browser", "click", "screenshot", "web", "UI automation"

## 优先级规则

| Priority | 规则 | Impact | Tools |
|----------|------|--------|-------|
| 1 | 首先启用 MCP_BROWSER_ENABLED | Critical | 环境变量 |
| 2 | 在交互之前打开 URL | High | `browser_open` |
| 3 | 在点击之前等待元素 | High | `browser_wait_for_selector` |
| 4 | 为验证截取 screenshots | Medium | `browser_screenshot` |

## 快速参考

| Task | Tool | Example |
|------|------|---------|
| 打开 URL | `browser_open` | `browser_open(url)` |
| 点击元素 | `browser_click` | `browser_click(selector)` |
| 截取 screenshot | `browser_screenshot` | `browser_screenshot(path)` |
| 等待元素 | `browser_wait_for_selector` | `browser_wait_for_selector(selector)` |

## 先决条件

- **Browser Tools 启用**: 必需
  ```bash
  export MCP_BROWSER_ENABLED=true
  ```

## Enable Browser Tools

```bash
export MCP_BROWSER_ENABLED=true

# 可选: Cloud provider
export MCP_BROWSER_PROVIDER=browserbase  # or browseruse
export BROWSERBASE_API_KEY=bb_...
```

## Basic Navigation

```python
# 打开 URL
browser_open(url="http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/")

# 打开带 auth headers
browser_open_with_headers(
    url="https://grafana.example.com",
    headers={"Authorization": "Bearer token123"}
)

# 导航
browser_navigate(url="https://argocd.example.com/applications")

# 后退/前进
browser_back()
browser_forward()

# 刷新
browser_refresh()
```

## Screenshots and Content

```python
# 截取 screenshot
browser_screenshot(path="dashboard.png")

# 完整页面 screenshot
browser_screenshot(path="full-page.png", full_page=True)

# 获取 page content
browser_content()

# 获取 page title
browser_title()

# 获取 current URL
browser_url()
```

## Interactions

```python
# 点击元素
browser_click(selector="button.submit")
browser_click(selector="text=Deploy")
browser_click(selector="#sync-button")

# 输入文本
browser_type(selector="input[name=search]", text="my-Deployment")
browser_type(selector=".search-box", text="nginx")

# 填充表单
browser_fill(selector="#namespace", text="production")

# 选择下拉框
browser_select(selector="select#cluster", value="prod-cluster")

# 按下按键
browser_press(key="Enter")
browser_press(key="Escape")
```

## Waiting

```python
# 等待元素
browser_wait_for_selector(selector=".loading", state="hidden")
browser_wait_for_selector(selector=".data-table", state="visible")

# 等待导航
browser_wait_for_navigation()

# 等待 network idle
browser_wait_for_load_state(state="networkidle")
```

## Session Management

```python
# 列表 sessions
browser_session_list()

# 切换 session
browser_session_switch(session_id="my-session")

# 关闭 browser
browser_close()
```

## Viewport and Device

```python
# 设置 viewport size
browser_set_viewport(width=1920, height=1080)

# 模拟设备
browser_set_viewport(device="iPhone 12")
```

## Kubernetes Dashboard Workflow

```python
# 1. 启动 kubectl proxy
# kubectl proxy &

# 2. 打开 dashboard
browser_open(url="http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/")

# 3. 导航到 workloads
browser_click(selector="text=Workloads")

# 4. 截取 screenshot
browser_screenshot(path="workloads.png")

# 5. 搜索 Deployment
browser_type(selector="input[placeholder*=search]", text="nginx")
browser_press(key="Enter")
```

## Grafana Dashboard Workflow

```python
# 1. 打开 Grafana
browser_open_with_headers(
    url="https://grafana.example.com/d/k8s-cluster",
    headers={"Authorization": "Bearer admin-token"}
)

# 2. 设置 time range
browser_click(selector="button[aria-label='Time picker']")
browser_click(selector="text=Last 1 hour")

# 3. Screenshot dashboard
browser_screenshot(path="grafana-cluster.png", full_page=True)
```

## ArgoCD UI Workflow

```python
# 1. 打开 ArgoCD
browser_open(url="https://argocd.example.com")

# 2. 登录
browser_fill(selector="input[name=username]", text="admin")
browser_fill(selector="input[name=password]", text="password")
browser_click(selector="button[type=submit]")

# 3. 导航到 app
browser_wait_for_selector(selector=".applications-list")
browser_click(selector="text=my-app")

# 4. 同步 app
browser_click(selector="button.sync-button")
browser_click(selector="text=Synchronize")
```

## 相关 Skills

- [k8s-gitops](../k8s-gitops/SKILL.md) - ArgoCD CLI tools
- [k8s-diagnostics](../k8s-diagnostics/SKILL.md) - 集群分析
