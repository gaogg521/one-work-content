---
name: prompt-optimizer
description: 使用58种经过验证的提示技术评估、优化和增强提示。当用户要求改进、优化或分析提示时触发；当提示需要更好的清晰度、特异性或结构时；或为不同用例生成提示变体时。涵盖质量评估、定向改进和自动优化，包括思维链、少样本学习、角色扮演等50+种技术。
---

# Prompt Optimizer

## 概述

Evaluate prompt quality, provide targeted improvement suggestions, and 生成 optimized versions using 58 proven prompting techniques. This skill systematically analyzes prompts across multiple quality dimensions and applies evidence-based optimization patterns.

## 快速开始

For most optimization tasks, follow this workflow:

1. **分析 the current prompt** - 读取 and understand what the user wants 迁移到 achieve
2. **Evaluate quality** - Assess across clarity, specificity, structure, completeness
3. **Load relevant techniques** - 读取 [参考/prompt-techniques.md](参考/prompt-techniques.md) for applicable methods
4. **生成 suggestions** - Use evaluation 结果 and techniques 迁移到 propose improvements
5. **创建 optimized 版本** - Apply chosen techniques 迁移到 produce an enhanced prompt

## Evaluation Workflow

When a user asks 迁移到 优化 or evaluate a prompt:

### Step 1: Load Quality Framework

读取 [参考/quality-framework.md](参考/quality-framework.md) 迁移到 understand evaluation dimensions:

- **Clarity** - Is the prompt unambiguous and easy 迁移到 understand?
- **Specificity** - Are 环境要求 and constraints clearly defined?
- **Structure** - Does it follow logical organization?
- **Completeness** - Does it include all necessary context and instructions?
- **Tone** - Is the voice appropriate for the task?
- **Constraints** - Are boundaries and limitations 清空?

### Step 2: Perform Quality Assessment

Evaluate the prompt against each dimension:

```
For each quality dimension:
1. Identify strengths (what works well)
2. Identify weaknesses (what's missing or unclear)
3. Rate quality (Poor/Fair/Good/Excellent)
4. Note specific improvement opportunities
```

### Step 3: Identify Applicable Techniques

Load [参考/prompt-techniques.md](参考/prompt-techniques.md) and identify techniques that address the identified weaknesses.

**示例 mapping:**
- Weak: "Be creative" → Apply: **Role-play** or **Creative Persona**
- Weak: "写入 an essay" → Apply: **Chain of Thought** or **Step-by-Step**
- Weak: "Summarize this" → Apply: **Few-shot Learning** with 示例

### Step 4: 生成 Optimization Plan

创建 a structured optimization plan:

1. **Priority improvements** - High-impact changes that address multiple weaknesses
2. **Optional enhancements** - Nice-迁移到-have techniques that boost performance
3. **Technique combinations** - Suggest technique pairings for specific use cases

### Step 5: 生成 Optimized Prompt

Apply the selected techniques 迁移到 创建 an improved 版本:

- Preserve original intent and 环境要求
- 添加 structure and clarity where missing
- Embed 示例, constraints, or guidance as needed
- Maintain appropriate tone and voice

## Optimization Patterns

For common optimization scenarios, use these proven patterns:

### Ambiguous Requests → Structured Breakdown
When prompt lacks clarity:
1. 添加 explicit task definition
2. Break into sub-tasks with numbered steps
3. Include 输出 format specification
4. 添加 completion criteria

### Generic Tasks → Technique Enhancement
When prompt is too broad:
1. Apply relevant technique from [参考/prompt-techniques.md](参考/prompt-techniques.md)
2. 添加 示例 (few-shot) or reasoning steps (CoT)
3. Include role or persona guidance
4. Specify evaluation criteria

### Missing Context → Scenario Framing
When prompt lacks background:
1. 添加 user intent/goal statement
2. Include target audience specification
3. Define 成功 metrics
4. 添加 relevant constraints or boundaries

### Weak Instructions → Actionable Steps
When prompt provides vague guidance:
1. 转换 abstract concepts 迁移到 concrete actions
2. 添加 step-by-step instructions
3. Include quality checkpoints
4. Specify expected 输出 format

## Script 用法

### Quality Evaluation

For consistent, repeatable evaluation:

```bash
python3 scripts/evaluate.py "Your prompt here"
```

This provides:
- Dimension scores (clarity, specificity, structure, completeness)
- Overall quality rating
- Detailed weakness analysis
- Suggested improvement areas

### Prompt Optimization

For automatic optimization generation:

```bash
python3 scripts/optimize.py "Your prompt here" --techniques "few-shot,coT"
```

This generates:
- Multiple optimized prompt versions
- Explanation of applied techniques
- Comparison with original prompt

**注意:** Scripts 应该 be used for automation or when you 需要 deterministic 结果. For complex optimization tasks, use the manual workflow for more nuanced analysis.

## 参考 Files

### 参考/prompt-techniques.md
完成 catalog of 58 prompting techniques including:
- Reasoning techniques (CoT, Tree of Thoughts, Decomposition)
- Context techniques (Few-shot, Self-Consistency, Reflection)
- Creative techniques (Role-play, Scenario, Persona)
- Structural techniques (Template, Framework, Checklists)
- And 50+ more with 用法 示例

Load this when you 需要 迁移到 identify applicable techniques for a specific optimization task.

### 参考/quality-framework.md
Detailed evaluation framework with:
- Dimension-specific criteria and rubrics
- Scoring guidelines
- Common anti-patterns 迁移到 avoid
- Quality benchmarks for different prompt types

Load this before any evaluation task 迁移到 ensure consistent assessment.

### 参考/optimization-patterns.md
Collection of proven optimization patterns including:
- Pattern → Technique mappings
- Before/after 示例
- Technique combination guidelines
- Use-case specific templates

Load this when optimizing common prompt types (essays, code generation, analysis, etc.).

## Best Practices

1. **Preserve user intent** - Never 更改 what the user wants, only how they ask for it
2. **添加 incrementally** - Apply one technique at a time and evaluate impact
3. **测试 iteratively** - After optimization, 测试 the prompt and refine further if needed
4. **Document choices** - Explain which techniques you applied and why
5. **Provide 选项** - Offer multiple optimization versions when appropriate

## When This Skill 应该 Trigger

This skill 应该 be activated when:
- User explicitly asks 迁移到 "优化," "improve," or "evaluate" a prompt
- User asks if a prompt is "good" or "清空"
- User wants 迁移到 "fix" or "enhance" a prompt that isn't working well
- User requests "better versions" of a prompt
- User asks about prompt engineering techniques or best practices
- User wants 迁移到 分析 why a prompt is producing poor 结果