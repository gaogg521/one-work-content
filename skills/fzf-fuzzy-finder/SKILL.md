---
name: fzf-fuzzy-finder
description: 命令行模糊查找器，用于交互式过滤和选择。与Shell、Vim及其他工具深度集成，支持快速文件搜索、历史记录检索和动态补全。
homepage: https://github.com/junegunn/fzf
metadata:
  clawdbot:
    emoji: 🔮
    requires:
      bins:
      - fzf
    install:
    - id: brew
      kind: brew
      formula: fzf
      bins:
      - fzf
      label: Install fzf (brew)
    - id: apt
      kind: apt
      package: fzf
      bins:
      - fzf
      label: Install fzf (apt)
---

# fzf - Fuzzy Finder

Interactive 命令-line fuzzy finder with powerful integration 能力.

## Basic 用法

### Simple filtering
```bash
# Pipe list to fzf
ls | fzf

# Select file
find . -type f | fzf

# Multi-select (Tab to select, Shift+Tab to deselect)
ls | fzf -m

# Preview files while selecting
ls | fzf --preview 'cat {}'

# With bat for syntax highlighting
ls | fzf --preview 'bat --color=always {}'
```

### Shell integration
```bash
# After installing, add to ~/.bashrc or ~/.zshrc:
# source /path/to/fzf/shell/completion.bash
# source /path/to/fzf/shell/key-bindings.bash

# Key bindings:
# Ctrl+R - Command history
# Ctrl+T - File search
# Alt+C  - Directory navigation

# Use in command line
vim **<TAB>      # File completion
cd **<TAB>       # Directory completion
kill -9 **<TAB>  # Process completion
```

## Common Patterns

### 文件 selection
```bash
# Open file in vim
vim $(fzf)

# Edit with preview
vim $(fzf --preview 'bat --color=always --line-range :500 {}')

# Select and copy
fzf | xargs -I {} cp {} /destination/

# Delete selected files
fzf -m | xargs rm
```

### 目录 navigation
```bash
# CD to selected directory
cd $(find . -type d | fzf)

# Alias for quick nav
alias cdf='cd $(find . -type d | fzf)'

# Or use Alt+C keybinding
```

### Git integration
```bash
# Checkout branch
git branch | fzf | xargs git checkout

# Show commit
git log --oneline | fzf | awk '{print $1}' | xargs git show

# Add files interactively
git status -s | fzf -m | awk '{print $2}' | xargs git add

# Fuzzy git log browser
alias gll='git log --oneline | fzf --preview "git show {1}"'
```

### 处理 management
```bash
# Kill process
ps aux | fzf | awk '{print $2}' | xargs kill

# Kill multiple processes
ps aux | fzf -m | awk '{print $2}' | xargs kill -9
```

## Advanced 功能特性

### Preview window
```bash
# Preview on the right
fzf --preview 'cat {}'

# Preview position and size
fzf --preview 'cat {}' --preview-window=right:50%

# Preview with bat
fzf --preview 'bat --color=always --style=numbers --line-range=:500 {}'

# Toggle preview with Ctrl+/
fzf --preview 'cat {}' --bind 'ctrl-/:toggle-preview'

# Preview directory contents
find . -type d | fzf --preview 'ls -la {}'
```

### Custom 键 bindings
```bash
# Execute action on selection
fzf --bind 'enter:execute(vim {})'

# Multiple bindings
fzf --bind 'ctrl-e:execute(vim {})' \
    --bind 'ctrl-o:execute(open {})'

# Reload on key press
fzf --bind 'ctrl-r:reload(find . -type f)'

# Accept non-matching input
fzf --print0 --bind 'enter:print-query'
```

### Filtering 选项
```bash
# Case-insensitive (default)
fzf -i

# Case-sensitive
fzf +i

# Exact match
fzf -e

# Inverse match (exclude)
fzf --query='!pattern'

# OR operator
fzf --query='py$ | js$'  # .py or .js files

# AND operator
fzf --query='test .py'  # Contains both 'test' and '.py'
```

## Integration 示例

### With ripgrep
```bash
# Search content and open in vim
rg --line-number . | fzf | awk -F: '{print "+"$2, $1}' | xargs vim

# Search and preview matches
rg --line-number . | fzf --delimiter : \
  --preview 'bat --color=always {1} --highlight-line {2}' \
  --preview-window +{2}-/2
```

### With fd
```bash
# Find and preview files
fd --type f | fzf --preview 'bat --color=always {}'

# Find files modified today
fd --changed-within 1d | fzf --preview 'bat {}'
```

### With docker
```bash
# Select and enter container
docker ps | fzf | awk '{print $1}' | xargs -I {} docker exec -it {} bash

# Remove selected images
docker images | fzf -m | awk '{print $3}' | xargs docker rmi

# View logs
docker ps | fzf | awk '{print $1}' | xargs docker logs -f
```

### With kubectl
```bash
# Select pod
kubectl get pods | fzf | awk '{print $1}' | xargs kubectl describe pod

# Get logs
kubectl get pods | fzf | awk '{print $1}' | xargs kubectl logs -f

# Delete pods
kubectl get pods | fzf -m | awk '{print $1}' | xargs kubectl delete pod
```

## Useful Aliases

添加 迁移到 your shell config:

```bash
# Fuzzy file search and open in vim
alias fv='vim $(fzf --preview "bat --color=always --style=numbers {}")'

# Fuzzy directory change
alias fcd='cd $(find . -type d | fzf)'

# Fuzzy git checkout
alias gco='git branch | fzf | xargs git checkout'

# Fuzzy process kill
alias fkill='ps aux | fzf | awk "{print \$2}" | xargs kill -9'

# Fuzzy history search (Ctrl+R is built-in)
alias fh='history | fzf | awk "{print \$2}" | xargs -I {} sh -c "{}"'

# Find and edit
alias fe='fd --type f | fzf --preview "bat --color=always --style=numbers {}" | xargs -r $EDITOR'
```

## 配置

### 环境 variables
```bash
# Default options
export FZF_DEFAULT_OPTS='
  --height 40%
  --layout=reverse
  --border
  --inline-info
  --preview "bat --style=numbers --color=always --line-range :500 {}"
'

# Use fd instead of find
export FZF_DEFAULT_COMMAND='fd --type f --hidden --follow --exclude .git'

# For Ctrl+T
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"

# For Alt+C
export FZF_ALT_C_COMMAND='fd --type d --hidden --follow --exclude .git'
```

### Color scheme
```bash
export FZF_DEFAULT_OPTS='
  --color=bg+:#313244,bg:#1e1e2e,spinner:#f5e0dc,hl:#f38ba8
  --color=fg:#cdd6f4,header:#f38ba8,info:#cba6f7,pointer:#f5e0dc
  --color=marker:#f5e0dc,fg+:#cdd6f4,prompt:#cba6f7,hl+:#f38ba8
'
```

## Advanced Workflows

### 项目 文件 browser
```bash
# Smart file browser with preview
fzf \
  --preview 'bat --color=always --style=numbers --line-range=:500 {}' \
  --preview-window='right:60%:wrap' \
  --bind 'enter:execute(vim {})' \
  --bind 'ctrl-y:execute-silent(echo {} | pbcopy)+abort' \
  --header 'Enter: edit | Ctrl+Y: copy path'
```

### Multi-用途 搜索
```bash
# Search in files and navigate to line
rg --line-number --no-heading . | \
  fzf --delimiter=: \
      --preview 'bat --color=always --style=numbers --highlight-line {2} {1}' \
      --preview-window='+{2}-/2' \
      --bind 'enter:execute(vim {1} +{2})'
```

### Docker container manager
```bash
#!/bin/bash
# docker-fzf.sh
container=$(docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Image}}" | fzf --header-lines=1 | awk '{print $1}')
if [ -n "$container" ]; then
    docker exec -it "$container" bash
fi
```

## 提示

- Use `--preview` for visual context
- Combine with __CODE___CODE__COD__CODE__CODE__CODE`fd`_ `rg`, `fd` for powerful workflows
- Press `?` in fzf 迁移到 see keybindings
- Use `Tab` for multi-select
- `Ctrl+/` 迁移到 toggle preview (if bound)
- `Ctrl+K`_CODE_1_`J`J` 迁移到 navigate
- 启动 query with `'` for exact match
- 启动 with `!` 迁移到 exclude
- Use `|` for OR, space for AND
- 集合 `FZF_DEFAULT_OPTS` for persistent config

## 性能

```bash
# For large file lists, use fd or rg
export FZF_DEFAULT_COMMAND='fd --type f'

# Limit depth for faster results
export FZF_DEFAULT_COMMAND='fd --type f --max-depth 5'

# Use parallel preview
fzf --preview 'bat {}' --preview-window 'hidden'
```

## Documentation

GitHub: https://github.com/junegunn/fzf
Wiki: https://github.com/junegunn/fzf/wiki
示例: https://github.com/junegunn/fzf/wiki/examples