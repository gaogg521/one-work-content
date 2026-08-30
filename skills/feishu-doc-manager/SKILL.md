---
name: feishu-doc-manager
description: 飞书文档管理器，将 Markdown 内容无缝发布到飞书文档并自动渲染格式。解决 Markdown 表格转换、权限管理和批量写入等核心痛点。
homepage: https://github.com/Shuai-DaiDai/feishu-doc-manager
tags:
- 飞书
---

# 📄 Feishu Doc Manager | 飞书文档管理器

> 将 Markdown 内容无缝发布到飞书文档，自动渲染格式。
> 
> Seamlessly publish Markdown content to Feishu Docs with automatic formatting.

## 🎯 解决的痛点 | Problems Solved

| 问题 | 解决方案 | Problem | Solution |
|---------|----------|------|----------|
| **Markdown 表格无法渲染** | 自动转换为格式化列表 | Markdown tables not rendering | Auto-convert tables to formatted lists |
| **权限管理复杂** | 一键协作者管理 | Permission management complexity | One-click collaborator management |
| **长内容 400 错误** | 自动分段写入 | 400 errors on long content | Auto-split long documents |
| **格式不一致** | `write`/`append` 自动渲染 | Inconsistent formatting | `write`/`append` auto-render Markdown |

## ✨ 核心功能 | Key Features

### 1. 📝 智能 Markdown 发布 | Smart Markdown Publishing
- **自动渲染**: `write`/`append` 动作自动渲染 Markdown
- **表格处理**: 表格自动转换为格式化列表
- **语法支持**: 标题、列表、粗体、斜体、代码、引用

### 2. 🔐 权限管理 | Permission Management
- 添加/移除协作者
- 更新权限级别（查看/编辑/full_access）
- 列出当前权限

### 3. 📄 文档操作 | Document Operations
- 创建新文档
- 使用 Markdown 写入完整内容
- 追加到现有文档
- 更新/删除特定块

## 🚀 快速开始 | Quick Start

```bash
cd ~/.openclaw/workspace/skills
git clone https://github.com/Shuai-DaiDai/feishu-doc-manager.git
```

## 📋 支持的 Markdown | Supported Markdown

| Markdown | 飞书结果 |
|----------|---------------|
| `# Title` | Heading 1 |
| `- Item` | Bullet list |
| `**bold**` | Bold |
| `> quote` | Blockquote |

## 🔧 重要区分 | Important Distinctions

**`write`/`append` vs `update_block`**:

| 特性 | `write`/`append` | `update_block` |
|---------|------------------|----------------|
| Markdown 渲染 | ✅ 是 | ❌ 否（纯文本） |

## 📦 必需权限 | Required Permissions

- `docx:document`
- `docx:document:write_only`
- `docs:permission.member`

## 📝 许可证 | License

MIT
