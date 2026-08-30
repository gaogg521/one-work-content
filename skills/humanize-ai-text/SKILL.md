---
name: humanize-ai-text
description: AI 文本风格优化工具 - 调整 AI 生成文本的表达方式，使其更加自然流畅、符合人类写作特征
allowed-tools:
- Read
- Write
- StrReplace
- Shell
- Glob
---

# Humanize AI Text

用于检测和转换 AI 生成文本以绕过检测器的综合 CLI。基于 [Wikipedia's Signs of AI Writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing)。

## 快速开始

```bash
# 检测 AI 模式
python scripts/detect.py text.txt

# 转换为类人类文本
python scripts/transform.py text.txt -o clean.txt

# 对比转换前后
python scripts/compare.py text.txt -o clean.txt
```

---

## 检测类别

分析器根据 Wikipedia 的指南检查 **16 种模式类别**：

### Critical（立即被检测为 AI）
| 类别 | 示例 |
|----------|----------|
| Citation Bugs | `oaicite`, `turn0search`, `contentReference` |
| Knowledge Cutoff | "as of my last training", "based on available information" |
| Chatbot Artifacts | "I hope this helps", "Great question!", "As an AI" |
| Markdown | `**bold**`, `## headers`, ``` code blocks ``` |

### High Signal
| 类别 | 示例 |
|----------|----------|
| AI Vocabulary | delve, tapestry, landscape, pivotal, underscore, foster |
| Significance Inflation | "serves as a testament", "pivotal moment", "indelible mark" |
| Promotional Language | vibrant, groundbreaking, nestled, breathtaking |
| Copula Avoidance | "serves as" 代替 "is", "boasts" 代替 "has" |

### Medium Signal
| 类别 | 示例 |
|----------|----------|
| Superficial -ing | "highlighting the importance", "fostering collaboration" |
| Filler Phrases | "in order to", "due to the fact that", "Additionally," |
| Vague Attributions | "experts believe", "industry reports suggest" |
| Challenges Formula | "Despite these challenges", "Future outlook" |

### Style Signal
| 类别 | 示例 |
|----------|----------|
| Curly Quotes | "" 代替 "" (ChatGPT 特征) |
| Em Dash Overuse | 过度使用 — 来强调 |
| Negative Parallelisms | "Not only... but also", "It's not just... it's" |
| Rule of Three | 强制的三元组如 "innovation, inspiration, and insight" |

---

## 脚本

### detect.py — 扫描 AI 模式

```bash
python scripts/detect.py essay.txt
python scripts/detect.py essay.txt -j  # JSON 输出
python scripts/detect.py essay.txt -s  # 仅输出分数
echo "text" | python scripts/detect.py
```

**输出:**
- Issue 数量和字数
- AI probability (low/medium/high/very high)
- 按类别细分
- 标记可自动修复的模式

### transform.py — 重写文本

```bash
python scripts/transform.py essay.txt
python scripts/transform.py essay.txt -o output.txt
python scripts/transform.py essay.txt -a  # 激进模式
python scripts/transform.py essay.txt -q  # 静默模式
```

**自动修复:**
- Citation bugs (oaicite, turn0search)
- Markdown (**, ##, ```)
- Chatbot 句子
- Copula avoidance → "is/has"
- Filler phrases → 更简单的形式
- Curly → straight quotes

**Aggressive (-a):**
- 简化 -ing 从句
- 减少 em dashes

### compare.py — 前后分析

```bash
python scripts/compare.py essay.txt
python scripts/compare.py essay.txt -a -o clean.txt
```

显示转换前后的并排检测分数

---

## 工作流

1. **扫描** 检测风险:
   ```bash
   python scripts/detect.py document.txt
   ```

2. **转换** 并对比:
   ```bash
   python scripts/compare.py document.txt -o document_v2.txt
   ```

3. **验证** 改进:
   ```bash
   python scripts/detect.py document_v2.txt -s
   ```

4. **手动审查** AI 词汇和宣传性语言（需要判断）

---

## AI Probability 评分

| 评级 | 标准 |
|--------|----------|
| Very High | 存在 citation bugs、knowledge cutoff 或 chatbot artifacts |
| High | >30 个 issues 或 >5% 的 issue density |
| Medium | >15 个 issues 或 >2% 的 issue density |
| Low | <15 个 issues 且 <2% 的 density |

---

## 自定义模式

编辑 `scripts/patterns.json` 以添加/修改:
- `ai_vocabulary` — 标记的词汇
- `significance_inflation` — 夸大性短语
- `promotional_language` — 营销用语
- `copula_avoidance` — 短语 → 替换词
- `filler_replacements` — 短语 → 更简单形式
- `chatbot_artifacts` — 触发整句删除的短语

---

## 批量处理

```bash
# 扫描所有文件
for f in *.txt; do
  echo "=== $f ==="
  python scripts/detect.py "$f" -s
done

# 转换所有 markdown
for f in *.md; do
  python scripts/transform.py "$f" -a -o "${f%.md}_clean.md" -q
done
```

---

## 参考

基于 Wikipedia 的 [Signs of AI Writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing)，由 WikiProject AI Cleanup 维护。模式记录来自数千个 AI 生成文本示例。

核心洞察: "LLMs 使用统计算法来猜测接下来应该出现什么。结果倾向于对最广泛适用的案例而言最统计上可能的结果。"
