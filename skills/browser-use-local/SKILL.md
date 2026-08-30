---
name: browser-use-local
description: 在 OpenClaw 容器/主机中使用 browser-use CLI 或 Python 代码进行浏览器自动化，支持打开页面、点击/输入、截图、提取 HTML/链接，并可配合 OpenAI 兼容的 LLM（如 Moonshot/Kimi）与自定义 base_url 运行 Agent。也用于调试 browser-use 会话及从登录页提取二维码。触发词：browser-use、浏览器自动化(browser automation)、截图(screenshot)、二维码(QR code)、Moonshot。
tags:
- AI
- Python
- 自动化
---

# browser-use (本地) 操作手册

## 此环境中的默认约束

- 在此优先使用 **browser-use** (CLI/Python) 而不是 OpenClaw `browser` 工具；如果没有支持的系统浏览器，OpenClaw `browser` 可能会失败。
- 使用 **持久会话** 进行多步骤流程：`--session <name>`。

## 快速 CLI 工作流（非 agent）

1) 打开

```bash
browser-use --session demo open https://example.com
```

2) 检查（有时在重度 JS 站点上 `state` 返回 0 个元素）

```bash
browser-use --session demo --json state | jq '.data | {url,title,elements:(.elements|length)}'
```

3) 屏幕截图（始终有效；最佳调试原语）

```bash
browser-use --session demo screenshot /home/node/.openclaw/workspace/page.png
```

4) 用于链接发现的 HTML（即使 `state` 为空也有效）

```bash
browser-use --session demo --json get html > /tmp/page_html.json
python3 - <<'PY'
import json,re
html=json.load(open('/tmp/page_html.json')).get('data',{}).get('html','')
urls=set(re.findall(r"https?://[^\s\"'<>]+", html))
for u in sorted([u for u in urls if any(k in u for k in ['demo','login','console','qr','qrcode'])])[:200]:
    print(u)
PY
```

5) 通过 JS 进行轻量级 DOM 查询（当 `state` 为空时有用）

```bash
browser-use --session demo --json eval "location.href"
browser-use --session demo --json eval "document.title"
```

## 使用 OpenAI 兼容 LLM 的 Agent 工作流（Moonshot/Kimi）

当 CLI `run` 路径需要 Browser-Use 云密钥，或当你需要严格控制 LLM 参数时，使用 Python 进行 Agent 运行。

### 最小可用 Kimi 示例

创建 `.env`（或导出环境变量）包含：

- `OPENAI_API_KEY=...`
- `OPENAI_BASE_URL=https://api.moonshot.cn/v1`

然后运行捆绑脚本：

```bash
source /home/node/.openclaw/workspace/.venv-browser-use/bin/activate
python /home/node/.openclaw/workspace/skills/browser-use-local/scripts/run_agent_kimi.py
```

**实践中观察到的 Kimi/Moonshot 特性**（修复）：

- `temperature` 对于 `kimi-k2.5` 必须是 `1`。
- `frequency_penalty` 对于 `kimi-k2.5` 必须是 `0`。
- Moonshot 可以拒绝用于结构化输出的严格 JSON Schema。启用：
  - `remove_defaults_from_schema=True`
  - `remove_min_items_from_schema=True`

如果你收到 400 错误提到 `response_format.json_schema ... keyword 'default' is not allowed` 或 `min_items unsupported`，设置这两个标志是首要措施。

## 二维码提取（登录/演示页面）

### 优先顺序

1) **截取页面屏幕截图** 并裁剪候选区域（快速、稳健）。
2) 如果 HTML 包含 `data:image/png;base64,...`，提取并解码它。

### 裁剪候选

使用 `scripts/crop_candidates.py` 从屏幕截图生成多个可能的 QR 裁剪。

```bash
source /home/node/.openclaw/workspace/.venv-browser-use/bin/activate
python skills/browser-use-local/scripts/crop_candidates.py \
  --in /home/node/.openclaw/workspace/login.png \
  --outdir /home/node/.openclaw/workspace/qr_crops
```

### 从 HTML 提取 base64 嵌入图像

```bash
source /home/node/.openclaw/workspace/.venv-browser-use/bin/activate
browser-use --session demo --json get html > /tmp/page_html.json
python skills/browser-use-local/scripts/extract_data_images.py \
  --in /tmp/page_html.json \
  --outdir /home/node/.openclaw/workspace/data_imgs
```

## 故障排除

- **`state` 显示 `elements: 0`**: 使用 `get html` + 正则发现，加上屏幕截图；使用 `eval` 查询 DOM。
- **页面就绪超时警告**: 通常无害；依赖屏幕截图 + HTML。
- **CLI 标志顺序**: 全局标志放在子命令*之前*：
  - ✅ `browser-use --browser chromium --json open https://...`
  - ❌ `browser-use open https://... --browser chromium`
