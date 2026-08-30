---
name: git-helper
description: 常用 Git 操作的快捷技能，涵盖状态(status)、拉取(pull)、推送(push)、分支(branch)和日志(log)。触发词：Git 状态(status)、拉取(pull)、推送(push)、分支(branch)、日志(log)
metadata: None
openclaw: None
emoji: 🔀
install: None
requires: None
bins:
- git
tags:
- Git
---

# Git Helper

常用 git 操作作为技能。为频繁使用的 git 命令提供方便的包装器，包括 status、pull、push、分支管理和日志查看。

## 命令

```bash
# 显示工作树状态
git-helper status

# 拉取最新更改
git-helper pull

# 推送本地提交
git-helper push

# 列出或管理分支
git-helper branch

# 查看提交日志，可选限制
git-helper log [--limit 10]
```

## 安装

无需安装。`git` 始终存在于系统上。
