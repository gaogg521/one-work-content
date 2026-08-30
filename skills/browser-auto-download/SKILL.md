---
name: browser-auto-download
version: 4.1.0
description: 浏览器自动化(Browser Automation)文件下载，支持平台自动检测（Windows/macOS/Linux、64/32 位、ARM/Intel）、多步导航、页面加载自动下载捕获和按钮点击回退。适用于客户端渲染、自动下载或多页导航导致 curl/wget 失效的复杂下载场景。
tags:
- Linux
- Windows
- 自动化
---

# Browser Auto Download v4.1.0 (Enhanced)

使用**智能检测和多步导航**从动态网页下载文件。

## 主要功能

- **自动下载捕获**：检测页面加载时自动触发的下载
- **多步导航**：查找并导航到平台特定页面（PC/桌面版本）
- **平台自动检测**：Windows x64/ARM64、macOS Intel/Apple Silicon、Linux
- **事件监听**：捕获所有下载事件，无需点击按钮
- **智能回退**：尝试多种策略（自动下载、导航、点击）

## 何时使用

使用此技能用于：
- **自动下载网站**：页面加载时自动开始下载
- **多步流程**：首页 - 点击 "PC 版本" - 下载页面
- **动态内容**：通过 JavaScript 生成的下载链接
- **交互式下载**：需要点击按钮或导航 UI

**不适用于**：直接文件 URL（请改用 `curl`/`wget`）

## 快速开始

### 选项 1：自动（推荐）

```bash
python skills/browser-auto-download/scripts/auto_download.py \
  --url "https://example.com/download"
```

脚本将：
1. 检查页面加载时的自动下载
2. 查找平台特定页面链接（PC/桌面版本）
3. 如果需要则导航
4. 尝试点击下载按钮作为回退

### 选项 2：内置快捷方式

```bash
# WeChat DevTools
python skills/browser-auto-download/scripts/auto_download.py --wechat

# Meitu Xiuxiu
python skills/browser-auto-download/scripts/auto_download.py --meitu
```

### 选项 3：Python 模块

```python
from skills.browser-auto-download.scripts.auto_download import auto_download

result = auto_download(
    url="https://example.com/download",
    auto_select=True,   # 平台检测
    auto_navigate=True  # 多步导航
)

if result:
    print(f"Downloaded: {result['path']}")
```

## 工作原理

### 三阶段策略

**阶段 1：自动下载检测**
```
页面加载 - 检查下载 - 成功？
    是:                    否:
    保存文件               进入阶段 2
```

**阶段 2：多步导航**
```
查找 "PC/桌面" 链接 - 导航 - 检查下载 - 成功？
    是:                        否:
    保存文件                  进入阶段 3
```

**阶段 3：按钮点击**
```
尝试多个选择器 - 点击 - 等待下载 - 保存
```

### 平台特定页面检测

自动查找如下链接：
- "meitu for PC" - pc.meitu.com
- "Desktop version" - desktop.example.com
- "Windows Download" - windows.example.com

关键词：`pc`、`desktop`、`windows`、`mac`、`download`、`电脑`、`桌面`、`客户端`

## 示例

### 自动下载网站（最佳情况）

```bash
# 页面加载时触发下载的网站
python skills/browser-auto-download/scripts/auto_download.py \
  --url "https://pc.meitu.com/en/pc?download=pc"
```

### 多步导航

```bash
# 首页 - PC 版本 - 下载
python skills/browser-auto-download/scripts/auto_download.py \
  --url "https://xiuxiu.meitu.com/" \
  --auto-navigate  # 启用（默认：True）
```

### 手动选择器（回退）

```bash
# 如果自动检测失败
python skills/browser-auto-download/scripts/auto_download.py \
  --url "https://example.com/download" \
  --selector "button:has-text('Download for free')"
```

### 禁用功能

```bash
# 不导航到平台页面
python skills/browser-auto-download/scripts/auto_download.py \
  --url "https://example.com" \
  --no-auto-navigate

# 不检测平台
python skills/browser-auto-download/scripts/auto_download.py \
  --url "https://example.com" \
  --no-auto-select
```

## 平台检测

| 系统 | 架构 | 使用的关键词 |
|--------|--------------|---------------|
| Windows | AMD64/x86_64 | windows, win64, x64, 64-bit, pc |
| Windows | x86/i686 | windows, win32, x86, 32-bit, pc |
| macOS | ARM64 (M1/M2/M3) | macos, arm64, apple silicon |
| macOS | x86_64 (Intel) | macos, x64, intel |
| Linux | x86_64 | linux, x64, amd64 |

## 故障排除

**下载未开始**：
- 使用 `--headless`（默认：False）观察过程
- 检查 stderr 中的自动下载消息
- 如果导航导致问题，尝试 `--no-auto-navigate`
- 使用 `--selector` 手动指定按钮

**下载了错误版本**：
- 检查 stderr 输出中的平台检测
- 使用 `--no-auto-select` 并手动指定 `--selector`
- 验证网站是否提供多个版本

**导航到错误页面**：
- 使用 `--no-auto-navigate` 禁用
- 网站可能没有平台特定页面

**文件未保存**：
- 检查输出目录的写入权限
- 确保有足够的磁盘空间
- 等待大文件（最多 3 分钟）

## 输出格式

### stderr（进度）
```
Starting browser (visible)...
Opening: https://example.com
Checking for auto-downloads...
Checking for platform-specific page link...
Found platform page: https://pc.example.com
Navigating to platform page...
Download detected: software_v2.1.0_win64.exe
Saving: software_v2.1.0_win64.exe

SUCCESS!
File: C:\Users\User\Downloads\software_v2.1.0_win64.exe
Size: 231.9 MB
```

### stdout（JSON 结果）
```json
{
  "path": "C:\\Users\\User\\Downloads\\software_v2.1.0_win64.exe",
  "filename": "software_v2.1.0_win64.exe",
  "size_bytes": 243209941,
  "size_mb": 231.9,
  "platform": "Windows AMD64"
}
```

## 真实示例

### Meitu Xiuxiu（多步 + 自动下载）

```python
from auto_download import quick_download_meitu

result = quick_download_meitu()
# 流程：首页 - PC 页面链接 - 导航 - 自动下载
```

### WeChat DevTools（按钮点击）

```python
from auto_download import quick_download_wechat_devtools

result = quick_download_wechat_devtools()
# 流程：首页 - 点击 "Stable Windows 64" - 下载
```

### 通用软件（混合）

```python
result = auto_download(
    url="https://example.com/downloads",
    auto_select=True,    # 检测 Windows 64-bit
    auto_navigate=True   # 查找 "Desktop version" 链接
)
```

## 要求

```bash
pip install playwright
playwright install chromium
```

## 高级用法

### 自定义平台关键词

修改脚本中的 `get_system_preference()` 以添加自定义关键词。

### 与脚本集成

```python
import subprocess
import json

result = subprocess.run([
    'python', 'skills/browser-auto-download/scripts/auto_download.py',
    '--url', 'https://example.com/download'
], capture_output=True, text=True)

if result.returncode == 0:
    data = json.loads(result.stdout)
    print(f"Downloaded: {data['path']}")  # 使用文件
```

### 批量下载

```python
urls = [
    "https://example1.com/download",
    "https://example2.com/download",
    "https://example3.com/download"
]

for url in urls:
    result = auto_download(url)
    if result:
        print(f"Success: {result['filename']}")
```
