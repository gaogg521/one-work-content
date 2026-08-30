---
name: oauth-helper
description: 通过 Telegram 用户确认自动化 OAuth 登录流程，支持 Google、Apple、Microsoft、GitHub、Discord、微信和 QQ 共 7 个提供商。自动检测登录页 OAuth 选项、多选项时询问用户选择、授权前确认并自动处理账户选择和同意页面。
tags:
- 社交媒体
- 即时通讯
---

# OAuth Helper

通过 Telegram 确认自动化 OAuth 登录。支持 7 个主要提供商。

## 支持的提供商

| 提供商 | 状态 | 检测域 |
|----------|--------|------------------|
| Google | ✅ | accounts.google.com |
| Apple | ✅ | appleid.apple.com |
| Microsoft | ✅ | login.microsoftonline.com, login.live.com |
| GitHub | ✅ | github.com/login/oauth |
| Discord | ✅ | discord.com/oauth2 |
| WeChat | ✅ | open.weixin.qq.com |
| QQ | ✅ | graph.qq.com |

## 先决条件

1. Clawd 浏览器登录到 OAuth 提供商（一次性设置）
2. 已配置 Telegram 频道

## 核心工作流

### 流程 A：具有多个 OAuth 选项的登录页面

当用户请求登录网站时：

```
1. 打开网站登录页面
2. 扫描页面以查找可用的 OAuth 按钮
3. 发送 Telegram 消息：
   "🔐 [Site] supports these login methods:
    1️⃣ Google
    2️⃣ Apple  
    3️⃣ GitHub
    Reply with number to choose"
4. 等待用户回复（60秒超时）
5. 点击选定的 OAuth 按钮
6. 进入流程 B
```

### 流程 B：OAuth 授权页面

当在 OAuth 提供商的页面上时：

```
1. 通过 URL 检测 OAuth 页面类型
2. 提取目标站点信息
3. 发送 Telegram："🔐 [Site] requests [Provider] login. Confirm? Reply yes"
4. 等待 "yes"（60秒超时）
5. 执行特定于提供商的点击序列
6. 等待重定向回原始站点
7. 发送："✅ Login successful!"
```

## 检测模式

### Google
```
URL 模式：
- accounts.google.com/o/oauth2
- accounts.google.com/signin/oauth
- accounts.google.com/v3/signin
```

### Apple
```
URL 模式：
- appleid.apple.com/auth/authorize
- appleid.apple.com/auth/oauth2
```

### Microsoft
```
URL 模式：
- login.microsoftonline.com/common/oauth2
- login.microsoftonline.com/consumers
- login.live.com/oauth20
```

### GitHub
```
URL 模式：
- github.com/login/oauth/authorize
- github.com/login
- github.com/sessions/two-factor
```

### Discord
```
URL 模式：
- discord.com/oauth2/authorize
- discord.com/login
- discord.com/api/oauth2
```

### WeChat
```
URL 模式：
- open.weixin.qq.com/connect/qrconnect
- open.weixin.qq.com/connect/oauth2
```

### QQ
```
URL 模式：
- graph.qq.com/oauth2.0/authorize
- ssl.xui.ptlogin2.qq.com
- ui.ptlogin2.qq.com
```

## 按提供商的点击序列

### Google
```
帐户选择器：[data-identifier], .JDAKTe
授权按钮：button:has-text("Allow"), button:has-text("Continue")
```

### Apple
```
电子邮件输入：input[type="email"], #account_name_text_field
密码：input[type="password"], #password_text_field  
继续：button#sign-in, button:has-text("Continue")
信任设备：button:has-text("Trust")
```

### Microsoft
```
帐户选择器：.table-row[data-test-id]
电子邮件输入：input[name="loginfmt"]
密码：input[name="passwd"]
下一步：button#idSIButton9
接受：button#idBtn_Accept
```

### GitHub
```
电子邮件：input#login_field
密码：input#password
登录：input[type="submit"]
授权：button[name="authorize"]
2FA：input#app_totp
```

### Discord
```
电子邮件：input[name="email"]
密码：input[name="password"]
登录：button[type="submit"]
授权：button:has-text("Authorize")
```

### WeChat
```
方法：二维码扫描
- 向用户截图二维码
- 等待移动端扫描确认
- 检测页面重定向
```

### QQ
```
方法：二维码或密码登录
二维码：向用户截图
密码模式：
  - 切换：a:has-text("密码登录")
  - 用户名：input#u
  - 密码：input#p
  - 登录：input#login_button
```

## OAuth 按钮检测

扫描登录页面的这些选择器：

| 提供商 | 选择器 | 常见文本 |
|----------|-----------|-------------|
| Google | `[data-provider="google"]`, `.google-btn` | "Continue with Google" |
| Apple | `[data-provider="apple"]`, `.apple-btn` | "Sign in with Apple" |
| Microsoft | `[data-provider="microsoft"]` | "Sign in with Microsoft" |
| GitHub | `[data-provider="github"]` | "Continue with GitHub" |
| Discord | `[data-provider="discord"]` | "Login with Discord" |
| WeChat | `.wechat-btn`, `img[src*="wechat"]` | "WeChat Login" |
| QQ | `.qq-btn`, `img[src*="qq"]` | "QQ Login" |

## 一次性设置

在 clawd 浏览器中登录每个提供商：

```bash
# Google
browser action=navigate profile=clawd url=https://accounts.google.com

# Apple
browser action=navigate profile=clawd url=https://appleid.apple.com

# Microsoft  
browser action=navigate profile=clawd url=https://login.live.com

# GitHub
browser action=navigate profile=clawd url=https://github.com/login

# Discord
browser action=navigate profile=clawd url=https://discord.com/login

# WeChat/QQ - 使用二维码扫描，无需预登录
```

## 错误处理

- 没有 "yes" 回复 → 取消并通知用户
- 需要 2FA → 提示用户手动输入代码
- 二维码超时 → 重新截图新二维码
- 登录失败 → 截图并发送给用户进行调试

## 使用示例

```
User: Login to Kaggle for me

Agent:
1. Navigate to kaggle.com/account/login
2. Detect Google/Facebook/Yahoo options
3. Send: "🔐 Kaggle supports:
   1️⃣ Google
   2️⃣ Facebook
   3️⃣ Yahoo
   Reply number to choose"
4. User replies: 1
5. Click Google login
6. Detect Google OAuth page
7. Send: "🔐 Kaggle requests Google login. Confirm? Reply yes"
8. User replies: yes
9. Select account, click Continue
10. Send: "✅ Logged into Kaggle!"
```

## 版本历史

- v1.0.0 - 初始版本，支持 7 个 OAuth 提供商
