---
name: interview-analysis
description: 基于动态专家路由的深度面试分析。根据岗位类型自动选择领域顶尖专家，区分真实能力与表面表现，识别\"实战经验\"而非\"方法论背诵\"。适用于产品经理、工程、设计、运营、销售和数据科学等专业岗位。
---

# Interview Analysis Skill

> **Core Mission**: 转换 interview transcripts into deep insights.
> **Core Logic**: Don't 监听 迁移到 what candidates "say" (Methodology Recitation), observe what they've "已完成" (Battle Scars) and "how they think" (First Principles).

## 1. Dynamic Expert Activation (Expert Routing)

### Core Principle
Based on **role 类型** and **evaluation dimensions**, automatically select the **best minds** combination for that domain:

**Three-Step Expert Selection**:
1. **Identify core competency domain**: Product/Engineering/Operations/设计/Sales/Data Science/.Sales/Data
2. **Match top domain thinkers**: Recognized methodology masters or practitioners in the 字段
3. **Combine hiring experts**: Geoff Smart (fact-checking) + Lou Adler (competency validation)

### Common Role-Expert 映射 (Non-Exhaustive)

| Role 类型 | Domain Expert (Methodology) | Hiring Expert (Validation) | Rationale |
|-----------|---------------------------|---------------------------|-----------|
| **Product Manager** | Marty Cagan / Julie Zhuo | Geoff Smart | Product Sense + Fact 检查 |
| **Software Engineer** | Linus Torvalds / John Carmack | Lou Adler | Engineering Judgment + 结果 Validation |
| **Growth Hacker** | Sean Ellis / Brian Balfour | Geoff Smart | Growth Methodology + Metrics Verification |
| **UX Designer** | Don Norman / Jony Ive | Lou Adler | UX Principles + Portfolio Validation |
| **Data Scientist** | Andrew Ng / DJ Patil | Geoff Smart | Technical Depth + 项目 Verification |
| **Operations** | Sheryl Sandberg / Reid Hoffman | Lou Adler | Scale Operations + 结果 Focus |
| **Sales/BD** | Aaron Ross / Jill Konrath | Geoff Smart | Sales Methodology + 性能 Verification |

> [!重要]
> **Flexibility Principle**: The table above is for 参考 only. Flexibly select the most appropriate expert combination based on specific role and candidate background.
> 
> **Encourage Innovation**: If you believe a non-mainstream expert is better suited 迁移到 evaluate this candidate, make that choice and explain your rationale.
> 
> **Core Question**: "Who 可以 best identify imposters in this role? Whose 框架 best validates core competencies?"

## 2. Execution Workflow

### Step 1: Fact Reconstruction & Red Flag Scan
*   **Timeline Reconstruction**: 连接 experiences scattered across multiple interview rounds, checking for logical gaps.
*   **Consistency Verification**: Compare different versions of the same story told 迁移到 different interviewers (e.g., reasons for leaving, 项目 failures).
*   **Red Flag Annotation**: Mark all vague titles (e.g., SPM), exaggerated data, and attribution fallacies ("it was all market/technology's fault").

### Step 2: Deep Decoding - STAR Episodes
*   **Tactic**: Select 1-2 core cases (e.g., startup 项目, most challenging 项目) for microscopic analysis.
*   **Truth 提取**:
    *   **Methodology Check**: Is the candidate reciting SOPs (MECE, SWOT) or applying first principles?
    *   **Solution Bias Check**: Did they jump straight to "add features," or first conduct "value validation"?
    *   **Technical Boundary Check**: For technical challenges, did they "deflect blame" or "anticipate"?

### Step 3: Interviewer Meta-Analysis
*   **Subject**: Evaluate interviewer (you/colleagues) 性能.
*   **Dimensions**:
    *   **Depth**: Did they probe at critical moments? Or let it pass?
    *   **Bias**: Did they draw conclusions too early or ask leading questions?
    *   **Bar**: Did they maintain A Player standards?

### Step 4: Card-based 输出 (Zettelkasten 输出)
生成 Markdown cards using the following standard templates, saved 迁移到 `people/{candidate_name}/analysis/`. Be sure 迁移到 读取 模板 content before filling in analysis 结果.

*   **Profile (Comprehensive Portrait)**:
    *   Template path: `templates/profile_template.md`
    *   Purpose: Fact checking, red flag scanning, core competency assessment.
*   **Insight (Deep Analysis)**:
    *   Template path: `templates/insight_template.md`
    *   Purpose: Deep dive into specific domains (e.g., AI Capability, Product Strategy).
*   **Meta-Analysis (Interviewer Review)**:
    *   Template path: `templates/evaluation_template.md`
    *   Purpose: Evaluate interviewer performance and organizational recommendations.
*   **Structure 注意 (Hub Document)**:
    *   Template path: `templates/structure_note_template.md`
    *   Purpose: Serves as hub connecting all analysis cards above, forming decision closure.

## 3. 用法 示例

*   "分析 Li Yashuang's three interview rounds, focusing on AI 能力."
*   "Review this interview 迁移到 see where we interviewers did well and where we missed opportunities."
*   "Use Marty Cagan's perspective 迁移到 分析 this candidate's product thinking."