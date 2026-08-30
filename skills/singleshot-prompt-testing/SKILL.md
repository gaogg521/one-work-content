---
name: singleshot-prompt-testing
description: 使用 single shot 进行 prompt 成本测试与优化。支持多提供商(provider)对比、token 效率基准测试及本地免费测试。触发词：prompt 测试、成本优化(cost optimization)、token 效率(token efficiency)、single shot、模型对比(model comparison)
tags:
- 性能优化
- 测试
---

# Singleshot Prompt Testing & Optimization Skill

## Description

使用 single shot 进行 prompt 成本测试

## Installation

```bash
brew tap vincentzhangz/singleshot
brew install singleshot
```

或者: `cargo install singleshot`

## When to Use

- 在 openclaw 实现之前测试新 prompts
- 对 prompt 变体进行 token 效率基准测试
- 比较 model 性能和成本
- 在生产环境之前验证 prompt 输出

## Core Commands

**始终使用 `-d` (detail) 和 `-r` (report) flags 进行效率分析：**

```bash
# 使用完整指标进行基本测试
singleshot chat -p "Your prompt" -P openai -d -r report.md

# 使用配置文件测试
singleshot chat -l config.md -d -r report.md

# 比较 providers
singleshot chat -p "Test" -P openai -m gpt-4o-mini -d -r openai.md
singleshot chat -p "Test" -P anthropic -m claude-sonnet-4-20250514 -d -r anthropic.md

# 批量测试变体
for config in *.md; do
  singleshot chat -l "$config" -d -r "report-${config%.md}.md"
done
```

## Report Analysis Workflow

### 1. Generate Baseline
```bash
singleshot chat -p "Your prompt" -P openai -d -r baseline.md
cat baseline.md
```

### 2. Optimize & Compare
```bash
# 创建优化版本、测试并比较
cat > optimized.md << 'EOF'
---provider---
openai
---model---
gpt-4o-mini
---max_tokens---
200
---system---
Expert. Be concise.
---prompt---
Your optimized prompt
EOF

singleshot chat -l optimized.md -d -r optimized-report.md

# 比较指标
echo "Baseline:" && grep -E "(Tokens|Cost)" baseline.md
echo "Optimized:" && grep -E "(Tokens|Cost)" optimized-report.md
```

## Report Metrics

报告包含：
```markdown
## Token Usage
- Input Tokens: 245
- Output Tokens: 180
- Total Tokens: 425

## Cost (estimated)
- Input Cost: $0.00003675
- Output Cost: $0.000108
- Total Cost: $0.00014475

## Timing
- Time to First Token: 0.45s
- Total Time: 1.23s
```

## Optimization Strategies

1. **首先使用更便宜的 models 测试：**
   ```bash
   singleshot chat -p "Test" -P openai -m gpt-4o-mini -d -r report.md
   ```

2. **减少 tokens：**
   - 缩短 system prompts
   - 使用 `--max-tokens` 限制 output
   - 在 system prompt 中添加 "be concise"

3. **本地测试 (免费)：**
   ```bash
   singleshot chat -p "Test" -P ollama -m llama3.2 -d -r report.md
   ```

## Example: Full Optimization

```bash
# Step 1: Baseline (verbose)
singleshot chat \
  -p "How do I write a Rust function to add two numbers?" \
  -s "You are an expert Rust programmer with 10 years experience" \
  -P openai -d -r v1.md

# Step 2: 读取指标
cat v1.md
# Expected: ~130 input tokens, ~400 output tokens

# Step 3: Optimized version
singleshot chat \
  -p "Rust function: add(a: i32, b: i32) -> i32" \
  -s "Rust expert. Code only." \
  -P openai --max-tokens 100 -d -r v2.md

# Step 4: 比较
echo "=== COMPARISON ==="
grep "Total Cost" v1.md v2.md
grep "Total Tokens" v1.md v2.md
```

## Quick Reference

```bash
# 使用完整详情测试
singleshot chat -p "prompt" -P openai -d -r report.md

# 提取指标
grep -E "(Input|Output|Total)" report.md

# 比较报告
diff report1.md report2.md

# Vision test
singleshot chat -p "Describe" -i image.jpg -P openai -d -r report.md

# List models
singleshot models -P openai

# 测试连接
singleshot ping -P openai
```

## Environment Variables

```bash
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENROUTER_API_KEY="sk-or-..."
```

## Best Practices

1. **Always use `-d`** 获取详细的 token 指标
2. **Always use `-r`** 保存报告
3. **Always `cat` reports** 分析指标
4. **Test variations** 并比较成本
5. **Set `--max-tokens`** 控制成本
6. **Use gpt-4o-mini** 用于测试 (更便宜)

## Troubleshooting

- **No metrics**: 确保使用了 `-d` flag
- **No report file**: 确保使用了 `-r` flag
- **High costs**: 切换到 gpt-4o-mini 或 Ollama
- **Connection issues**: 运行 `singleshot ping -P <provider>`
