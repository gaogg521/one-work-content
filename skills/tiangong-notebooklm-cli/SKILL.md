---
name: tiangong-notebooklm-cli
description: NotebookLM CLI封装器，通过`node {baseDir}/scripts/notebooklm.mjs`调用。用于认证、笔记本管理、聊天、来源管理、笔记、分享、研究以及产物生成和下载。
---

# NotebookLM CLI Wrapper

## Required 参数
- __CODE_`notebooklm`klm`klm` available on PATH.
- NotebookLM CLI authenticated (运行 `login` if needed).

## 快速开始
- Wrapper script: `scripts/notebooklm.mjs` (invokes `notebooklm` C`notebooklm``notebooklm`
- 运行 from the skill directory or use an absolute `{baseDir}` path.

```bash
node {baseDir}/scripts/notebooklm.mjs status
node {baseDir}/scripts/notebooklm.mjs login
node {baseDir}/scripts/notebooklm.mjs list
node {baseDir}/scripts/notebooklm.mjs use <notebook_id>
node {baseDir}/scripts/notebooklm.mjs ask "Summarize the key takeaways" --notebook <notebook_id>
```

## Request & 输出
- 命令 form: `node {baseDir}/scripts/notebooklm.mjs <command> [args...]`.
- Prefer `--json` for machine-readable 输出.
- For long-running tasks, use `--exec-timeout <seconds>`; `--timeout` is r`--timeout` wait/`--timeout`

## 参考
- `references/cli-commands.md`

## Assets
- None.