---
name: feishu-docx-powerwrite
description: 通过 OpenClaw 高质量撰写飞书/Lark Docx。当你想将 Markdown 转换为格式良好的飞书 Docx（标题、列表、嵌套、代码块）时使用 feishu_docx_write_markdown；包含安全工作流、模板和故障排除。触发条件：飞书 doc/docx 链接、“写入飞书文档”、“生成飞书文档”、“追加/替换 docx”、“将 markdown 转换为飞书文档”，或当用户希望获得一致良好的文档格式时。
tags:
- 飞书
---

# Feishu Docx PowerWrite

This skill focuses on **reliably writing great-looking Feishu Docx** using OpenClaw's Feishu OpenAPI tools.

Key idea: prefer **`feishu_docx_write_markdown`** (Markdown → Docx blocks) for structure-preserving output.

## Quick workflow

1) Get `document_id` (Docx token)
- From a Docx URL: `https://.../docx/<document_id>`

2) Decide write mode
- **Append**: add new content below existing content (most common)
- **Replace**: overwrite the entire document (use carefully)

3) Write markdown
- Use headings + lists + short paragraphs
- Avoid huge single paragraphs (harder to read)

## Recommended defaults

### Append mode (safe)
Use when adding sections, meeting notes, daily logs.

- `mode: append`
- Keep each append chunk <= ~300-600 lines if possible

### Replace mode (destructive)
Use when generating the full doc from scratch.

- `mode: replace`
- MUST set `confirm: true`

## Markdown patterns that render well

### Title + summary
```md
# <Title>

**Summary**
- Point 1
- Point 2

---
```

### Sections
```md
## Section

Short paragraph.

- Bullet
- Bullet

### Subsection

1) Step
2) Step
```

### Code
Use fenced blocks.

```md
```bash
openclaw skills check
```
```

## Templates & references

- Templates: `references/templates.md`
- Troubleshooting: `references/troubleshooting.md`

## Safety / privacy

- Never hardcode tokens, chat_id, open_id, or document links inside this skill.
- Always use the user's own Feishu app credentials and scopes.
