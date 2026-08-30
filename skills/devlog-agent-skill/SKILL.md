---
name: devlog-agent-skill
description: 使用 dev-log-cli 跟踪 OpenClaw 代理的进度、任务和项目状态，维护结构化的开发者日志
tags:
- AI
---

# DevLog Skill 🦞

一个标准化的日志记录技能，供 OpenClaw 代理使用 `dev-log-cli` 跟踪进度、任务和项目状态。

## 描述
此技能使代理能够维护专业的开发者日志。它旨在以结构化的 SQLite 数据库捕获上下文、项目里程碑和任务状态。

## 要求
- `dev-log-cli`（通过 `pipx` 安装）

## 链接
- **GitHub**: [https://github.com/CrimsonDevil333333/dev-log-cli](https://github.com/CrimsonDevil333333/dev-log-cli)
- **PyPI**: [https://pypi.org/project/dev-log-cli/](https://pypi.org/project/dev-log-cli/)
- **ClawHub**: [https://clawhub.com/skills/devlog-skill](https://clawhub.com/skills/devlog-skill) (Pending Publication)

## 用法

### 📝 添加条目
代理应使用此功能来记录重大进展或阻碍。
```bash
devlog add "Finished implementing the auth module" --project "Project Alpha" --status "completed" --tags "auth,feature"
```

### 📋 列出日志
查看最近的活动以获取上下文。
```bash
devlog list --project "Project Alpha" --limit 5
```

### 📊 查看统计
检查项目健康状况和活动。
```bash
devlog stats --project "Project Alpha"
```

### 🔍 搜索
查找特定主题的历史上下文。
```bash
devlog search "infinite loop"
```

### 🛠️ 编辑/查看
详细检查或更正条目。
```bash
devlog view <id>
devlog edit <id>
```

## 内部设置
该技能包含一个 `setup.sh`，以确保 CLI 可用。
