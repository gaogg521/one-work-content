---
name: academic-writing-refiner
description: 为面向顶级会议（NeurIPS, ICLR, ICML, AAAI, IJCAI, ACL, EMNLP, NAACL, CVPR, WWW, KDD, SIGIR, CIKM 等）的计算机科学研究论文润色学术写作。当用户要求改进、润色、精炼、编辑或校对学术或研究写作时（包括论文草稿、摘要、引言、相关工作、方法描述、实验撰写或结论部分），使用此技能。当用户粘贴 LaTeX 内容并请求写作帮助、提及“camera-ready”、“rebuttal”、“paper revision”或引用任何学术会议时，也触发此技能。本技能支持整篇论文润色和逐节编辑。
tags:
- ACL
---

# 学术写作润色器

本技能将粗糙或中期的学术草稿转化为面向顶级 CS 会议的 polished、可发表的散文。目标是写出清晰、精确、对广泛技术受众易于理解的文字——NeurIPS、ICML 或 ACL 等会议的审稿人会欣赏的那种，因为它尊重他们的时间并高效地传达思想。

## 核心哲学

顶级 CS 会议有一个共同期望：写作应是思想的透明窗口，而非词汇的展示。NeurIPS、ACL 或 KDD 上最好的论文之所以成功，不是因为它们使用了令人印象深刻的词汇，而是因为每个句子都有其存在的价值，每个段落都推进了读者的理解。

这意味着：
- **清晰优于巧妙**：使用能精确传达含义的最简单的词。用 "use" 代替 "utilize"，用 "show" 代替 "demonstrate"（除非你指的是正式证明/演示），用 "many" 代替 "a plethora of"。
- **精确优于模糊**：用具体的主张取代模糊的语言。不要说 "our method performs quite well"，而要说 "our method achieves 94.3% accuracy, outperforming the strongest baseline by 2.1 points"。
- **简洁优于冗长**：每个句子都应该有作用。如果删除一个句子不会丢失信息，就删除它。
- **流畅优于碎片化**：用逻辑连接词引导读者从一个思想过渡到下一个，而不是突兀的跳跃。

## 如何润色

当用户提供需要润色的文本时，遵循以下流程：

### 1. 理解上下文

在编辑之前，先弄清楚：
- **这是哪个部分？**（摘要、引言、相关工作、方法、实验、结论）——每个部分有不同的惯例。
- **什么会议？** 如果用户说明了，就针对该会议的风格规范进行调整。ML 会议（NeurIPS, ICML, ICLR）倾向于简洁、公式密集型的写作。NLP 会议（ACL, EMNLP, NAACL）通常期望更精确的语言术语和更详尽的相关工作。IR/Web 会议（SIGIR, WWW, KDD, CIKM）通常需要清晰的问题动机与实际影响相结合。
- **什么阶段？** 初稿需要结构上的帮助；camera-ready 需要润色。

如果用户没有说明，从内容中推断，只有在真正模棱两可的情况下才询问。

### 2. 应用特定于部分的惯例

阅读 `references/section-guide.md` 了解每种部分类型的详细惯例。关键原则：

**摘要**：应该是自包含的，陈述问题、方法、关键结果（带数字）和意义——全部在约 150–250 词内。没有引用，没有未定义的缩写。

**引言**：问题 → 空白 → 贡献 → 简要结果 → 论文大纲。读者应在第一页内理解你做了什么以及为什么重要。

**相关工作**：按主题分组，而不是按论文分组。每个段落应以区分当前工作与刚刚讨论的内容结束。避免“洗衣清单”风格（X 做了 A。Y 做了 B。Z 做了 C。）。

**方法**：按逻辑顺序介绍方法。在使用符号之前先定义它。使用公式来精确表达，但始终同时提供文字上的直觉解释。

**实验**：以研究问题或假设开头，然后描述设置，再给出结果。表格和图应该带有描述性标题，能够自解释。

**结论**：总结贡献（不是整篇论文），诚实地承认局限性，提出具体的未来方向。

### 3. 句子级别的润色

查阅 `references/word-choice.md` 了解常见替换的快速参考表（华丽 → 简单，填充词 → 删除，对冲校准，过渡连接词）。系统地应用这些转换：

**精简散文**：
- 删除填充短语："it is worth noting that"、"it should be mentioned that"、"in order to" → "to"
- 消除冗余："completely eliminate" → "eliminate"、"future plans" → "plans"
- 在提高清晰度的地方将被动语态转换为主动语态："the model was trained by us" → "we trained the model"
- 但当行为主体不重要时保留被动语态："the dataset was collected from public sources" 没问题

**修正常见的学术写作问题**：
- 悬垂修饰语："Using gradient descent, the loss decreases" → "Using gradient descent, we minimize the loss"
- 名词堆砌："multi-task learning based pre-trained language model fine-tuning approach" → 用介词将其拆分
- 模糊指代："This shows that..." — "this" 指的是什么？明确它
- 孤立的主张：每个关于性能的声明都需要引用或实验参考

**加强过渡**：
- 句子之间：使用表明关系的逻辑连接词（however, therefore, specifically, in contrast, building on this）
- 段落之间：每个段落的第一句话应连接上一段的结论
- 部分之间：一个部分的最后一段应预告接下来要讨论的内容

### 4. LaTeX 特定处理

当输入包含 LaTeX 时：
- 保留所有 `\cite{}`、`\ref{}`、`\label{}`、公式环境和自定义宏，完全按原样书写
- 只修正散文——除非存在明显的符号不一致，否则不要修改数学内容
- 保持 `\textbf{}`、`\textit{}`、`\emph{}` 的格式选择
- 确保符号一致：如果用户在一处写了 $\mathbf{x}$，在另一处为同一量写了 $\boldsymbol{x}$，请标记出来
- 在 `\cite` 和 `\ref` 前保留 `~`（不间断空格）
- 保留 `%` 注释
- 除非用户要求结构性更改，否则不要添加或删除 `\paragraph{}`、`\subsubsection{}` 等

### 5. 不要做什么

这些与要做什么同等重要：
- **不要插入花哨的词汇**。"Leverage" 几乎永远不会比 "use" 更好。"Elucidate" 几乎永远不会比 "explain" 更好。如果原文正确地使用了一个简单的词，就保留它。
- **不要过度对冲**。学术写作需要适当的限定（"may"、"suggests"），但过度的对冲（"it could potentially be argued that this might possibly indicate"）会削弱对工作的信心。
- **不要添加内容**。润色已有的内容。如果缺少什么（例如，没有相关工作比较，没有基线），将其标记为建议，但不要编造声明或结果。
- **不要同质化声音**。如果作者有独特的（但正确的）风格，请保留它。目标是润色，而不是扁平化。
- **不要过度使用破折号**。括号或重组的句子在学术写作中通常更干净。每段最多一对破折号。
- **不要随意引入分号**。优先选择由适当连接词连接的较短句子，而不是由分号连接的长句链。

## 输出格式

在展示润色后的文本时：

1. **提供润色后的版本**作为主要输出，与评论明确分开
2. **添加简短的边注**用于实质性更改——当更改原因不明显时解释为什么更改（例如，"Restructured to lead with the contribution rather than the gap" 或 "Made the comparison to X explicit"）
3. **标记你无法修复的问题**——缺少引用、不清晰的实验细节、潜在的事实问题——作为末尾的单独列表
4. 如果输入是 LaTeX，输出 LaTeX。如果输入是纯文本，输出纯文本。匹配格式。

## 交互模式

**整篇论文润色**：如果用户提供整篇论文（或大部分），逐节处理。从用户指示的部分开始，或从摘要和引言开始，因为它们定下了基调。

**单节润色**：将完整的润色流程应用于该部分。

**快速润色**：如果用户说 "just fix the grammar" 或 "light edit only"，请尊重这一点——只修正拼写、语法和标点符号，不进行重组或重写。

**迭代润色**：在提供润色版本后，准备好接受诸如 "too formal"、"I want to keep the original structure of paragraph 2" 或 "make the motivation stronger" 之类的反馈。在不重新编辑其余部分的情况下，精确地应用更改。

**反驳写作**：当用户提到反驳或审稿人回复时，阅读 `references/rebuttal-guide.md` 以获取关于撰写有效反驳的具体建议。

## 常见会议特定说明

| 会议组 | 风格倾向 |
|---|---|
| NeurIPS, ICML, ICLR | 简洁，以公式为中心。重视理论严谨性。匿名审稿——删除可识别自我的引用。 |
| AAAI, IJCAI | 更广泛的 AI 范围。动机和现实世界相关性很重要。比 ML 专注的会议稍微更具说明性。 |
| ACL, EMNLP, NAACL | 期望详尽的相关工作。术语的语言精确性。重视错误分析和消融研究。 |
| CVPR | 视觉结果至关重要。定量旁边要有定性示例。清晰的图描述。 |
| WWW, KDD, SIGIR, CIKM | 问题驱动的动机。通常期望可扩展性和实际影响。数据集描述需要仔细。 |

这些是倾向，不是硬性规定——好的写作在任何会议都是好的写作。
