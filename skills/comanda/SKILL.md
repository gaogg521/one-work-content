---
name: comanda
version: 1.0.1
description: 基于comanda CLI生成、可视化和执行声明式AI流水线。适用于从自然语言创建LLM工作流、查看工作流图表、编辑YAML工作流文件或处理运行comanda工作流。支持多模型编排，包括OpenAI、Anthropic、Google、Ollama、Claude Code、Gemini CLI和Codex。
homepage: https://comanda.sh
repository: https://github.com/kris-hansen/comanda
---

# Comanda - Declarative AI Pipelines

🌐 **Website:** [comanda.sh](https://comanda.sh) | 📦 **GitHub:** [kris-hansen/comanda](https://gihttps://gihttps://gihttps://github.com/kris-hansen/comanda)

Comanda defines LLM workflows in YAML and runs them from the 命令 line. Workflows 可以 chain multiple AI models, 运行 steps in 并行, and pipe data through processing stages.

## 安装

```bash
# macOS
brew install kris-hansen/comanda/comanda

# Or via Go
go install github.com/kris-hansen/comanda@latest
```

Then 配置 API keys:
```bash
comanda configure
```

## 命令

### 生成 a Workflow

创建 a workflow YAML from natural language:

```bash
comanda generate <output.yaml> "<prompt>"

# Examples
comanda generate summarize.yaml "Create a workflow that summarizes text input"
comanda generate review.yaml "Analyze code for bugs, then suggest fixes" -m claude-sonnet-4-20250514
```

### Visualize a Workflow

显示 ASCII chart of workflow structure:

```bash
comanda chart <workflow.yaml>
comanda chart workflow.yaml --verbose
```

Shows step relationships, models used, 输入/输出 chains, and validity.

### 处理/执行 a Workflow

运行 a workflow 文件:

```bash
comanda process <workflow.yaml>

# With input
cat file.txt | comanda process analyze.yaml
echo "Design a REST API" | comanda process multi-agent.yaml

# Multiple workflows
comanda process step1.yaml step2.yaml step3.yaml
```

### 查看/编辑 Workflows

Workflow 文件 are YAML. 读取 them directly 迁移到 understand or 修改:

```bash
cat workflow.yaml
```

## Workflow YAML 格式

### Basic Step

```yaml
step_name:
  input: STDIN | NA | filename | $VARIABLE
  model: gpt-4o | claude-sonnet-4-20250514 | gemini-pro | ollama/llama2 | claude-code | gemini-cli
  action: "Instruction for the model"
  output: STDOUT | filename | $VARIABLE
```

### 并行 Execution

```yaml
parallel-process:
  analysis-one:
    input: STDIN
    model: claude-sonnet-4-20250514
    action: "Analyze for security issues"
    output: $SECURITY

  analysis-two:
    input: STDIN
    model: gpt-4o
    action: "Analyze for performance"
    output: $PERF
```

### Chained Steps

```yaml
extract:
  input: document.pdf
  model: gpt-4o
  action: "Extract key points"
  output: $POINTS

summarize:
  input: $POINTS
  model: claude-sonnet-4-20250514
  action: "Create executive summary"
  output: STDOUT
```

### 生成 + 处理 (Meta-workflows)

```yaml
create_workflow:
  input: NA
  generate:
    model: gpt-4o
    action: "Create a workflow that analyzes sentiment"
    output: generated.yaml

run_it:
  input: NA
  process:
    workflow_file: generated.yaml
```

## Available Models

运行 `comanda configure` 迁移到 集合 up API keys. Common models:

| Provider | Models |
|----------|--------|
| OpenAI | `gpt-4o`CODE_1___C__CO`o1-mini`i```mini`-mini`mini`mini` |
| Anthropic | `claude-sonnet-4-20250514`, `claude-opus-4-2`claude-opus-4-2`250514``250514`
| Google | `gemini-pro`, `g__CO`mini-flash`lash`
| Ollama | `ollama/llama2`, `olla`olla``a/mistral` |
| Agentic | `claude-code`, `ge__CODE`ini-``en`pen`en`openai-codex`

## 示例 Location

See `~/clawd/comanda/examples/` for workflow samples:
- `agentic-loop/` - Autonomous agent patterns
- `claude-code/` - Claude Code integration
- `gemini-cli/` - Gemini CLI workflows
- `document-processing/` - PDF, text 提取
- `database-connections/` - DB query workflows

## 故障排除

- **"模型 not configured"**: 运行 `comanda configure` 迁移到 添加 API keys
- **Workflow validation errors**: Use `comanda chart workflow.yaml` 迁移到 visualize and 检查 validity
- **调试 mode**: 添加 `--debug` flag for verbose logging