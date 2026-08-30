---
name: commits
description: Git 提交规范专家 - Conventional Commits 格式、提交信息优化、代码审查辅助
---

# 提交信息指南

遵循 **Conventional Commits** 格式：

```
<type>(<scope>): <subject>

<body>

<footer>
```

## 提交类型

- `feat` - 新功能
- `fix` - Bug 修复
- `refactor` - 代码重构
- `test` - 添加或更新测试
- `docs` - 文档变更
- `chore` - 维护任务
- `style` - 代码样式变更（格式化）
- `perf` - 性能改进
- `ci` - CI/CD 变更

## 示例

```bash
feat(ui): add user search functionality

Implements real-time search with debouncing.

References: #RI-123

---

fix(api): resolve memory leak in connection pool

Properly cleanup subscriptions on unmount.

Fixes #456

---

test(ui): add tests for data serialization

refactor(api): extract common validation logic

docs: update API endpoint documentation

chore: upgrade React to version 18.2
```

## 最佳实践

### 好的提交

- 清晰、描述性的主题行
- 原子性变更（每次提交一个逻辑变更）
- 在正文中引用问题/工单
- 解释**为什么**，而不仅仅是**什么**
- **保持简洁** - 不要在正文中列出每个文件变更

```bash
feat(ui): add user profile editing

Allows users to update their profile information including
name, email, and avatar. Includes validation and error handling.

References: #RI-123
```

### 不好的提交

```bash
# 太模糊
fix stuff
WIP
update

# 太宽泛
add feature, fix bugs, refactor code, update tests
```

## 问题引用

- **JIRA (内部)**：`References: #RI-123` 或 `Fixes #RI-123`
- **GitHub (开源)**：`Fixes #456` 或 `Closes #456`
- 在提交信息中使用 `#` 进行自动链接
