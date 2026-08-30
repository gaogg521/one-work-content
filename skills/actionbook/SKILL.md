---
name: actionbook
description: 浏览器自动化、网页抓取、截图、表单填写、UI 测试、监控与 AI agent 构建。提供经过预验证的页面动作、分步说明与经过测试的 selector。触发词：浏览器自动化(browser automation)、网页抓取(web scraping)、截图(screenshot)、表单填写(form filling)、UI 测试(UI testing)、AI agent。
tags:
- AI
- Web
- 测试
- 自动化
---

## 何时使用此 Skill

当用户的请求涉及与网站交互时激活：

当用户：
- 需要在网站上执行任何操作（"发送 LinkedIn 消息"、"预订 Airbnb"、"在 Google 上搜索..."）
- 询问如何与站点交互（"如何发推文？"、"如何在 LinkedIn 上申请？"）
- 想要在特定站点上填写表单、点击按钮、导航、搜索、筛选或浏览
- 想要截取网页截图或监控更改
- 构建基于浏览器的 AI agent、网页抓取器或外部网站的 E2E 测试
- 自动化重复性 Web 任务（数据录入、表单提交、内容发布）
- 想要控制其现有的 Chrome 浏览器（Extension 模式）


## Actionbook 提供什么

Actionbook 是一个**经过预验证的页面交互数据**库。`actionbook search` 查找与任务描述匹配的动作；`actionbook get "<ID>"` 返回一个结构化文档，描述页面的用途、功能能力和 DOM 结构，并内联 CSS selector —— 无需运行时页面结构发现。

## search 和 get

### search —— 按任务描述查找动作

```bash
actionbook search "<query>"                      # 按任务意图搜索
actionbook search "<query>" --domain site.com    # 按 domain 筛选
actionbook search "<query>" --url <url>          # 按 URL 筛选
actionbook search "<query>" -p 2 -s 20           # 分页
```

**返回**每个结果：
- `ID` —— 与 `actionbook get "<ID>"` 一起使用以检索完整详情
- `Type` —— `page`（完整页面）或 `area`（页面区域）
- `Description` —— 页面概述和功能摘要
- `URL` —— 此动作适用的页面
- `Health Score` —— selector 可靠性百分比（0–100%）
- `Updated` —— 最后验证日期

### 构建有效的搜索查询

`query` 字符串是查找正确动作的**主要信号**。将用户的完整意图打包进去 —— 不只是站点名称或模糊关键词。

**在查询中包含：**
1. **目标站点** —— 网站名称或 domain
2. **任务动词** —— 用户想要做什么（search、book、post、filter、login、compose 等）
3. **对象 / 上下文** —— 他们正在操作什么（listings、messages、flights、repositories 等）
4. **具体细节** —— 用户提到的任何约束、筛选器或参数（dates、location、category、language 等）

**经验法则：** 将用户的请求重写为单个描述性句子，并将其用作查询。

| 用户说 | 差的查询 | 好的查询 |
|-----------|-----------|------------|
| "下周在东京预订 Airbnb" | `"airbnb"` | `"airbnb search listings Tokyo dates check-in check-out guests"` |
| "在 arXiv 上搜索最近的 NLP 论文" | `"arxiv search"` | `"arxiv advanced search papers NLP natural language processing recent"` |
| "发送 LinkedIn 连接请求" | `"linkedin"` | `"linkedin send connection request invite someone"` |
| "发布带图片的推文" | `"twitter post"` | `"twitter compose new tweet post with image media attachment"` |
| "按标签筛选 GitHub issues" | `"github issues"` | `"github repository issues filter by label search issues"` |

**当用户提供额外上下文时**（例如，具体日期、城市名称、主题），即使它不会按字面意思匹配存储的动作，也将其纳入查询 —— 它有助于搜索引擎将相关页面排名更高。

```bash
# 用户："帮我在 LinkedIn 上申请软件工程师工作"
actionbook search "linkedin job search apply software engineer application form"

# 用户："我需要在 arXiv 上搜索机器学习论文"
actionbook search "arxiv advanced search papers machine learning subject category"
```

如果知道 `--domain` 或 `--url`，始终添加它们 —— 它们可以缩小结果范围并提高精确度。

### get —— 按 ID 检索完整动作详情

```bash
# 直接使用搜索结果中的 ID
actionbook get "arxiv.org:/search/advanced:default"
```

**返回**一个结构化文档，包含：

1. **Page URL** —— 确切的 URL 和 query/path 参数
2. **Page Overview** —— 页面的功能
3. **Page Function Summary** —— 交互能力（例如，"Search Term Input"、"Subject Classification Filtering"）
4. **Page Structure Summary** —— 内联 CSS selector 的 DOM 层级

Selector 嵌入在结构描述中，例如：
```
Search Term Form Section: Contains search term input field (input[type="text"]),
field selector dropdown (select[name="searchtype"]), and submit button (button.Search)
```

从结构摘要中提取 CSS selector 以用于浏览器命令。

## 浏览器命令

快速参考。所有标志和选项的完整详情：[command-reference.md](references/command-reference.md)。

### 导航

```bash
actionbook browser open <url>           # 在新标签页打开 URL
actionbook browser goto <url>           # 导航当前页面
actionbook browser back / forward       # 历史导航
actionbook browser reload               # 重新加载页面
actionbook browser pages                # 列出打开的标签页
actionbook browser switch <page_id>     # 切换标签页
actionbook browser close                # 关闭浏览器
```

### 交互

```bash
actionbook browser click "<selector>"          # 点击元素
actionbook browser fill "<selector>" "text"    # 清空并输入
actionbook browser type "<selector>" "text"    # 追加文本
actionbook browser select "<selector>" "value" # 选择下拉选项
actionbook browser hover "<selector>"          # 悬停
actionbook browser press Enter                 # 按键
```

### 观察

```bash
actionbook browser text                        # 完整页面文本
actionbook browser text "<selector>"           # 元素文本
actionbook browser snapshot                    # Accessibility tree（实时页面结构）
actionbook browser screenshot                  # 保存截图
actionbook browser screenshot --full-page      # 完整页面截图
actionbook browser wait "<selector>"           # 等待元素
actionbook browser wait-nav                    # 等待导航
```

`actionbook browser close` 清理浏览器会话。如果用户请求浏览器保持打开，则跳过。

## 示例

用户请求："在 arXiv 上搜索关于 Neural Networks 的论文，仅在标题中搜索"

```bash
# 1. 搜索 —— 包含完整意图：site + task + subject + filter preference
actionbook search "arxiv advanced search papers neural network title field" --domain arxiv.org

# 2. 获取详情 —— 读取 Page Structure Summary 中的 selector
actionbook get "arxiv.org:/search/advanced:default"
# 响应包含：input[type="text"], select[name="searchtype"], button.Search 等

# 3. 使用响应中的 selector 自动化
actionbook browser open "https://arxiv.org/search/advanced"
actionbook browser fill "input[type='text']" "Neural Network"
actionbook browser select "select[name='searchtype']" "title"
actionbook browser click "button.Search"
actionbook browser wait-nav
actionbook browser text
actionbook browser close
```

## 回退

Actionbook 存储在索引时捕获的页面数据。网站会演变，因此 selector 可能会过时。

当 `actionbook get` 中的 selector 在运行时失败时，`actionbook browser snapshot` 提供带有当前 selector 的**实时 accessibility tree**。使用 snapshot 输出中的 selector 重试交互。

浏览器命令中使用的 selector 应来自当前会话中的 `actionbook get` 或 `actionbook browser snapshot` 输出 —— 而不是来自先前知识或记忆。

如果 `actionbook search` 对页面没有返回结果，请使用 `snapshot` 作为主要来源，或回退到其他可用工具。

## 参考

| 参考 | 描述 |
|-----------|-------------|
| [command-reference.md](references/command-reference.md) | 包含所有标志和选项的完整命令参考 |
| [authentication.md](references/authentication.md) | 登录流程、OAuth、2FA 处理、会话持久化 |
