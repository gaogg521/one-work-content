---
name: oracle
description: 使用 Oracle CLI 的最佳实践（提示 + 文件打包、引擎、会话和文件附件模式）。
homepage: https://askoracle.dev
metadata:
  openclaw:
    emoji: 🧿
    requires:
      bins:
      - oracle
    install:
    - id: node
      kind: node
      package: '@steipete/oracle'
      bins:
      - oracle
      label: 安装 oracle (node)
---

# oracle — 最佳使用

Oracle 将你的 prompt + 选中的文件打包成一个 "one-shot" 请求，以便另一个模型可以用真实的 repo context 回答（API 或 browser automation）。将输出视为建议性的：对照 code + tests 进行验证。

## 主要使用场景 (browser, GPT‑5.2 Pro)

此处的默认工作流：在 ChatGPT 中使用 `--engine browser` 配合 GPT‑5.2 Pro。这是常见的 "long think" 路径：~10 分钟到 ~1 小时是正常的；预期会有一个 stored session 可以重新 attach。

推荐的默认值：

- Engine: browser (`--engine browser`)
- Model: GPT‑5.2 Pro (`--model gpt-5.2-pro` 或 `--model "5.2 Pro"`)

## 黄金路径

1. 选择一个精简的文件集合（包含 truth 的最少文件）。
2. Preview payload + token spend (`--dry-run` + `--files-report`)。
3. 对于通常的 GPT‑5.2 Pro 工作流使用 browser mode；只有在你明确想要时才使用 API。
4. 如果 run detaches/timeouts：reattach 到 stored session（不要 re-run）。

## 命令 (推荐)

- 帮助：
  - `oracle --help`
  - 如果 binary 未安装：`npx -y @steipete/oracle --help`（此处避免使用 `pnpx`；sqlite bindings）。

- Preview (no tokens)：
  - `oracle --dry-run summary -p "<task>" --file "src/**" --file "!**/*.test.*"`
  - `oracle --dry-run full -p "<task>" --file "src/**"`

- Token 检查：
  - `oracle --dry-run summary --files-report -p "<task>" --file "src/**"`

- Browser run (main path; long-running 是正常的)：
  - `oracle --engine browser --model gpt-5.2-pro -p "<task>" --file "src/**"`

- 手动粘贴备选：
  - `oracle --render --copy -p "<task>" --file "src/**"`
  - 注意：`--copy` 是 `--copy-markdown` 的 hidden alias。

## 附加文件 (`--file`)

`--file` 接受 files、directories 和 globs。你可以多次传递它；entries 可以用逗号分隔。

- 包含：
  - `--file "src/**"`
  - `--file src/index.ts`
  - `--file docs --file README.md`

- 排除：
  - `--file "src/**" --file "!src/**/*.test.ts" --file "!**/*.snap"`

- 默认值 (implementation behavior)：
  - Default-ignored dirs: `node_modules`, `dist`, `coverage`, `.git`, `.turbo`, `.next`, `build`, `tmp`（除非作为 literal dirs/files 显式传递，否则跳过）。
  - 在扩展 globs 时遵循 `.gitignore`。
  - 不跟随 symlinks。
  - Dotfiles 被过滤，除非通过 pattern 选择加入（例如 `--file ".github/**"`）。
  - Files > 1 MB 被拒绝。

## Engines (API vs browser)

- 自动选择：当设置了 `OPENAI_API_KEY` 时使用 `api`；否则使用 `browser`。
- Browser 仅支持 GPT + Gemini；对于 Claude/Grok/Codex 或 multi-model runs 使用 `--engine api`。
- Browser attachments：
  - `--browser-attachments auto|never|always` (auto 内联粘贴最多 ~60k 字符然后上传)。
- Remote browser host：
  - Host: `oracle serve --host 0.0.0.0 --port 9473 --token <secret>`
  - Client: `oracle --engine browser --remote-host <host:port> --remote-token <secret> -p "<task>" --file "src/**"`

## Sessions + slugs

- 存储在 `~/.oracle/sessions` 下（用 `ORACLE_HOME_DIR` 覆盖）。
- Runs 可能会 detach 或花费很长时间（browser + GPT‑5.2 Pro 经常如此）。如果 CLI times out：不要 re-run；reattach。
  - 列表：`oracle status --hours 72`
  - Attach：`oracle session <id> --render`
- 使用 `--slug "<3-5 words>"` 保持 session IDs 可读。
- Duplicate prompt guard 存在；只有在你真正想要 fresh run 时才使用 `--force`。

## Prompt template (high signal)

Oracle 以 **zero** project knowledge 开始。假设 model 无法推断你的 stack、build tooling、conventions 或 "obvious" paths。包含：

- Project briefing (stack + build/test commands + platform constraints)。
- "Where things live" (key directories、entrypoints、config files、boundaries)。
- Exact question + what you tried + error text (verbatim)。
- Constraints ("don't change X"、"must keep public API" 等)。
- Desired output ("return patch plan + tests"、"give 3 options with tradeoffs")。

## 安全

- 默认不要附加 secrets (`.env`、key files、auth tokens)。积极 redact；只分享必需的。

## "Exhaustive prompt" 恢复模式

对于长期调查，编写一个 standalone prompt + file set，以便你可以在几天后 rerun：

- 6–30 sentence project briefing + goal。
- Repro steps + exact errors + what you tried。
- 附加所有需要的 context files (entrypoints、configs、key modules、docs)。

Oracle runs 是 one-shot；model 不记得 prior runs。"Restoring context" 意味着用相同的 prompt + `--file …` set rerun（或 reattaching 一个仍在运行的 stored session）。
