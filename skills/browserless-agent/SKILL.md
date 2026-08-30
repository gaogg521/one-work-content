---
name: browserless-agent
description: 专业网页自动化(Web Automation)技能，基于 headless 浏览器实现导航、数据采集、自动化、测试和交互，支持 30+ 浏览器操作。
homepage: https://github.com/openclaw/browserless-agent
user-invocable: True
metadata: None
openclaw: None
emoji: 🌐
primaryEnv: BROWSERLESS_URL
requires: None
env:
- BROWSERLESS_URL
tags:
- AI
- Web
- 自动化
---

# Browserless Agent 🌐

一个全面的 OpenClaw 网页自动化技能，提供 30+ 浏览器操作，包括导航、数据提取、表单填写、截图捕获、PDF 生成、文件处理和高级网页抓取功能。

## 🚀 功能

- **导航**：完全控制页面导航、重定向和历史
- **数据提取**：获取文本、属性、HTML、计算样式和结构化数据
- **表单自动化**：输入文本、点击按钮、选择选项、上传文件
- **视觉捕获**：截图（整页、仅元素、视口）
- **内容生成**：使用自定义选项将页面保存为 PDF
- **高级交互**：悬停、拖放、键盘快捷键、滚动
- **多标签支持**：管理多个页面和窗口
- **网络控制**：拦截请求、修改 header、阻止资源
- **存储访问**：管理 cookies、localStorage、sessionStorage
- **动态内容**：等待选择器、网络空闲、自定义条件
- **iFrames**：与嵌套框架内容交互
- **浏览器状态**：模拟设备、设置地理位置、处理对话框

## 🔧 配置

此技能需要在 OpenClaw 中配置 `BROWSERLESS_URL` 环境变量。
可选地，您也可以设置 `BROWSERLESS_TOKEN` 进行认证。

**设置方法：**
1. 打开 OpenClaw 设置
2. 导航到 Skills → browserless-agent
3. 在 API Key 字段中输入您的 Browserless 基础 URL
4. （可选）在 env 部分添加 BROWSERLESS_TOKEN 进行令牌认证

**配置示例：**

### 云服务（带令牌）：
```
BROWSERLESS_URL=wss://chrome.browserless.io
BROWSERLESS_TOKEN=your-token-here
```

### 本地服务（无令牌）：
```
BROWSERLESS_URL=ws://localhost:3000
```

### 自定义端点：
```
BROWSERLESS_URL=wss://your-host.com/playwright/chromium
BROWSERLESS_TOKEN=optional-token
```

该技能将自动：
- 如果未指定端点，添加 `/playwright/chromium`
- 如果设置了 `BROWSERLESS_TOKEN`，将令牌附加为查询参数
- 无论是否有认证令牌都能工作

在 [browserless.io](https://browserless.io) 获取您的 Browserless 服务

在 [browserless.io](https://browserless.io) 获取您的 Browserless 服务

## 📖 可用操作

### 导航与页面控制

#### `navigate`
导航到 URL。
```json
{"url": "https://example.com"}
```

#### `go_back`
导航到历史中的上一页。
```json
{}
```

#### `go_forward`
导航到历史中的下一页。
```json
{}
```

#### `reload`
重新加载当前页面。
```json
{"hard": false}
```

#### `wait_for_load`
等待页面完成加载。
```json
{"timeout": 30000}
```

### 数据提取

#### `get_text`
从元素中提取文本内容。
```json
{"selector": "h1", "all": false}
```

#### `get_attribute`
从元素获取属性值。
```json
{"selector": "img", "attribute": "src", "all": false}
```

#### `get_html`
获取元素的内部或外部 HTML。
```json
{"selector": "article", "outer": false, "all": false}
```

#### `get_value`
从表单元素获取输入值。
```json
{"selector": "input[name='email']"}
```

#### `get_style`
获取计算的 CSS 样式属性。
```json
{"selector": ".box", "property": "background-color"}
```

#### `get_multiple`
同时提取多段数据。
```json
{
  "extractions": [
    {"name": "title", "selector": "h1", "type": "text"},
    {"name": "image", "selector": "img", "type": "attribute", "attribute": "src"},
    {"name": "price", "selector": ".price", "type": "text"}
  ]
}
```

### 交互与输入

#### `type_text`
在元素中输入文本。
```json
{"selector": "input[type='search']", "text": "hello world", "delay": 0, "clear": true}
```

#### `click`
点击元素。
```json
{"selector": "button.submit", "force": false, "delay": 0}
```

#### `double_click`
双击元素。
```json
{"selector": ".item"}
```

#### `right_click`
右键点击（上下文菜单）元素。
```json
{"selector": ".context-target"}
```

#### `hover`
将鼠标移到元素上。
```json
{"selector": ".menu-item"}
```

#### `focus`
聚焦元素。
```json
{"selector": "input"}
```

#### `select_option`
在下拉框中选择选项。
```json
{"selector": "select", "values": ["option1", "option2"]}
```

#### `check`
勾选复选框或单选按钮。
```json
{"selector": "input[type='checkbox']"}
```

#### `uncheck`
取消勾选复选框。
```json
{"selector": "input[type='checkbox']"}
```

#### `upload_file`
上传文件到文件输入。
```json
{"selector": "input[type='file']", "files": ["path/to/file.pdf"]}
```

#### `press_key`
按下键盘按键。
```json
{"key": "Enter"}
```
常用按键：Enter、Tab、Escape、ArrowDown、Control+A 等。

#### `keyboard_type`
使用键盘输入文本（支持快捷键）。
```json
{"text": "Hello World"}
```

### 滚动与定位

#### `scroll_to`
滚动到特定位置。
```json
{"x": 0, "y": 500}
```

#### `scroll_into_view`
将元素滚动到视口中。
```json
{"selector": ".footer"}
```

#### `scroll_to_bottom`
滚动到页面底部。
```json
{}
```

#### `scroll_to_top`
滚动到页面顶部。
```json
{}
```

### 视觉与捕获

#### `screenshot`
截取页面或元素的屏幕截图。
```json
{
  "path": "screenshot.png",
  "full_page": true,
  "selector": null,
  "quality": 90,
  "type": "png"
}
```

#### `pdf`
从当前页面生成 PDF。
```json
{
  "path": "page.pdf",
  "format": "A4",
  "landscape": false,
  "margin": {"top": "1cm", "right": "1cm", "bottom": "1cm", "left": "1cm"},
  "print_background": true
}
```

### 评估与执行

#### `evaluate`
在页面上下文中执行 JavaScript。
```json
{"expression": "document.title"}
```

#### `evaluate_function`
执行带参数的 JavaScript 函数。
```json
{
  "function": "(x, y) => x + y",
  "args": [5, 10]
}
```

### 等待与计时

#### `wait_for_selector`
等待元素出现。
```json
{"selector": ".dynamic-content", "timeout": 10000, "state": "visible"}
```
状态：visible、hidden、attached、detached

#### `wait_for_timeout`
等待指定的毫秒数。
```json
{"timeout": 2000}
```

#### `wait_for_function`
等待 JavaScript 表达式返回真值。
```json
{
  "expression": "() => document.readyState === 'complete'",
  "timeout": 10000
}
```

#### `wait_for_navigation`
等待导航完成。
```json
{"timeout": 30000, "wait_until": "networkidle"}
```
wait_until 选项：load、domcontentloaded、networkidle

### 元素状态检查

#### `is_visible`
检查元素是否可见。
```json
{"selector": ".modal"}
```

#### `is_enabled`
检查元素是否启用。
```json
{"selector": "button"}
```

#### `is_checked`
检查复选框/单选按钮是否已勾选。
```json
{"selector": "input[type='checkbox']"}
```

#### `element_exists`
检查元素是否存在于 DOM 中。
```json
{"selector": ".optional-element"}
```

#### `element_count`
计数匹配选择器的元素。
```json
{"selector": ".list-item"}
```

### 存储与 Cookies

#### `get_cookies`
获取所有 cookies 或特定 cookie。
```json
{"name": "session_id"}
```

#### `set_cookie`
设置 cookie。
```json
{
  "name": "user_preference",
  "value": "dark_mode",
  "domain": "example.com",
  "path": "/",
  "expires": 1735689600,
  "httpOnly": false,
  "secure": true,
  "sameSite": "Lax"
}
```

#### `delete_cookies`
删除 cookies。
```json
{"name": "session_id"}
```
省略名称以删除所有 cookies。

#### `get_local_storage`
获取 localStorage 项。
```json
{"key": "user_data"}
```

#### `set_local_storage`
设置 localStorage 项。
```json
{"key": "theme", "value": "dark"}
```

#### `clear_local_storage`
清除所有 localStorage。
```json
{}
```

### 网络与请求

#### `set_extra_headers`
为所有请求设置额外的 HTTP header。
```json
{
  "headers": {
    "Authorization": "Bearer token123",
    "X-Custom-Header": "value"
  }
}
```

#### `block_resources`
阻止特定资源类型。
```json
{"types": ["image", "stylesheet", "font"]}
```
类型：document、stylesheet、image、media、font、script、xhr、fetch、other

#### `get_page_info`
获取全面的页面信息。
```json
{}
```
返回：title、url、html（可选）、viewport size 等。

### iFrame 处理

#### `get_frame_text`
从 iframe 内的元素获取文本。
```json
{
  "frame_selector": "iframe#content",
  "selector": "h1"
}
```

#### `click_in_frame`
点击 iframe 内的元素。
```json
{
  "frame_selector": "iframe#content",
  "selector": "button"
}
```

### 多页面/标签

#### `new_page`
打开新页面/标签。
```json
{"url": "https://example.com"}
```

#### `close_page`
关闭特定页面。
```json
{"index": 1}
```

#### `switch_page`
切换到不同页面。
```json
{"index": 0}
```

#### `list_pages`
列出所有打开的页面。
```json
{}
```

### 浏览器上下文

#### `set_viewport`
设置视口大小。
```json
{"width": 1920, "height": 1080}
```

#### `emulate_device`
模拟移动设备。
```json
{"device": "iPhone 12"}
```
常用设备：iPhone 12、iPad Pro、Galaxy S21、Pixel 5

#### `set_geolocation`
设置地理位置。
```json
{
  "latitude": 37.7749,
  "longitude": -122.4194,
  "accuracy": 100
}
```

#### `set_user_agent`
设置自定义 user agent。
```json
{"user_agent": "Mozilla/5.0..."}
```

### 高级自动化

#### `drag_and_drop`
拖动元素并放到目标上。
```json
{
  "source": ".draggable",
  "target": ".drop-zone"
}
```

#### `fill_form`
同时填写多个表单字段。
```json
{
  "fields": {
    "input[name='email']": "user@example.com",
    "input[name='password']": "secret123",
    "select[name='country']": "US"
  }
}
```

#### `extract_table`
从 HTML 表格提取数据。
```json
{
  "selector": "table.data",
  "headers": true
}
```

#### `extract_links`
从页面提取所有链接。
```json
{
  "selector": "a",
  "filter": "^https://example\\.com"
}
```

#### `handle_dialog`
设置如何处理 JavaScript 对话框（alert/confirm/prompt）。
```json
{
  "action": "accept",
  "text": "Optional prompt response"
}
```
操作：accept、dismiss

## 💡 使用示例

### 示例 1：网页抓取
```bash
python main.py get_multiple '{
  "url": "https://news.ycombinator.com",
  "extractions": [
    {"name": "titles", "selector": ".titleline > a", "type": "text", "all": true},
    {"name": "links", "selector": ".titleline > a", "type": "attribute", "attribute": "href", "all": true}
  ]
}'
```

### 示例 2：表单自动化
```bash
python main.py fill_form '{
  "url": "https://example.com/contact",
  "fields": {
    "input[name='name']": "John Doe",
    "input[name='email']": "john@example.com",
    "textarea[name='message']": "Hello!"
  }
}'
```

### 示例 3：带元素高亮的截图
```bash
python main.py screenshot '{
  "url": "https://example.com",
  "selector": ".hero-section",
  "path": "hero.png",
  "full_page": false
}'
```

### 示例 4：PDF 生成
```bash
python main.py pdf '{
  "url": "https://example.com/report",
  "path": "report.pdf",
  "format": "A4",
  "margin": {"top": "2cm", "bottom": "2cm"}
}'
```

## 🎯 OpenClaw 集成

要从 OpenClaw 使用此技能，代理可以自动调用这些操作。示例：

**用户：** "Take a screenshot of example.com"  
**代理：** 使用 URL 执行 `screenshot` 操作

**用户：** "What's the title of wikipedia.org?"  
**代理：** 导航到 Wikipedia 并从标题元素提取文本

**用户：** "Search for 'Python' on Google and get the first result link"  
**代理：** 导航到 Google，输入搜索，点击搜索，提取第一个结果

## 🔒 安全注意

- Browserless 连接使用基于 TLS 的 WebSocket (wss://)
- 凭证永远不会被记录或在响应中暴露
- 所有浏览器操作都在 Browserless 容器中隔离
- 无需本地浏览器安装

## 🐛 故障排除

**连接失败：**
- 验证 `BROWSERLESS_WS` URL 是否正确
- 检查令牌是否有效且未过期
- 确保网络允许 WebSocket 连接

**超时错误：**
- 为加载缓慢的页面增加超时值
- 在与动态内容交互之前使用 `wait_for_selector`
- 考虑对 AJAX 重的网站使用 `wait_until: "networkidle"`

**元素未找到：**
- 使用浏览器 DevTools 验证选择器
- 使用 `wait_for_selector` 等待元素加载
- 检查元素是否在 iframe 内

## 📚 资源

- [Playwright Documentation](https://playwright.dev)
- [CSS Selectors Reference](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors)
- [Browserless Documentation](https://docs.browserless.io)
