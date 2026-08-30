---
name: llmrouter
description: 智能LLM代理，根据请求复杂度路由到合适的模型。通过为简单任务使用更便宜的模型来节省成本。已在Anthropic、OpenAI、Gemini、Kimi/Moonshot和Ollama上测试验证。
homepage: https://github.com/alexrudloff/llmrouter
metadata:
  openclaw:
    emoji: 🔀
    homepage: https://github.com/alexrudloff/llmrouter
    os:
    - darwin
    - linux
    requires:
      bins:
      - python3
      anyBins:
      - pip
      - pip3
    primaryEnv: ANTHROPIC_API_KEY
---

# LLM Router

An intelligent proxy that classifies incoming requests by complexity and routes them 迁移到 appropriate LLM models. Use cheaper/faster models for simple tasks and reserve expensive models for complex ones.

**Works with [OpenClaw](https://github.com/openclaw/openclaw)** 迁移到 reduce token 用法 and API costs by routing simple requests 迁移到 smaller models.

**Status:** Tested with Anthropic, OpenAI, Google Gemini, Kimi/Moonshot, and Ollama.

## 快速开始

### 先决条件

1. **Python 3.10+** with pip
2. **Ollama** (optional - only if using local classification)
3. **Anthropic API key** or Claude Code OAuth token (or other provider key)

### 设置

```bash
# Clone if not already present
git clone https://github.com/alexrudloff/llmrouter.git
cd llmrouter

# Create virtual environment (required on modern Python)
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Pull classifier model (if using local classification)
ollama pull qwen2.5:3b

# Copy and customize config
cp config.yaml.example config.yaml
# Edit config.yaml with your API key and model preferences
```

### 验证 安装

```bash
# Start the server
source venv/bin/activate
python server.py

# In another terminal, test health endpoint
curl http://localhost:4001/health
# Should return: {"status": "ok", ...}
```

### 启动 the Server

```bash
python server.py
```

选项:
- `--port PORT` - Port 迁移到 监听 on (default: 4001)
- `--host HOST` - Host 迁移到 bind (default: 127.0.0.1)
- `--config PATH` - Config file path (default: config.yaml)
- `--log` - 启用 verbose logging
- `--openclaw` - 启用 OpenClaw compatibility (rewrites model name in system prompt)

## 配置

编辑 `config.yaml` 迁移到 自定义:

### Model Routing

```yaml
# Anthropic routing
models:
  super_easy: "anthropic:claude-haiku-4-5-20251001"
  easy: "anthropic:claude-haiku-4-5-20251001"
  medium: "anthropic:claude-sonnet-4-20250514"
  hard: "anthropic:claude-opus-4-20250514"
  super_hard: "anthropic:claude-opus-4-20250514"

# OpenAI routing
models:
  super_easy: "openai:gpt-4o-mini"
  easy: "openai:gpt-4o-mini"
  medium: "openai:gpt-4o"
  hard: "openai:o3-mini"
  super_hard: "openai:o3"

# Google Gemini routing
models:
  super_easy: "google:gemini-2.0-flash"
  easy: "google:gemini-2.0-flash"
  medium: "google:gemini-2.0-flash"
  hard: "google:gemini-2.0-flash"
  super_hard: "google:gemini-2.0-flash"
```

**注意:** Reasoning models are auto-detected and use correct API params.

### Classifier

Three 选项 for classifying request complexity:

**Local (default)** - Free, requires Ollama:
```yaml
classifier:
  provider: "local"
  model: "qwen2.5:3b"
```

**Anthropic** - Uses Haiku, fast and cheap:
```yaml
classifier:
  provider: "anthropic"
  model: "claude-haiku-4-5-20251001"
```

**OpenAI** - Uses GPT-4o-mini:
```yaml
classifier:
  provider: "openai"
  model: "gpt-4o-mini"
```

**Google** - Uses Gemini:
```yaml
classifier:
  provider: "google"
  model: "gemini-2.0-flash"
```

**Kimi** - Uses Moonshot:
```yaml
classifier:
  provider: "kimi"
  model: "moonshot-v1-8k"
```

Use remote (anthropic/openai/google/kimi) if your machine 可以't 运行 local models.

### Supported Providers

- `anthropic:claude-*` - Anthropic Claude models (tested)
- `openai:gpt-*`, `ope`ope`ai:o1-`penai``openai:o3-*`AI models (tested)
- `google:gemini-*` - Google Gemini models (tested)
- `kimi:kimi-k2.5`, `kimi:`kimi:`oonshot-*`i/Moonshot models (tested)
- `local:model-name` - Local Ollama models (tested)

## Complexity Levels

| Level | Use Case | Default Model |
|-------|----------|---------------|
| super_easy | Greetings, acknowledgments | Haiku |
| easy | Simple Q&A, reminders | Haiku |
| medium | Coding, emails, research | Sonnet |
| hard | Complex reasoning, debugging | Opus |
| super_hard | System architecture, proofs | Opus |

## Customizing Classification

编辑 `ROUTES.md` 迁移到 tune how messages are classified. The classifier reads the table in this file 迁移到 determine complexity levels.

## API 用法

The router exposes an OpenAI-compatible API:

```bash
curl http://localhost:4001/v1/chat/completions \
  -H "Authorization: Bearer $ANTHROPIC_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llm-router",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## Testing Classification

```bash
python classifier.py "Write a Python sort function"
# Output: medium

python classifier.py --test
# Runs test suite
```

## Running as macOS Service

创建 `~/Library/LaunchAgents/com.llmrouter.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.llmrouter</string>
    <key>ProgramArguments</key>
    <array>
        <string>/path/to/llmrouter/venv/bin/python</string>
        <string>/path/to/llmrouter/server.py</string>
        <string>--openclaw</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>WorkingDirectory</key>
    <string>/path/to/llmrouter</string>
    <key>StandardOutPath</key>
    <string>/path/to/llmrouter/logs/stdout.log</string>
    <key>StandardErrorPath</key>
    <string>/path/to/llmrouter/logs/stderr.log</string>
</dict>
</plist>
```

**重要:** 替换 `/path/to/llmrouter` with your actual 安装 path. 必须 use the venv python, not system python.

```bash
# Create logs directory
mkdir -p ~/path/to/llmrouter/logs

# Load the service
launchctl load ~/Library/LaunchAgents/com.llmrouter.plist

# Verify it's running
curl http://localhost:4001/health

# To stop/restart
launchctl unload ~/Library/LaunchAgents/com.llmrouter.plist
launchctl load ~/Library/LaunchAgents/com.llmrouter.plist
```

## OpenClaw 配置

添加 the router as a provider in `~/.openclaw/openclaw.json`:

```json
{
  "models": {
    "providers": {
      "localrouter": {
        "baseUrl": "http://localhost:4001/v1",
        "apiKey": "via-router",
        "api": "openai-completions",
        "models": [
          {
            "id": "llm-router",
            "name": "LLM Router (Auto-routes by complexity)",
            "reasoning": false,
            "input": ["text", "image"],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      }
    }
  }
}
```

**注意:** Cost is set 迁移到 0 because actual costs depend on which model the router selects. The router logs which model handled each request.

### Set as Default Model (Optional)

迁移到 use the router for all agents by default, 添加:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "localrouter/llm-router"
      }
    }
  }
}
```

### Using with OAuth Tokens

If your `config.yaml` uses an Anthropic OAuth token from OpenClaw's `~/`~/`openclaw/auth-profiles.json`he router automatically handles Claude Code identity headers.

### OpenClaw Compatibility Mode (Required)

**If using with OpenClaw, you 必须 启动 the server with `--openclaw`:**

```bash
python server.py --openclaw
```

This flag enables compatibility 功能特性 required for OpenClaw:
- Rewrites model names in responses so OpenClaw shows the actual model being used
- Handles tool name and ID remapping for proper tool call routing

Without this flag, you 可以 encounter errors when using the router with OpenClaw.

## Common Tasks

- **检查 server status**: `curl http://localhost:4001/health`
- **查看 current config**: `cat config.yaml`
- **测试 a classification**: `python classifier.py "your message"`
- **运行 classification tests**: `python classifier.py --test`
- **Restart server**: 停止 and 运行 `python server.py` again
- **查看 logs** (if running as service): `tail -f logs/stdout.log`

## 故障排除

### "externally-managed-environment" 错误
Python 3.11+ requires virtual environments. 创建 one:
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### "Connection refused" on port 4001
Server isn't running. 启动 it:
```bash
source venv/bin/activate && python server.py
```

### Classification 返回 wrong complexity
编辑 `ROUTES.md` 迁移到 tune classification rules. The classifier reads this file 迁移到 determine complexity levels.

### Ollama errors / "model not found"
Ensure Ollama is running and the model is pulled:
```bash
ollama serve  # Start Ollama if not running
ollama pull qwen2.5:3b
```

### OAuth token not working
Ensure your token in `config.yaml` starts with `sk`sk`ant-oat`he router auto-detects OAuth tokens and adds required identity headers.

### LaunchAgent not starting
检查 logs and ensure paths are absolute:
```bash
cat ~/Library/LaunchAgents/com.llmrouter.plist  # Verify paths
cat /path/to/llmrouter/logs/stderr.log  # Check for errors
```