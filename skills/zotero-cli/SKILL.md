---
name: zotero-cli
version: 1.0.0
description: Zotero 的命令行界面 - 搜索你的 Zotero 库、添加/编辑笔记、阅读附件，并从终端管理参考文献。
homepage: https://github.com/jbaiter/zotero-cli
metadata:
  openclaw:
    emoji: 📚
    requires:
      bins:
      - python3
      anyBins:
      - zotcli
      - zotero-cli
    install:
    - id: pip
      kind: pip
      package: zotero-cli
      label: Install zotero-cli Python package (pip)
    - id: pipx
      kind: pipx
      package: zotero-cli
      label: Install zotero-cli Python package (pipx - recommended for systems with
        PEP 668 compliance)
      platforms:
      - linux-debian
      - linux-ubuntu
      - linux-arch
      - linux-fedora
      - linux-rhel
tags:
- 知识共享
- CLI
- 办公
- 管理
---

# Zotero CLI

Zotero 参考文献管理器的命令行界面，通过 Zotero API 提供对 Zotero 库的终端访问。

## 快速开始

```bash
# 安装（PEP 668 系统）
sudo apt install pipx && pipx ensurepath && pipx install zotero-cli

# 配置
zotcli configure

# 开始使用
zotcli query "machine learning"
zotcli add-note "\"deep learning\""
zotcli read "\"attention mechanism\""
```

📖 **详细指南：** [QUICKSTART.md](QUICKSTART.md)

## 安装

### pipx（推荐用于 PEP 668 系统）
```bash
pipx install zotero-cli
```

### pip（通用）
```bash
pip install --user zotero-cli
export PATH="$HOME/.local/bin:$PATH"
```

### 虚拟环境
```bash
python3 -m venv ~/.venvs/zotero-cli
source ~/.venvs/zotero-cli/bin/activate
pip install zotero-cli
```

📖 **完整安装指南：** [INSTALL.md](INSTALL.md)

## 核心命令

| 命令 | 描述 |
|---------|-------------|
| `zotcli query "topic"` | 搜索库 |
| `zotcli add-note "paper"` | 添加笔记 |
| `zotcli edit-note "paper"` | 编辑笔记 |
| `zotcli read "paper"` | 阅读第一个 PDF 附件 |
| `zotcli configure` | 配置 API 凭据 |

## 配置

```bash
# 设置默认编辑器
export VISUAL=nano  # 或 vim、emacs、code

# 运行配置
zotcli configure

# 验证设置
./scripts/setup_and_check.sh
```

## 辅助脚本

| 脚本 | 用途 |
|--------|---------|
| `setup_and_check.sh` | 自动设置和验证 |
| `backup_restore.sh` | 备份和恢复配置 |
| `update_check.sh` | 检查更新 |
| `quick_search.py` | 格式化搜索输出 |
| `export_citations.py` | 导出引用（BibTeX、RIS） |
| `batch_process.sh` | 处理多个查询 |

**使用示例：**

```bash
# 快速搜索
python scripts/quick_search.py "topic" --format table

# 导出引用
python scripts/export_citations.py "topic" --format bib > refs.bib

# 备份
./scripts/backup_restore.sh backup

# 更新检查
./scripts/update_check.sh check
```

📖 **脚本文档：** [scripts/README.md](scripts/README.md)

## 查询语法

```bash
"neural AND networks"        # 布尔 AND
"(deep OR machine) AND learning"  # OR + 分组
"learning NOT neural"        # NOT
"\"deep learning\""          # 短语搜索
"transform*"                 # 前缀搜索
```

## 工作流

### 文献综述
```bash
zotcli query "topic"
zotcli add-note "paper"
python scripts/export_citations.py "topic" --format bib > refs.bib
```

### 日常研究
```bash
python scripts/quick_search.py "\"recent\"" --format table
zotcli add-note "\"interesting paper\""
./scripts/backup_restore.sh backup
```

📖 **更多示例：** [EXAMPLES.md](EXAMPLES.md)

## 文档

| 文档 | 描述 |
|----------|-------------|
| [QUICKSTART.md](QUICKSTART.md) | 5 分钟快速入门指南 |
| [INSTALL.md](INSTALL.md) | 综合安装指南 |
| [EXAMPLES.md](EXAMPLES.md) | 实用使用示例 |
| [scripts/README.md](scripts/README.md) | 辅助脚本指南 |

## 故障排除

**命令未找到：**
```bash
export PATH="$HOME/.local/bin:$PATH"
pipx ensurepath
```

**权限被拒绝（PEP 668 系统）：**
```bash
pipx install zotero-cli
```

**配置错误：**
```bash
zotcli configure
```

📖 **详细故障排除：** [INSTALL.md](INSTALL.md)

## 快速参考

```bash
# 基本命令
zotcli query "topic"              # 搜索
zotcli add-note "paper"           # 添加笔记
zotcli edit-note "paper"          # 编辑笔记
zotcli read "paper"               # 阅读 PDF

# 辅助脚本
./scripts/setup_and_check.sh      # 设置
./scripts/backup_restore.sh backup # 备份
./scripts/update_check.sh check   # 更新
./scripts/batch_process.sh queries.txt --output results.txt  # 批量
```

---

**完整文档：**
- [QUICKSTART.md](QUICKSTART.md) - 开始使用
- [INSTALL.md](INSTALL.md) - 安装详情
- [EXAMPLES.md](EXAMPLES.md) - 使用示例
- [SKILL_SUMMARY.md](SKILL_SUMMARY.md) - 完整概述
