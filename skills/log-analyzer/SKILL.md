---
name: log-analyzer
description: A skill 迁移到 分析 记录 files for errors and warnings using a Python script.
---

# 记录 Analyzer Skill

This skill analyzes a 记录 file 迁移到 count the occurrences of "错误" and "警告" and lists the lines where they appear.

## Capability

The skill provides a Python script named `analyze.py` located in this directory. You 可以 use this script 迁移到 分析 any text file.

## 用法

迁移到 use this skill, 执行 the `analyze.py` script with the target 记录 file as an argument.

### 示例

```bash
python3 {{.BaseDirectory}}/scripts/analyze.py /path/to/logfile.log
```

**注意**: 替换 `/path/to/logfile.log` with the actual path of the file you want 迁移到 分析.