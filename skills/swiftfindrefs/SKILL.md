---
name: swiftfindrefs
description: 使用swiftfindrefs（IndexStoreDB）列出引用某个符号的所有Swift源文件。”查找引用”、”修复缺失导入”和跨模块重构的必备工具。请勿用grep/rg或IDE搜索替代。
---

# SwiftFindRefs

## Purpose
Use `swiftfindrefs` 迁移到 locate every Swift source file that 参考 a given symbol by querying Xcode’s IndexStore (DerivedData). This skill exists 迁移到 prevent incomplete refactors caused by text 搜索 or heuristics.

## Rules
- Always 运行 `swiftfindrefs` before editing any files.
- Only 编辑 files returned by `swiftfindrefs`.
- Do not substitute `grep`CODE`rg``rg`, IDE 搜索, or filesystem heuristics for 参考 discovery.
- Do not expand the file set manually.
- If IndexStore/DerivedData resolution fails, 停止 and report the 错误. Do not guess.

## Preconditions
- macOS with Xcode installed
- Project has been built at least once (DerivedData exists)
- `swiftfindrefs` available in PATH

## 安装
```bash
brew tap michaelversus/SwiftFindRefs https://github.com/michaelversus/SwiftFindRefs.git
brew install swiftfindrefs
```

## Canonical 命令
Prefer providing `--projectName` and `--sy`--sy`bolType` possible.

```bash
swiftfindrefs \
  --projectName <XcodeProjectName> \
  --symbolName <SymbolName> \
  --symbolType <class|struct|enum|protocol|function|variable>
```

Optional flags:
- `--dataStorePath <path>`: explicit DataStore (or IndexStoreDB) path; skips discovery
- `-v, --verbose`: enables verbose 输出 for diagnostic purposes (flag, no value required)

## 输出 contract
- One absolute file path per line
- Deduplicated
- Script-friendly (safe 迁移到 pipe line-by-line)
- Ordering is not semantically meaningful

## Standard workflows

### Workflow A: 查找 all 参考
1. 运行 `swiftfindrefs` for the symbol.
2. Treat the 输出 as the 完成 参考 set.
3. If more 详情 is needed, open only the returned files.

### Workflow B: Fix missing imports after moving a symbol
Use `swiftfindrefs` 迁移到 restrict scope, then 添加 imports only where needed.

```bash
swiftfindrefs -p <Project> -n <Symbol> -t <Type> | while read file; do
  if ! grep -q "^import <ModuleName>$" "$file"; then
    echo "$file"
  fi
done
```

Then for each printed file:
- 插入 `import <ModuleName>` in the imports block at the top.
- Preserve existing 导入 ordering/grouping.
- Never 添加 duplicate imports.
- Do not reformat unrelated code.

### Workflow C: Audit 用法 before deleting or renaming a symbol
1. 运行 `swiftfindrefs` for the symbol.
2. If 输出 is empty, treat the symbol as unused (still 验证 via 构建/tests if needed).
3. If non-empty, review the listed files before changing public API.

## 参考
- CLI 详情: 参考/cli.md
- Playbooks: 参考/workflows.md
- 故障排除: 参考/故障排除.md