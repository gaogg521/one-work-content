---
name: git-essentials
description: 版本控制(version control)、分支(branching)和协作(collaboration)的基本 Git 命令和工作流。触发词：Git 基础(basics)、版本控制(version control)、分支管理(branch management)、提交(commit)
homepage: https://git-scm.com/
metadata: None
clawdbot: None
emoji: 🌳
requires: None
bins:
- git
tags:
- Git
---

# Git 基础

版本控制和协作的基本 Git 命令。

## 初始设置

```bash
# 配置用户信息
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# 初始化仓库
git init

# 克隆仓库
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git custom-name
```

## 基本工作流

### 暂存和提交
```bash
# 查看状态
git status

# 添加文件到暂存区
git add file.txt
git add .
git add -A  # 所有更改，包括删除

# 提交更改
git commit -m "Commit message"

# 一步完成添加和提交
git commit -am "Message"

# 修改最后一次提交
git commit --amend -m "New message"
git commit --amend --no-edit  # 保留提交信息
```

### 查看更改
```bash
# 显示未暂存的更改
git diff

# 显示已暂存的更改
git diff --staged

# 显示特定文件的更改
git diff file.txt

# 显示两次提交之间的更改
git diff commit1 commit2
```

## 分支与合并

### 分支管理
```bash
# 列出分支
git branch
git branch -a  # 包含远程分支

# 创建分支
git branch feature-name

# 切换分支
git checkout feature-name
git switch feature-name  # 现代替代方案

# 创建并切换
git checkout -b feature-name
git switch -c feature-name

# 删除分支
git branch -d branch-name
git branch -D branch-name  # 强制删除

# 重命名分支
git branch -m old-name new-name
```

### 合并
```bash
# 将分支合并到当前分支
git merge feature-name

# 无快进合并
git merge --no-ff feature-name

# 中止合并
git merge --abort

# 显示合并冲突
git diff --name-only --diff-filter=U
```

## 远程操作

### 管理远程仓库
```bash
# 列出远程仓库
git remote -v

# 添加远程仓库
git remote add origin https://github.com/user/repo.git

# 更改远程仓库 URL
git remote set-url origin https://github.com/user/new-repo.git

# 删除远程仓库
git remote remove origin
```

### 与远程同步
```bash
# 从远程获取
git fetch origin

# 拉取更改（fetch + merge）
git pull

# 使用 rebase 拉取
git pull --rebase

# 推送更改
git push

# 推送新分支
git push -u origin branch-name

# 强制推送（谨慎使用！）
git push --force-with-lease
```

## 历史与日志

### 查看历史
```bash
# 显示提交历史
git log

# 每个提交一行
git log --oneline

# 带图形显示
git log --graph --oneline --all

# 最近 N 个提交
git log -5

# 按作者筛选
git log --author="Name"

# 按日期范围筛选
git log --since="2 weeks ago"
git log --until="2024-01-01"

# 文件历史
git log -- file.txt
```

### 搜索历史
```bash
# 搜索提交信息
git log --grep="bug fix"

# 搜索代码更改
git log -S "function_name"

# 显示每行代码的最后修改者
git blame file.txt

# 查找引入 bug 的提交
git bisect start
git bisect bad
git bisect good commit-hash
```

## 撤销更改

### 工作目录
```bash
# 丢弃文件中的更改
git restore file.txt
git checkout -- file.txt  # 旧方式

# 丢弃所有更改
git restore .
```

### 暂存区
```bash
# 取消暂存文件
git restore --staged file.txt
git reset HEAD file.txt  # 旧方式

# 取消所有暂存
git reset
```

### 提交
```bash
# 撤销最后一次提交（保留更改）
git reset --soft HEAD~1

# 撤销最后一次提交（丢弃更改）
git reset --hard HEAD~1

# 回退提交（创建新提交）
git revert commit-hash

# 重置到特定提交
git reset --hard commit-hash
```

## 暂存

```bash
# 暂存更改
git stash

# 带消息暂存
git stash save "Work in progress"

# 列出暂存
git stash list

# 应用最新暂存
git stash apply

# 应用并删除暂存
git stash pop

# 应用特定暂存
git stash apply stash@{2}

# 删除暂存
git stash drop stash@{0}

# 清除所有暂存
git stash clear
```

## 变基

```bash
# 对当前分支变基
git rebase main

# 交互式变基（最近 3 个提交）
git rebase -i HEAD~3

# 解决冲突后继续
git rebase --continue

# 跳过当前提交
git rebase --skip

# 中止变基
git rebase --abort
```

## 标签

```bash
# 列出标签
git tag

# 创建轻量标签
git tag v1.0.0

# 创建附注标签
git tag -a v1.0.0 -m "Version 1.0.0"

# 对特定提交打标签
git tag v1.0.0 commit-hash

# 推送标签
git push origin v1.0.0

# 推送所有标签
git push --tags

# 删除标签
git tag -d v1.0.0
git push origin --delete v1.0.0
```

## 高级操作

### 挑选提交
```bash
# 应用特定提交
git cherry-pick commit-hash

# 挑选但不提交
git cherry-pick -n commit-hash
```

### 子模块
```bash
# 添加子模块
git submodule add https://github.com/user/repo.git path/

# 初始化子模块
git submodule init

# 更新子模块
git submodule update

# 递归克隆
git clone --recursive https://github.com/user/repo.git
```

### 清理
```bash
# 预览将要删除的文件
git clean -n

# 删除未跟踪的文件
git clean -f

# 删除未跟踪的文件和目录
git clean -fd

# 包含被忽略的文件
git clean -fdx
```

## 常见工作流

**功能分支工作流：**
```bash
git checkout -b feature/new-feature
# 进行修改
git add .
git commit -m "Add new feature"
git push -u origin feature/new-feature
# 创建 PR，合并后：
git checkout main
git pull
git branch -d feature/new-feature
```

**热修复工作流：**
```bash
git checkout main
git pull
git checkout -b hotfix/critical-bug
# 修复 bug
git commit -am "Fix critical bug"
git push -u origin hotfix/critical-bug
# 合并后：
git checkout main && git pull
```

**同步 Fork：**
```bash
git remote add upstream https://github.com/original/repo.git
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

## 实用别名

添加到 `~/.gitconfig`：
```ini
[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    unstage = reset HEAD --
    last = log -1 HEAD
    visual = log --graph --oneline --all
    amend = commit --amend --no-edit
```

## 提示

- 频繁提交，稍后完善（交互式变基）
- 编写有意义的提交信息
- 使用 `.gitignore` 排除不需要的文件
- 永远不要对共享分支强制推送
- 开始工作前先拉取
- 使用功能分支，不要直接在 main 上开发
- 合并前先对功能分支变基
- 使用 `--force-with-lease` 代替 `--force`

## 常见问题

**撤销意外提交：**
```bash
git reset --soft HEAD~1
```

**恢复已删除的分支：**
```bash
git reflog
git checkout -b branch-name <commit-hash>
```

**修正错误的提交信息：**
```bash
git commit --amend -m "Correct message"
```

**解决合并冲突：**
```bash
# 编辑文件以解决冲突
git add resolved-files
git commit  # 或 git merge --continue
```

## 文档

官方文档: https://git-scm.com/doc
Pro Git 书籍: https://git-scm.com/book
可视化 Git 指南: https://marklodato.github.io/visual-git-guide/