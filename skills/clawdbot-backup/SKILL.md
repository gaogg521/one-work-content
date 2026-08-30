---
name: clawdbot-backup
description: 备份与恢复 ClawdBot 配置、技能、命令和设置，支持跨设备同步、git 版本控制、自动化备份及迁移到新机器。触发词：备份(backup)、恢复(restore)、同步(sync)、git、迁移(migration)。
homepage: https://github.com/clawdbot/backup-skill
metadata: None
clawdbot: None
emoji: 💾
requires: None
bins:
- git
- tar
- rsync
env: None
tags:
- API
- Git
---

# ClawdBot 备份技能

直接从 Clawdbot 备份、恢复和同步你的 ClawdBot 配置跨设备。

## 概述

此技能帮助你：
- 备份所有 ClawdBot 数据和设置
- 从备份恢复
- 在多台机器之间同步
- 版本控制你的配置
- 自动化备份例程
- 迁移到新设备

## ClawdBot 目录结构

### 关键位置

```
~/.claude/                    # 主 ClawdBot 目录
├── settings.json             # 全局设置
├── settings.local.json       # 本地覆盖（机器特定）
├── projects.json             # 项目配置
├── skills/                   # 你的自定义技能
│   ├── skill-name/
│   │   ├── SKILL.md
│   │   └── supporting-files/
│   └── another-skill/
├── commands/                 # 自定义斜杠命令（遗留）
│   └── command-name.md
├── contexts/                 # 保存的上下文
├── templates/                # 响应模板
└── mcp/                      # MCP 服务器配置
    └── servers.json

~/projects/                   # 你的项目（可选备份）
├── project-1/
│   └── .claude/              # 项目特定配置
│       ├── settings.json
│       └── skills/
└── project-2/
```

### 备份什么

```
必需（始终备份）：
✓ ~/.claude/skills/           # 自定义技能
✓ ~/.claude/commands/         # 自定义命令
✓ ~/.claude/settings.json     # 全局设置
✓ ~/.claude/mcp/              # MCP 配置

推荐（通常备份）：
✓ ~/.claude/contexts/         # 保存的上下文
✓ ~/.claude/templates/        # 模板
✓ Project .claude/ folders    # 项目配置

可选（视情况而定）：
○ ~/.claude/settings.local.json  # 机器特定
○ Cache directories              # 可以重建
○ Log files                      # 通常不需要
```

## 快速备份命令

### 完整备份

```bash
# 创建带时间戳的备份
BACKUP_DIR="$HOME/clawdbot-backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_NAME="clawdbot_backup_$TIMESTAMP"

mkdir -p "$BACKUP_DIR"

tar -czvf "$BACKUP_DIR/$BACKUP_NAME.tar.gz" \
  -C "$HOME" \
  .claude/skills \
  .claude/commands \
  .claude/settings.json \
  .claude/mcp \
  .claude/contexts \
  .claude/templates \
  2>/dev/null

echo "Backup created: $BACKUP_DIR/$BACKUP_NAME.tar.gz"
```

### 快速仅技能备份

```bash
# 仅备份技能
tar -czvf ~/clawdbot_skills_$(date +%Y%m%d).tar.gz \
  -C "$HOME" .claude/skills .claude/commands
```

### 从备份恢复

```bash
# 恢复完整备份
BACKUP_FILE="$HOME/clawdbot-backups/clawdbot_backup_20260129.tar.gz"

# 先预览内容
tar -tzvf "$BACKUP_FILE"

# 恢复（将覆盖现有内容）
tar -xzvf "$BACKUP_FILE" -C "$HOME"

echo "Restore complete!"
```

## 备份脚本

### 全功能备份脚本

```bash
#!/bin/bash
# clawdbot-backup.sh - 综合 ClawdBot 备份工具

set -e

# 配置
BACKUP_ROOT="${CLAWDBOT_BACKUP_DIR:-$HOME/clawdbot-backups}"
CLAUDE_DIR="$HOME/.claude"
MAX_BACKUPS=10  # 保留最近 N 个备份
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# 颜色
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

log_info() { echo -e "${GREEN}[INFO]${NC} $1"; }
log_warn() { echo -e "${YELLOW}[WARN]${NC} $1"; }
log_error() { echo -e "${RED}[ERROR]${NC} $1"; }

# 检查 ClawdBot 目录是否存在
check_claude_dir() {
    if [ ! -d "$CLAUDE_DIR" ]; then
        log_error "ClawdBot directory not found: $CLAUDE_DIR"
        exit 1
    fi
}

# 创建备份
create_backup() {
    local backup_type="${1:-full}"
    local backup_name="clawdbot_${backup_type}_${TIMESTAMP}"
    local backup_path="$BACKUP_ROOT/$backup_name.tar.gz"
    
    mkdir -p "$BACKUP_ROOT"
    
    log_info "Creating $backup_type backup..."
    
    case $backup_type in
        full)
            tar -czvf "$backup_path" \
                -C "$HOME" \
                .claude/skills \
                .claude/commands \
                .claude/settings.json \
                .claude/settings.local.json \
                .claude/projects.json \
                .claude/mcp \
                .claude/contexts \
                .claude/templates \
                2>/dev/null || true
            ;;
        skills)
            tar -czvf "$backup_path" \
                -C "$HOME" \
                .claude/skills \
                .claude/commands \
                2>/dev/null || true
            ;;
        settings)
            tar -czvf "$backup_path" \
                -C "$HOME" \
                .claude/settings.json \
                .claude/settings.local.json \
                .claude/mcp \
                2>/dev/null || true
            ;;
        *)
            log_error "Unknown backup type: $backup_type"
            exit 1
            ;;
    esac
    
    if [ -f "$backup_path" ]; then
        local size=$(du -h "$backup_path" | cut -f1)
        log_info "Backup created: $backup_path ($size)"
    else
        log_error "Backup failed!"
        exit 1
    fi
}

# 列出备份
list_backups() {
    log_info "Available backups in $BACKUP_ROOT:"
    echo ""
    
    if [ -d "$BACKUP_ROOT" ]; then
        ls -lh "$BACKUP_ROOT"/*.tar.gz 2>/dev/null | \
            awk '{print $9, $5, $6, $7, $8}' || \
            echo "No backups found."
    else
        echo "Backup directory doesn't exist."
    fi
}

# 恢复备份
restore_backup() {
    local backup_file="$1"
    
    if [ -z "$backup_file" ]; then
        log_error "Please specify backup file"
        list_backups
        exit 1
    fi
    
    if [ ! -f "$backup_file" ]; then
        # 尝试备份目录中的相对路径
        backup_file="$BACKUP_ROOT/$backup_file"
    fi
    
    if [ ! -f "$backup_file" ]; then
        log_error "Backup file not found: $backup_file"
        exit 1
    fi
    
    log_warn "This will overwrite existing configuration!"
    read -p "Continue? (y/N) " confirm
    
    if [ "$confirm" != "y" ] && [ "$confirm" != "Y" ]; then
        log_info "Restore cancelled."
        exit 0
    fi
    
    log_info "Restoring from: $backup_file"
    tar -xzvf "$backup_file" -C "$HOME"
    log_info "Restore complete!"
}

# 清理旧备份
cleanup_backups() {
    log_info "Cleaning old backups (keeping last $MAX_BACKUPS)..."
    
    cd "$BACKUP_ROOT" 2>/dev/null || return
    
    local count=$(ls -1 *.tar.gz 2>/dev/null | wc -l)
    
    if [ "$count" -gt "$MAX_BACKUPS" ]; then
        local to_delete=$((count - MAX_BACKUPS))
        ls -1t *.tar.gz | tail -n "$to_delete" | xargs rm -v
        log_info "Removed $to_delete old backup(s)"
    else
        log_info "No cleanup needed ($count backups)"
    fi
}

# 显示备份统计
show_stats() {
    log_info "ClawdBot Backup Statistics"
    echo ""
    
    echo "=== Directory Sizes ==="
    du -sh "$CLAUDE_DIR"/skills 2>/dev/null || echo "Skills: N/A"
    du -sh "$CLAUDE_DIR"/commands 2>/dev/null || echo "Commands: N/A"
    du -sh "$CLAUDE_DIR"/mcp 2>/dev/null || echo "MCP: N/A"
    du -sh "$CLAUDE_DIR" 2>/dev/null || echo "Total: N/A"
    
    echo ""
    echo "=== Skills Count ==="
    find "$CLAUDE_DIR/skills" -name "SKILL.md" 2>/dev/null | wc -l | xargs echo "Skills:"
    find "$CLAUDE_DIR/commands" -name "*.md" 2>/dev/null | wc -l | xargs echo "Commands:"
    
    echo ""
    echo "=== Backup Directory ==="
    if [ -d "$BACKUP_ROOT" ]; then
        du -sh "$BACKUP_ROOT"
        ls -1 "$BACKUP_ROOT"/*.tar.gz 2>/dev/null | wc -l | xargs echo "Backup files:"
    else
        echo "No backups yet"
    fi
}

# 用法
usage() {
    cat << EOF
ClawdBot Backup Tool

Usage: $(basename $0) <command> [options]

Commands:
    backup [type]   Create backup (types: full, skills, settings)
    restore <file>  Restore from backup file
    list            List available backups
    cleanup         Remove old backups (keep last $MAX_BACKUPS)
    stats           Show backup statistics
    help            Show this help

Examples:
    $(basename $0) backup              # Full backup
    $(basename $0) backup skills       # Skills only
    $(basename $0) restore latest.tar.gz
    $(basename $0) list
    $(basename $0) cleanup

Environment:
    CLAWDBOT_BACKUP_DIR    Backup directory (default: ~/clawdbot-backups)

EOF
}

# 主函数
main() {
    check_claude_dir
    
    case "${1:-help}" in
        backup)
            create_backup "${2:-full}"
            ;;
        restore)
            restore_backup "$2"
            ;;
        list)
            list_backups
            ;;
        cleanup)
            cleanup_backups
            ;;
        stats)
            show_stats
            ;;
        help|--help|-h)
            usage
            ;;
        *)
            log_error "Unknown command: $1"
            usage
            exit 1
            ;;
    esac
}

main "$@"
```

### 保存和使用

```bash
# 保存脚本
cat > ~/.local/bin/clawdbot-backup << 'SCRIPT'
# 在此处粘贴脚本内容
SCRIPT

chmod +x ~/.local/bin/clawdbot-backup

# 用法
clawdbot-backup backup          # 完整备份
clawdbot-backup backup skills   # 仅技能
clawdbot-backup list            # 列出备份
clawdbot-backup restore <file>  # 恢复
```

## Git 版本控制

### 初始化 Git 仓库

```bash
cd ~/.claude

# 初始化 git
git init

# 创建 .gitignore
cat > .gitignore << 'EOF'
# 机器特定设置
settings.local.json

# 缓存和临时文件
cache/
*.tmp
*.log

# 大文件
*.tar.gz
*.zip

# 敏感数据（如果有）
*.pem
*.key
credentials/
EOF

# 初始提交
git add .
git commit -m "Initial ClawdBot configuration backup"
```

### 推送到远程

```bash
# 添加远程 (GitHub, GitLab, 等)
git remote add origin git@github.com:username/clawdbot-config.git

# 推送
git push -u origin main
```

### 日常工作流

```bash
# 在对技能/设置进行更改后
cd ~/.claude
git add .
git commit -m "Updated skill: trading-bot"
git push
```

### 自动提交脚本

```bash
#!/bin/bash
# auto-commit-claude.sh - 自动提交更改

cd ~/.claude || exit 1

# 检查更改
if git diff --quiet && git diff --staged --quiet; then
    echo "No changes to commit"
    exit 0
fi

# 获取提交消息的更改文件
CHANGED=$(git status --short | head -5 | awk '{print $2}' | tr '\n' ', ')

git add .
git commit -m "Auto-backup: $CHANGED ($(date +%Y-%m-%d))"
git push 2>/dev/null || echo "Push failed (offline?)"
```

## 跨设备同步

### 方法 1: Git 同步

```bash
# 在新设备上
git clone git@github.com:username/clawdbot-config.git ~/.claude

# 拉取最新更改
cd ~/.claude && git pull

# 推送本地更改
cd ~/.claude && git add . && git commit -m "Update" && git push
```

### 方法 2: Rsync

```bash
# 同步到远程服务器
rsync -avz --delete \
    ~/.claude/ \
    user@server:~/clawdbot-backup/

# 从远程服务器同步
rsync -avz --delete \
    user@server:~/clawdbot-backup/ \
    ~/.claude/
```

### 方法 3: 云存储

```bash
# 备份到云文件夹 (Dropbox, Google Drive, 等)
CLOUD_DIR="$HOME/Dropbox/ClawdBot"

# 同步技能
rsync -avz ~/.claude/skills/ "$CLOUD_DIR/skills/"
rsync -avz ~/.claude/commands/ "$CLOUD_DIR/commands/"

# 复制设置
cp ~/.claude/settings.json "$CLOUD_DIR/"
```

### 同步脚本

```bash
#!/bin/bash
# sync-clawdbot.sh - 在设备之间同步 ClawdBot 配置

SYNC_DIR="${CLAWDBOT_SYNC_DIR:-$HOME/Dropbox/ClawdBot}"
CLAUDE_DIR="$HOME/.claude"

sync_to_cloud() {
    echo "Syncing to cloud..."
    mkdir -p "$SYNC_DIR"
    
    rsync -avz --delete "$CLAUDE_DIR/skills/" "$SYNC_DIR/skills/"
    rsync -avz --delete "$CLAUDE_DIR/commands/" "$SYNC_DIR/commands/"
    rsync -avz "$CLAUDE_DIR/mcp/" "$SYNC_DIR/mcp/" 2>/dev/null
    cp "$CLAUDE_DIR/settings.json" "$SYNC_DIR/" 2>/dev/null
    
    echo "Sync complete!"
}

sync_from_cloud() {
    echo "Syncing from cloud..."
    
    rsync -avz "$SYNC_DIR/skills/" "$CLAUDE_DIR/skills/"
    rsync -avz "$SYNC_DIR/commands/" "$CLAUDE_DIR/commands/"
    rsync -avz "$SYNC_DIR/mcp/" "$CLAUDE_DIR/mcp/" 2>/dev/null
    
    # 默认不要覆盖本地设置
    if [ ! -f "$CLAUDE_DIR/settings.json" ]; then
        cp "$SYNC_DIR/settings.json" "$CLAUDE_DIR/" 2>/dev/null
    fi
    
    echo "Sync complete!"
}

case "$1" in
    push) sync_to_cloud ;;
    pull) sync_from_cloud ;;
    *)
        echo "Usage: $0 {push|pull}"
        echo "  push - Upload local config to cloud"
        echo "  pull - Download cloud config to local"
        ;;
esac
```

## 自动化备份

### Cron Job (Linux/Mac)

```bash
# 编辑 crontab
crontab -e

# 添加每天凌晨 2 点的备份
0 2 * * * /home/user/.local/bin/clawdbot-backup backup full

# 添加每周日清理
0 3 * * 0 /home/user/.local/bin/clawdbot-backup cleanup

# 添加每 6 小时 git 自动提交
0 */6 * * * cd ~/.claude && git add . && git commit -m "Auto-backup $(date +\%Y-\%m-\%d)" && git push 2>/dev/null
```

### Systemd Timer (Linux)

```bash
# 创建 service: ~/.config/systemd/user/clawdbot-backup.service
cat > ~/.config/systemd/user/clawdbot-backup.service << 'EOF'
[Unit]
Description=ClawdBot Backup

[Service]
Type=oneshot
ExecStart=/home/user/.local/bin/clawdbot-backup backup full
EOF

# 创建 timer: ~/.config/systemd/user/clawdbot-backup.timer
cat > ~/.config/systemd/user/clawdbot-backup.timer << 'EOF'
[Unit]
Description=Daily ClawdBot Backup

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
EOF

# 启用
systemctl --user enable clawdbot-backup.timer
systemctl --user start clawdbot-backup.timer
```

### Launchd (macOS)

```bash
# 创建 plist: ~/Library/LaunchAgents/com.clawdbot.backup.plist
cat > ~/Library/LaunchAgents/com.clawdbot.backup.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.clawdbot.backup</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/username/.local/bin/clawdbot-backup</string>
        <string>backup</string>
        <string>full</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>2</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
</dict>
</plist>
EOF

# 加载
launchctl load ~/Library/LaunchAgents/com.clawdbot.backup.plist
```

## 迁移指南

### 迁移到新机器

```bash
# === 在旧机器上 ===

# 1. 创建完整备份
clawdbot-backup backup full

# 2. 复制备份文件到新机器
scp ~/clawdbot-backups/clawdbot_full_*.tar.gz newmachine:~/

# 或使用 git
cd ~/.claude
git add . && git commit -m "Pre-migration backup"
git push


# === 在新机器上 ===

# 方法 A: 从备份文件
tar -xzvf ~/clawdbot_full_*.tar.gz -C ~

# 方法 B: 从 git
git clone git@github.com:username/clawdbot-config.git ~/.claude

# 3. 验证
ls -la ~/.claude/skills/
```

### 导出单个技能

```bash
# 导出一个技能用于分享
SKILL_NAME="my-awesome-skill"
tar -czvf "${SKILL_NAME}.tar.gz" -C ~/.claude/skills "$SKILL_NAME"

# 导入技能
tar -xzvf "${SKILL_NAME}.tar.gz" -C ~/.claude/skills/
```

### 导出所有技能用于分享

```bash
# 创建可分享的技能包（无个人设置）
tar -czvf clawdbot-skills-share.tar.gz \
    -C ~/.claude \
    skills \
    --exclude='*.local*' \
    --exclude='*personal*'
```

## 备份验证

### 验证备份完整性

```bash
# 不提取测试备份
tar -tzvf backup.tar.gz > /dev/null && echo "Backup OK" || echo "Backup CORRUPT"

# 列出内容
tar -tzvf backup.tar.gz

# 验证特定文件存在
tar -tzvf backup.tar.gz | grep "skills/my-skill/SKILL.md"
```

### 比较备份与当前

```bash
# 提取到临时目录
TEMP_DIR=$(mktemp -d)
tar -xzf backup.tar.gz -C "$TEMP_DIR"

# 比较
diff -rq ~/.claude/skills "$TEMP_DIR/.claude/skills"

# 清理
rm -rf "$TEMP_DIR"
```

## 故障排除

### 常见问题

```bash
# 问题: 权限被拒绝
chmod -R u+rw ~/.claude

# 问题: 备份太大
# 排除缓存和日志
tar --exclude='cache' --exclude='*.log' -czvf backup.tar.gz ~/.claude

# 问题: 恢复覆盖了设置
# 保留 settings.local.json 用于机器特定配置
# 如果使用正确的备份，它不会被覆盖

# 问题: 同步后 Git 冲突
cd ~/.claude
git stash
git pull
git stash pop
# 如果需要，手动解决冲突
```

### 从损坏中恢复

```bash
# 如果 ~/.claude 已损坏

# 1. 移动损坏的目录
mv ~/.claude ~/.claude.corrupted

# 2. 从备份恢复
clawdbot-backup restore latest.tar.gz

# 3. 或从 git 恢复
git clone git@github.com:username/clawdbot-config.git ~/.claude

# 4. 比较并恢复任何缺失的内容
diff -rq ~/.claude ~/.claude.corrupted/
```

## 快速参考

### 基本命令

```bash
# 备份
tar -czvf ~/clawdbot-backup.tar.gz -C ~ .claude/skills .claude/commands .claude/settings.json

# 恢复
tar -xzvf ~/clawdbot-backup.tar.gz -C ~

# 列出备份内容
tar -tzvf ~/clawdbot-backup.tar.gz

# Git 备份
cd ~/.claude && git add . && git commit -m "Backup" && git push

# Git 恢复
cd ~/.claude && git pull
```

### 备份检查清单

```
重大更改之前:
□ 创建备份
□ 验证备份完整性
□ 记录你正在更改的内容

定期维护:
□ 每周完整备份
□ 每日 git 提交（如果使用）
□ 每月清理旧备份
□ 每季度测试恢复流程
```

## 资源

### 相关技能
```
- skill-creator - 创建新技能
- mcp-builder - 配置 MCP 服务器
- dotfiles - 通用 dotfile 管理
```

### 文档
```
- ClawdBot 文档: docs.clawdbot.com
- 技能指南: docs.clawdbot.com/skills
- MCP 设置: docs.clawdbot.com/mcp
```

---

**提示:** 在真正需要之前，始终测试你的备份恢复流程。无法恢复的备份毫无价值！
