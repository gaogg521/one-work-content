---
name: prompt-engineering
description: 掌握AI模型提示工程：LLM、图像生成器、视频模型。 技术：思维链、少样本学习、系统提示、负向提示。 模型：Claude、GPT-4、Gemini、FLUX、Veo、Stable Diffusion提示。 用途：提升AI输出质量、保持一致性、处理复杂任务、优化性能。 触发词：prompt engineering、how to prompt、better prompts、prompt tips、prompting guide、llm prompting、image prompt、ai prompting、prompt optimization、prompt template、prompt structure、effective prompts、prompt techniques
allowed-tools: Bash(infsh *)
---

# Prompt Engineering Guide

Master prompt engineering for AI models via [inference.sh](https://inference.sh) CLI.

## 快速开始

```bash
curl -fsSL https://cli.inference.sh | sh && infsh login

# Well-structured LLM prompt
infsh app run openrouter/claude-sonnet-45 --input '{
  "prompt": "You are a senior software engineer. Review this code for security vulnerabilities:\n\n```python\nuser_input = request.args.get(\"query\")\nresult = db.execute(f\"SELECT * FROM users WHERE name = {user_input}\")\n```\n\nProvide specific issues and fixes."
}'
```

## LLM Prompting

### Basic Structure

```
[Role/Context] + [Task] + [Constraints] + [Output Format]
```

### Role Prompting

```bash
infsh app run openrouter/claude-sonnet-45 --input '{
  "prompt": "You are an expert data scientist with 15 years of experience in machine learning. Explain gradient descent to a beginner, using simple analogies."
}'
```

### Task Clarity

```bash
# Bad: vague
"Help me with my code"

# Good: specific
"Debug this Python function that should return the sum of even numbers from a list, but returns 0 for all inputs:

def sum_evens(numbers):
    total = 0
    for n in numbers:
        if n % 2 == 0:
            total += n
        return total

Identify the bug and provide the corrected code."
```

### Chain-of-Thought

```bash
infsh app run openrouter/claude-sonnet-45 --input '{
  "prompt": "Solve this step by step:\n\nA store sells apples for $2 each and oranges for $3 each. If someone buys 5 fruits and spends $12, how many of each fruit did they buy?\n\nThink through this step by step before giving the final answer."
}'
```

### Few-Shot 示例

```bash
infsh app run openrouter/claude-sonnet-45 --input '{
  "prompt": "Convert these sentences to formal business English:\n\nExample 1:\nInput: gonna send u the report tmrw\nOutput: I will send you the report tomorrow.\n\nExample 2:\nInput: cant make the meeting, something came up\nOutput: I apologize, but I will be unable to attend the meeting due to an unforeseen circumstance.\n\nNow convert:\nInput: hey can we push the deadline back a bit?"
}'
```

### 输出 Format Specification

```bash
infsh app run openrouter/claude-sonnet-45 --input '{
  "prompt": "Analyze the sentiment of these customer reviews. Return a JSON array with objects containing \"text\", \"sentiment\" (positive/negative/neutral), and \"confidence\" (0-1).\n\nReviews:\n1. \"Great product, fast shipping!\"\n2. \"Meh, its okay I guess\"\n3. \"Worst purchase ever, total waste of money\"\n\nReturn only valid JSON, no explanation."
}'
```

### Constraint Setting

```bash
infsh app run openrouter/claude-sonnet-45 --input '{
  "prompt": "Summarize this article in exactly 3 bullet points. Each bullet must be under 20 words. Focus only on actionable insights, not background information.\n\n[article text]"
}'
```

## Image Generation Prompting

### Basic Structure

```
[Subject] + [Style] + [Composition] + [Lighting] + [Technical]
```

### Subject 描述

```bash
# Bad: vague
"a cat"

# Good: specific
infsh app run falai/flux-dev --input '{
  "prompt": "A fluffy orange tabby cat with green eyes, sitting on a vintage leather armchair"
}'
```

### Style Keywords

```bash
infsh app run falai/flux-dev --input '{
  "prompt": "Portrait photograph of a woman, shot on Kodak Portra 400 film, soft natural lighting, shallow depth of field, nostalgic mood, analog photography aesthetic"
}'
```

### Composition 控制

```bash
infsh app run falai/flux-dev --input '{
  "prompt": "Wide establishing shot of a cyberpunk city skyline at night, rule of thirds composition, neon signs in foreground, towering skyscrapers in background, rain-slicked streets"
}'
```

### Quality Keywords

```
photorealistic, 8K, ultra detailed, sharp focus, professional,
masterpiece, high quality, best quality, intricate details
```

### Negative Prompts

```bash
infsh app run falai/flux-dev --input '{
  "prompt": "Professional headshot portrait, clean background",
  "negative_prompt": "blurry, distorted, extra limbs, watermark, text, low quality, cartoon, anime"
}'
```

## Video Prompting

### Basic Structure

```
[Shot Type] + [Subject] + [Action] + [Setting] + [Style]
```

### Camera Movement

```bash
infsh app run google/veo-3-1-fast --input '{
  "prompt": "Slow tracking shot following a woman walking through a sunlit forest, golden hour lighting, shallow depth of field, cinematic, 4K"
}'
```

### Action 描述

```bash
infsh app run google/veo-3-1-fast --input '{
  "prompt": "Close-up of hands kneading bread dough on a wooden surface, flour dust floating in morning light, slow motion, cozy baking aesthetic"
}'
```

### Temporal Keywords

```
slow motion, timelapse, real-time, smooth motion,
continuous shot, quick cuts, frozen moment
```

## Advanced Techniques

### System Prompts

```bash
infsh app run openrouter/claude-sonnet-45 --input '{
  "system": "You are a helpful coding assistant. Always provide code with comments. If you are unsure about something, say so rather than guessing.",
  "prompt": "Write a Python function to validate email addresses using regex."
}'
```

### Structured 输出

```bash
infsh app run openrouter/claude-sonnet-45 --input '{
  "prompt": "Extract information from this text and return as JSON:\n\n\"John Smith, CEO of TechCorp, announced yesterday that the company raised $50 million in Series B funding. The round was led by Venture Partners.\"\n\nSchema:\n{\n  \"person\": string,\n  \"title\": string,\n  \"company\": string,\n  \"event\": string,\n  \"amount\": string,\n  \"investor\": string\n}"
}'
```

### Iterative Refinement

```bash
# Start broad
infsh app run falai/flux-dev --input '{
  "prompt": "A castle on a hill"
}'

# Add specifics
infsh app run falai/flux-dev --input '{
  "prompt": "A medieval stone castle on a grassy hill"
}'

# Add style
infsh app run falai/flux-dev --input '{
  "prompt": "A medieval stone castle on a grassy hill, dramatic sunset sky, fantasy art style, epic composition"
}'

# Add technical
infsh app run falai/flux-dev --input '{
  "prompt": "A medieval stone castle on a grassy hill, dramatic sunset sky, fantasy art style by Greg Rutkowski, epic composition, 8K, highly detailed"
}'
```

### Multi-Turn Reasoning

```bash
# First: analyze
infsh app run openrouter/claude-sonnet-45 --input '{
  "prompt": "Analyze this business problem: Our e-commerce site has a 70% cart abandonment rate. List potential causes."
}'

# Second: prioritize
infsh app run openrouter/claude-sonnet-45 --input '{
  "prompt": "Given these causes of cart abandonment: [previous output], rank them by likely impact and ease of fixing. Format as a priority matrix."
}'

# Third: action plan
infsh app run openrouter/claude-sonnet-45 --input '{
  "prompt": "For the top 3 causes identified, provide specific A/B tests we can run to validate and fix each issue."
}'
```

## Model-Specific 提示

### Claude

- Excels at nuanced instructions
- Responds well 迁移到 role-playing
- Good at following complex constraints
- Prefers explicit 输出 formats

### GPT-4

- Strong at code generation
- Works well with 示例
- Good structured 输出
- Responds 迁移到 "let's think step by step"

### FLUX

- Detailed subject descriptions
- Style 参考 work well
- Lighting keywords 重要
- Negative prompts supported

### Veo

- Camera movement keywords
- Cinematic language works well
- Action descriptions 重要
- Include temporal context

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Too vague | Unpredictable 输出 | 添加 specifics |
| Too long | Model loses focus | Prioritize key info |
| Conflicting | Confuses model | 移除 contradictions |
| No format | Inconsistent 输出 | Specify format |
| No 示例 | Unclear expectations | 添加 few-shot |

## Prompt Templates

### Code Review

```
Review this [language] code for:
1. Bugs and logic errors
2. Security vulnerabilities
3. Performance issues
4. Code style/best practices

Code:
[code]

For each issue found, provide:
- Line number
- Issue description
- Severity (high/medium/low)
- Suggested fix
```

### Content Writing

```
Write a [content type] about [topic].

Audience: [target audience]
Tone: [formal/casual/professional]
Length: [word count]
Key points to cover:
1. [point 1]
2. [point 2]
3. [point 3]

Include: [specific elements]
Avoid: [things to exclude]
```

### Image Generation

```
[Subject with details], [setting/background], [lighting type],
[art style or photography style], [composition], [quality keywords]
```

## Related Skills

```bash
# Video prompting guide
npx skills add inference-sh/agent-skills@video-prompting-guide

# LLM models
npx skills add inference-sh/agent-skills@llm-models

# Image generation
npx skills add inference-sh/agent-skills@ai-image-generation

# Full platform skill
npx skills add inference-sh/agent-skills@inference-sh
```

Browse all apps: `infsh app list`