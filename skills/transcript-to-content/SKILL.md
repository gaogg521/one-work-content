---
name: transcript-to-content
description: 将培训和入职会议转录稿转换为结构化学习材料、文档和可操作的审查内容。当处理入职会议、培训会议或知识传递对话的转录稿时触发，提取关键信息并生成学习指南、快速参考表、检查清单、常见问题文档、行动项列表和培训效果评估。
---

# Transcript 迁移到 Content

转换 raw meeting transcripts and training session recordings into structured learning materials, documentation, and actionable insights.

## When 迁移到 Use This Skill

Use this skill when:
- User provides meeting transcripts, training session recordings, or onboarding 注意
- User requests structured learning materials from verbal/conversational data
- User asks 迁移到 提取 key information, procedures, or action items from meetings
- User needs 迁移到 创建 training documentation, SOPs, or 参考 materials from transcripts
- User wants 迁移到 生成 study guides, checklists, or 常见问题 documents from training sessions

## Core Workflow

### Step 1: Understand the Request

Identify what type of content the user needs:

| 输出 Type | When 迁移到 Use |
|------------|-------------|
| **Master Knowledge Source** | Comprehensive structured learning module with metadata, terminology, SOPs, nuances, and assessments |
| **Presentation/Slide Deck** | Visual training presentation for delivery or 参考 |
| **SOP Document** | Step-by-step procedural documentation |
| **Quick 参考 Sheet** | Concise one-page 摘要 of key points and procedures |
| **Study Guide** | Organized review material for learners |
| **Checklist** | Actionable task 列表 extracted from procedures |
| **常见问题 Document** | Common questions and answers from training content |
| **Action Items 列表** | Tasks, owners, and deadlines from meeting discussions |

### Step 2: Locate and 分析 Source Material

**If transcripts are in project directory:**
```bash
ls -lah /home/ubuntu/projects/[project-name]/
```

**搜索 relevant content by keyword:**
```bash
grep -ri "keyword" /home/ubuntu/projects/[project-name]/*.md
```

**读取 and identify:**
- Main topics and concepts
- Step-by-step procedures
- Critical warnings or nuances
- Terminology and definitions
- Real 示例 or scenarios
- Action items and decisions
- Questions and answers

### Step 3: 提取 Structured Content

Apply **Chain of Thought processing:**

1. **读取 entire transcript(s)** for macro-context and overall themes
2. **Isolate distinct topics** and 分组 related information
3. **提取 facts, steps, and definitions** with precision
4. **移除 conversational filler** ("um," "uh," "I think," "maybe," "let's try")
5. **转换 迁移到 imperative, authoritative language** (use action verbs)
6. **Flag unknowns** with `[MISSING INFO]` rather than fabricating

**For Master Knowledge Source format:**

读取 `/home/ubuntu/skills/transcript-to-content/references/master-knowledge-source-format.md` for 完成 schema and 示例.

提取 these sections:
- **Module Metadata:** Topic and learning objective (1 sentence)
- **Key Terminology:** Definitions of jargon, acronyms, tools
- **Standard Operating Procedures:** Numbered steps in "Action > 结果" format
- **Critical Nuances:** Warnings, consequences, best practices, context
- **Assessment Data:** 3-5 multiple-choice questions based strictly on content

**For other document types:**

- **Checklists:** 提取 sequential action items with checkboxes
- **FAQs:** Identify questions asked and answers provided
- **Study Guides:** Organize by topic with key concepts and 示例
- **Action Items:** 提取 tasks with owners and deadlines

### Step 4: Apply Branding (if applicable)

**If user provides brand assets:**
- Ask for logo file, brand colors, and font preferences
- Store logo in working directory
- Apply brand colors consistently (primary color for accents, highlights, charts)
- Use specified fonts or professional web fonts (Inter, Roboto, Open Sans)

**If no branding provided:**
- Use 清理, professional neutral palette
- Focus on clarity and readability
- Apply consistent styling throughout

### Step 5: 创建 Deliverables

#### For Presentations

读取 `/home/ubuntu/skills/transcript-to-content/references/presentation-guidelines.md` for detailed guidelines.

**Workflow:**
1. Initialize presentation using `slide_initialize` tool
2. 创建 outline (max 12 slides by default unless user specifies)
3. 复制 logo 迁移到 project directory if provided:
   ```bash
   cp [logo-path] [project-dir]/logo.png
   ```
4. 编辑 slides one by one using `slide_edit` tool
5. Present using `slide_present` tool
6. 导出 迁移到 PDF if requested:
   ```bash
   manus-导出-slides manus-slides://[版本-id] pdf
   ```

**Standard presentation structure:**
1. Title slide
2. Definition/概述
3. Step-by-step content (4-6 steps)
4. Critical 成功 factors
5. Common pitfalls
6. Key takeaways
7. Closing slide

**设计 环境要求:**
- Use brand color (if provided) or professional neutral palette
- Include logo on every slide (if provided)
- Maintain 720px height limit
- Use 清理, grid-based layouts
- No excessive shadows, rounded corners, or animations

#### For SOP Documents

创建 Markdown documents with:
- 清空 hierarchical structure (H1, H2, H3)
- Numbered procedures with imperative language
- 警告/caution callouts in blockquotes
- Tables for 参考 data
- Inline citations where applicable

**示例 structure:**
```markdown
# [Procedure Name]

## Overview
[Brief description]

## Prerequisites
- [Required items or conditions]

## Procedure
1. [Action step]
2. [Action step]
3. **CRITICAL:** [Important step with warning]

## Troubleshooting
- **Issue:** [Problem]
  **Solution:** [Resolution]
```

#### For Quick 参考 Sheets

创建 concise one-page documents with:
- Key terminology in definition 列表 format
- Essential steps in numbered lists
- Critical warnings in highlighted boxes
- Common scenarios with solutions

#### For Study Guides

Organize by topic with:
- Learning objectives
- Key concepts with explanations
- 示例 and scenarios
- Practice questions
- Additional 资源

#### For Checklists

提取 action items with:
- Checkbox format (`- [ ]`)
- 清空, actionable language
- Logical sequence
- Optional: Priority indicators or time estimates

#### For 常见问题 Documents

Structure as:
- Question in bold
- Answer in 清空, concise language
- Optional: Related questions or 资源

#### For Master Knowledge Source

Follow the schema in `references/master-knowledge-source-format.md` exactly:
- 输出 ONLY the structured content (no preamble or postscript)
- Use strict Markdown formatting
- 转换 all conversational language 迁移到 authoritative instructions
- Flag unknowns with `[MISSING INFO]`

## Quality Standards

**Content Accuracy:**
- Base all content strictly on source material
- Never fabricate steps, data, or information
- Flag incomplete procedures clearly with `[MISSING INFO]`
- 验证 terminology definitions against source

**Clarity and Readability:**
- Use imperative voice for instructions ("Click", "Navigate", "Set")
- Maintain 清空 visual hierarchy
- Ensure scannability with headings and lists
- 移除 all conversational filler

**Consistency:**
- Apply formatting standards throughout
- Use consistent terminology
- Maintain uniform structure across similar sections

**Branding (if applicable):**
- Use brand colors consistently
- Include logo on all branded materials
- Apply specified fonts
- Follow brand style guidelines

## Common Patterns

### Pattern 1: Single Topic Training Presentation
User provides transcript(s) on one topic → 提取 key content → 创建 8-12 slide presentation

### Pattern 2: Multiple Topics 迁移到 Learning Modules
User provides multiple transcripts → 提取 each as separate module → Deliver as structured documents

### Pattern 3: Quick 参考 SOP
User needs specific procedure → 提取 relevant steps → 创建 concise SOP document

### Pattern 4: Training 概述 摘要
User requests 摘要 of topic → 搜索 transcripts → 提取 and synthesize key points → Deliver as Markdown

### Pattern 5: Onboarding Checklist
User provides onboarding transcript → 提取 sequential tasks → 创建 checklist with checkboxes

### Pattern 6: Meeting Action Items
User provides meeting 注意 → 提取 decisions and tasks → 创建 action items 列表 with owners

## 故障排除

**Issue:** Slide appears empty in PDF
**Solution:** 检查 padding values. Reduce padding, 调整 spacing, ensure content fits within 720px height.

**Issue:** Logo not displaying
**Solution:** 验证 logo was copied 迁移到 project directory. Use absolute path in HTML.

**Issue:** Content seems incomplete
**Solution:** Flag with `[MISSING INFO]` rather than guessing. Ask user for clarification if critical.

**Issue:** Presentation exceeds height limit
**Solution:** Reduce font sizes, decrease spacing, condense content, or 拆分 into additional slides.

**Issue:** Too much conversational filler in 输出
**Solution:** Apply stricter filtering. 移除 phrases like "I think," "maybe," "um," "uh," "let's try."

**Issue:** Procedures lack clarity
**Solution:** 转换 迁移到 imperative voice. Use action verbs. 添加 "CRITICAL" prefix 迁移到 重要 steps.

## 资源

- **Master Knowledge Source Format:** `references/master-knowledge-source-format.md` - 完成 schema for structured learning modules
- **Presentation Guidelines:** `references/presentation-guidelines.md` - Detailed presentation 设计 and creation guidelines