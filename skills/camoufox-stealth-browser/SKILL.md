---
name: camoufox-stealth-browser
homepage: https://github.com/kesslerio/camoufox-stealth-browser-clawhub-skill
description: 在隔离容器中使用 Camoufox（补丁版 Firefox）进行 C++ 级别反机器人浏览器自动化，可绕过 Cloudflare Turnstile、Datadome、Airbnb、Yelp 等防护。优于仅 JS 级别补丁的 Chrome 方案（undetected-chromedriver、puppeteer-stealth），在标准 Playwright/Selenium 被拦截时使用。触发词：Camoufox、反机器人(anti-bot)、浏览器自动化(browser automation)、Cloudflare、隐身(stealth)。
metadata: None
openclaw: None
emoji: 🦊
requires: None
bins:
- distrobox
env: None
tags:
- 云服务
- 自动化
---

# Camoufox 隐身浏览器 🦊

使用 Camoufox 的 **C++ 级别**反机器人规避 —— 一个带有隐身补丁的自定义 Firefox 分支，编译到浏览器本身，而不是通过 JavaScript 附加。

## 为什么 Camoufox > 基于 Chrome 的解决方案

| 方法 | 检测级别 | 工具 |
|----------|-----------------|-------|
| **Camoufox (此技能)** | C++ 编译补丁 | 不可检测的指纹烘焙到浏览器中 |
| undetected-chromedriver | JS 运行时补丁 | 可以通过时序分析检测 |
| puppeteer-stealth | JS 注入 | 页面加载后应用的补丁 = 可检测 |
| playwright-stealth | JS 注入 | 相同的限制 |

**Camoufox 在源代码级别修补 Firefox** —— WebGL、Canvas、AudioContext 指纹是真正伪造的，而不是由反机器人系统可以检测的 JavaScript 覆盖来掩盖。

## 主要优势

1. **C++ 级别隐身** —— 指纹欺骗编译到浏览器中，不是 JS 技巧
2. **容器隔离** —— 在 distrobox 中运行，保持你的主机系统干净
3. **双工具方法** —— Camoufox 用于浏览器，curl_cffi 用于仅 API（无浏览器开销）
4. **基于 Firefox** —— 比 Chrome 更少被指纹化（每个人都用 Chrome 做机器人）

## 何时使用

- 标准 Playwright/Selenium 被阻止
- 站点显示 Cloudflare 挑战或 "checking your browser"
- 需要抓取 Airbnb、Yelp 或类似的受保护站点
- `puppeteer-stealth` 或 `undetected-chromedriver` 停止工作
- 你需要**实际的**隐身，不是 JS 创可贴

## 工具选择

| 工具 | 级别 | 最适合 |
|------|-------|----------|
| **Camoufox** | C++ 补丁 | 所有受保护站点 - Cloudflare, Datadome, Yelp, Airbnb |
| **curl_cffi** | TLS 欺骗 | 仅 API 端点 - 不需要 JS，非常快 |

## 快速开始

所有脚本在 `pybox` distrobox 中运行以进行隔离。

⚠️ **显式使用 `python3.14`** - pybox 可能有多个 Python 版本，安装了不同的包。

### 1. 设置（首次）

```bash
# 在 pybox 中安装工具（使用 python3.14）
distrobox-enter pybox -- python3.14 -m pip install camoufox curl_cffi

# Camoufox 浏览器在首次运行时自动下载（~700MB Firefox 分支）
```

### 2. 获取受保护页面

**浏览器 (Camoufox):**
```bash
distrobox-enter pybox -- python3.14 scripts/camoufox-fetch.py "https://example.com" --headless
```

**仅 API (curl_cffi):**
```bash
distrobox-enter pybox -- python3.14 scripts/curl-api.py "https://api.example.com/endpoint"
```

## 架构

```
┌─────────────────────────────────────────────────────────┐
│                     OpenClaw Agent                       │
├─────────────────────────────────────────────────────────┤
│  distrobox-enter pybox -- python3.14 scripts/xxx.py         │
├─────────────────────────────────────────────────────────┤
│                      pybox Container                     │
│         ┌─────────────┐  ┌─────────────┐               │
│         │  Camoufox   │  │  curl_cffi  │               │
│         │  (Firefox)  │  │  (TLS spoof)│               │
│         └─────────────┘  └─────────────┘               │
└─────────────────────────────────────────────────────────┘
```

## 工具详情

### Camoufox  
- **什么:** 带有 C++ 级别隐身补丁的自定义 Firefox 构建
- **优点:** 最佳指纹规避，自动通过 Turnstile
- **缺点:** ~700MB 下载，基于 Firefox
- **最适合:** 所有受保护站点 - Cloudflare, Datadome, Yelp, Airbnb

### curl_cffi
- **什么:** 带有浏览器 TLS 指纹欺骗的 Python HTTP 客户端
- **优点:** 无浏览器开销，非常快
- **缺点:** 无 JS 执行，仅 API 端点
- **最适合:** 已知 API 端点，移动应用逆向工程

## 关键：代理要求

**数据中心 IP (AWS, DigitalOcean) = 在 Airbnb/Yelp 上立即被阻止**

你必须使用住宅或移动代理：

```python
# 示例代理配置
proxy = "http://user:pass@residential-proxy.example.com:8080"
```

参见 **[references/proxy-setup.md](references/proxy-setup.md)** 获取代理配置。

## 行为提示

像 Airbnb/Yelp 这样的站点使用行为分析。为避免检测：

1. **预热：** 不要直接访问目标 URL。先访问主页，滚动，四处点击。
2. **鼠标移动：** 注入随机鼠标移动（Camoufox 处理这个）。
3. **时序：** 添加随机延迟（动作之间 2-5s），不是固定间隔。
4. **会话粘性：** 对 10-30 分钟的会话使用相同的代理 IP，不要每个请求都轮换。

## 无头模式警告

⚠️ 旧的 `--headless` 标志被检测。选项：

1. **新无头：** 使用 `headless="new"` (Chrome 109+)
2. **Xvfb:** 在虚拟显示器中运行 headed 浏览器
3. **Headed:** 如果可以，直接运行 headed（最可靠）

```bash
# Xvfb 方法 (Linux)
Xvfb :99 -screen 0 1920x1080x24 &
export DISPLAY=:99
python scripts/camoufox-fetch.py "https://example.com"
```

## 故障排除

| 问题 | 解决方案 |
|---------|----------|
| 立即 "Access Denied" | 使用住宅代理 |
| Cloudflare 挑战循环 | 尝试 Camoufox 而不是 Nodriver |
| pybox 中的浏览器崩溃 | 安装缺失的依赖：`sudo dnf install gtk3 libXt` |
| TLS 指纹被阻止 | 使用 curl_cffi 配合 `impersonate="chrome120"` |
| Turnstile 复选框出现 | 添加鼠标移动，增加等待时间 |
| `ModuleNotFoundError: camoufox` | 使用 `python3.14` 而不是 `python` 或 `python3` |
| `greenlet` segfault (exit 139) | Python 版本不匹配 - 显式使用 `python3.14` |
| `libstdc++.so.6` 错误 | NixOS 库路径问题 - 在 pybox 中使用 `python3.14` |

### Python 版本问题 (NixOS/pybox)

`pybox` 容器可能有多个 Python 版本，带有独立的 site-packages：

```bash
# 检查哪个 Python 有 camoufox
distrobox-enter pybox -- python3.14 -c "import camoufox; print('OK')"

# 错误（可能使用不同的 Python）
distrobox-enter pybox -- python3.14 scripts/camoufox-session.py ...

# 正确（显式版本）
distrobox-enter pybox -- python3.14 scripts/camoufox-session.py ...
```

如果你收到 segfaults 或导入错误，始终显式使用 `python3.14`。

## 示例

### 抓取 Airbnb 列表

```bash
distrobox-enter pybox -- python3.14 scripts/camoufox-fetch.py \
  "https://www.airbnb.com/rooms/12345" \
  --headless --wait 10 \
  --screenshot airbnb.png
```

### 抓取 Yelp 商家

```bash
distrobox-enter pybox -- python3.14 scripts/camoufox-fetch.py \
  "https://www.yelp.com/biz/some-restaurant" \
  --headless --wait 8 \
  --output yelp.html
```

### 使用 TLS 欺骗的 API 抓取

```bash
distrobox-enter pybox -- python3.14 scripts/curl-api.py \
  "https://api.yelp.com/v3/businesses/search?term=coffee&location=SF" \
  --headers '{"Authorization": "Bearer xxx"}'
```

## 会话管理

持久会话允许跨运行重用认证状态，无需重新登录。

### 快速开始

```bash
# 1. 交互式登录（headed 浏览器打开）
distrobox-enter pybox -- python3.14 scripts/camoufox-session.py \
  --profile airbnb --login "https://www.airbnb.com/account-settings"

# 在浏览器中完成登录，然后按 Enter 保存会话

# 2. 在无头模式下重用会话
distrobox-enter pybox -- python3.14 scripts/camoufox-session.py \
  --profile airbnb --headless "https://www.airbnb.com/trips"

# 3. 检查会话状态
distrobox-enter pybox -- python3.14 scripts/camoufox-session.py \
  --profile airbnb --status "https://www.airbnb.com"
```

### 标志

| 标志 | 描述 |
|------|-------------|
| `--profile NAME` | 会话存储的命名配置文件 (必需) |
| `--login` | 交互式登录模式 - 打开 headed 浏览器 |
| `--headless` | 在无头模式下使用保存的会话 |
| `--status` | 检查会话是否看起来有效 |
| `--export-cookies FILE` | 将 cookies 导出到 JSON 进行备份 |
| `--import-cookies FILE` | 从 JSON 文件导入 cookies |

### 存储

- **位置:** `~/.stealth-browser/profiles/<name>/`
- **权限:** 目录 `700`，文件 `600`
- **配置文件名称:** 仅字母、数字、`_`、`-` (1-63 字符)

### Cookie 处理

- **保存:** 所有 cookies 从所有域存储在浏览器配置文件中
- **恢复:** 仅使用匹配目标 URL 域的 cookies
- **SSO:** 如果重定向到 Google/认证域，重新认证一次，配置文件更新

### 登录墙检测

脚本使用多个信号检测会话过期：

1. **HTTP 状态:** 401, 403
2. **URL 模式:** `/login`, `/signin`, `/auth`
3. **标题模式:** "login", "sign in", 等
4. **内容关键词:** "captcha", "verify", "authenticate"
5. **表单检测:** 密码输入字段

如果在 `--headless` 模式下检测到，你会看到：
```
🔒 Login wall signals: url-path, password-form
```

使用 `--login` 重新运行以刷新会话。

### 远程登录 (SSH)

由于 `--login` 需要可见的浏览器，你需要显示转发：

**X11 转发 (首选):**
```bash
# 使用 X11 转发连接
ssh -X user@server

# 运行登录（在你的本地机器上打开浏览器）
distrobox-enter pybox -- python3.14 scripts/camoufox-session.py \
  --profile mysite --login "https://example.com"
```

**VNC 替代方案:**
```bash
# 在服务器上：启动 VNC 会话
vncserver :1

# 在客户端上：连接到 VNC
vncviewer server:1

# 在 VNC 会话中：运行登录
distrobox-enter pybox -- python3.14 scripts/camoufox-session.py \
  --profile mysite --login "https://example.com"
```

### 安全注意事项

⚠️ **Cookies 是凭据。** 像对待密码一样对待配置文件目录：
- 配置文件目录有 `chmod 700`（仅所有者）
- Cookie 导出有 `chmod 600`
- 不要通过不安全的渠道共享配置文件或导出的 cookies
- 考虑加密备份

### 限制

| 限制 | 原因 |
|------------|--------|
| localStorage/sessionStorage 未导出 | 改用浏览器配置文件（自动处理） |
| IndexedDB 不可移植 | 存储在浏览器配置文件中，不在 cookie 导出中 |
| 无并行配置文件访问 | v1 中没有文件锁定；每个配置文件使用一个进程 |

## 参考资料

- [references/proxy-setup.md](references/proxy-setup.md) —— 代理配置指南
- [references/fingerprint-checks.md](references/fingerprint-checks.md) —— 反机器人系统检查什么
