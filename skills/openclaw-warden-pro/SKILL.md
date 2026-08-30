---
name: openclaw-warden-pro
description: 完整的工作空间安全套件：检测未经授权的修改，扫描提示注入模式，并自动响应对策 —— 快照恢复、技能隔离、git 回滚和自动保护扫描。代理工作空间的完整安装后安全层。
user-invocable: True
metadata: None
openclaw: None
emoji: 🛡️
os:
- darwin
- linux
- win32
requires: None
bins:
- python3
tags:
- Git
---

# OpenClaw Warden Pro

[openclaw-warden](https://github.com/AtlasPA/openclaw-warden)（免费版）中的所有内容加上自动对策。

**免费版检测威胁。专业版响应它们。**

## 检测命令（也在免费版中）

```bash
python3 {baseDir}/scripts/integrity.py baseline --workspace /path/to/workspace
python3 {baseDir}/scripts/integrity.py verify --workspace /path/to/workspace
python3 {baseDir}/scripts/integrity.py scan --workspace /path/to/workspace
python3 {baseDir}/scripts/integrity.py full --workspace /path/to/workspace
python3 {baseDir}/scripts/integrity.py status --workspace /path/to/workspace
python3 {baseDir}/scripts/integrity.py accept SOUL.md --workspace /path/to/workspace
```

## 专业版对策

### 从快照恢复

将被篡改的文件恢复到其基线快照。建立基线时，关键、配置和技能文件会自动快照。

```bash
python3 {baseDir}/scripts/integrity.py restore SOUL.md --workspace /path/to/workspace
```

### Git 回滚

将文件恢复到其上次 git 提交的状态。

```bash
python3 {baseDir}/scripts/integrity.py rollback SOUL.md --workspace /path/to/workspace
```

### 隔离技能

通过重命名其目录来禁用可疑技能。代理不会加载隔离的技能。

```bash
python3 {baseDir}/scripts/integrity.py quarantine bad-skill --workspace /path/to/workspace
```

### 取消隔离技能

调查后恢复隔离的技能。

```bash
python3 {baseDir}/scripts/integrity.py unquarantine bad-skill --workspace /path/to/workspace
```

### 保护（自动响应）

一次通过完整扫描 + 自动对策：恢复被篡改的关键文件，隔离恶意技能，标记剩余问题。这是会话启动的推荐命令。

```bash
python3 {baseDir}/scripts/integrity.py protect --workspace /path/to/workspace
```

## 推荐集成

### 会话启动钩子 (Claude Code)

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 scripts/integrity.py protect",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Heartbeat (OpenClaw)

添加到 HEARTBEAT.md 以进行定期保护：
```
- Run workspace integrity protection (python3 {skill:openclaw-warden-pro}/scripts/integrity.py protect)
```

### 安装新技能后

运行 `protect` 以自动隔离修改了工作空间文件的技能。

## 监控内容

| Category | Files | Alert Level |
|----------|-------|-------------|
| **Critical** | SOUL.md, AGENTS.md, IDENTITY.md, USER.md, TOOLS.md, HEARTBEAT.md | WARNING |
| **Memory** | memory/*.md, MEMORY.md | INFO |
| **Config** | *.json in workspace root | WARNING |
| **Skills** | skills/*/SKILL.md | WARNING |

## 对策摘要

| Command | Action |
|---------|--------|
| `protect` | 完整扫描 + 自动恢复 + 自动隔离 + 标记 |
| `restore <file>` | 从基线快照恢复 |
| `rollback <file>` | 从 git 历史记录恢复 |
| `quarantine <skill>` | 通过重命名目录禁用技能 |
| `unquarantine <skill>` | 重新启用隔离的技能 |

## 无外部依赖

仅 Python 标准库。无需 pip install。无网络调用。所有内容都在本地运行。

## 跨平台

适用于 OpenClaw、Claude Code、Cursor 和任何使用 Agent Skills 规范的工具。
