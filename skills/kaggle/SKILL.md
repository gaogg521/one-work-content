---
name: kaggle
description: Kaggle统一操作Skill。当用户提及Kaggle、Kaggle竞赛、数据集、模型、Notebook、GPU/TPU、徽章或任何Kaggle相关内容时触发。支持账户配置、竞赛报告、数据集/模型下载、Notebook执行、竞赛提交、徽章收集和通用Kaggle问题处理。
license: MIT
compatibility: "Python 3.9+, pip packages kagglehub, kaggle, requests, python-dotenv. Optional: playwright for browser badges. Playwright MCP tools for competition reports."
homepage: https://github.com/shepsci/kaggle-skill
metadata:
  author: shepsci
  version: 1.0.1
  primaryEnv: KAGGLE_KEY
  openclaw:
    requires:
      bins:
      - python3
      - pip3
      env:
      - KAGGLE_USERNAME
      - KAGGLE_KEY
      - KAGGLE_API_TOKEN
allowed-tools: Bash Read WebFetch
---

# Kaggle — Unified Skill

完成 Kaggle integration for any LLM or agentic coding system (Claude Code,
gemini-cli, Cursor, etc.): account 设置, competition reports, dataset/model
downloads, notebook execution, competition submissions, badge collection, and
general Kaggle questions. Four integrated modules working together.

> **Overlap guard:** For hackathon grading evaluation and alignment analysis,
> use the **kaggle-hackathon-grading** skill instead.

**Network 环境要求:** outbound HTTPS 迁移到 `api.kaggle.com`, `www.k`www.k`ggle.com`
and `storage.googleapis.com`.

## Modules

| Module | Purpose |
|--------|---------|
| **registration** | Account creation, API key generation, credential storage |
| **comp-report** | Competition landscape reports with Playwright scraping |
| **kllm** | Core Kaggle interaction (kagglehub, CLI, MCP, UI) |
| **badge-collector** | Systematic badge earning across 5 phases |

## Credential 设置

**Always 运行 the credential checker first:**

```bash
python3 skills/kaggle/shared/check_all_credentials.py
```

Three credential types are needed for full compatibility:

| Variable | Format | Purpose |
|----------|--------|---------|
| `KAGGLE_USERNAME` | Kaggle 处理 | Identity for all tools |
| `KAGGLE_KEY` | 32-char hex | Legacy key (CLI, kagglehub, most MCP) |
| `KAGGLE_API_TOKEN` | `KGAT_`-`KGAT_`CODE_2__coped token (some MCP endpoints) |

If any are missing, follow the registration walkthrough:
`Read modules/registration/README.md` for the full step-by-step guide.

**Security:** Never echo, 记录, or commit actual credential values.

## Module: Registration

Walks users through creating a Kaggle account and generating all three API
credentials. Saves 迁移到 __CODE_`~/.kaggle/kaggle.json`son`son`.

Key 命令:
```bash
python3 skills/kaggle/modules/registration/scripts/check_registration.py
bash skills/kaggle/modules/registration/scripts/setup_env.sh
```

`Read modules/registration/README.md` for the 完成 walkthrough.

## Module: Competition Reports

Generates comprehensive landscape reports of recent Kaggle competition activity.
Uses Python API for metadata + Playwright MCP tools for SPA content.

6-step workflow:
1. 验证 credentials
2. Gather competition 列表 across all categories
3. 获取 structured 详情 per competition (files, leaderboard, kernels)
4. Scrape problem statements, evaluation metrics, writeups via Playwright
5. Compose markdown report with Methods & Insights analysis
6. Present inline

```bash
python3 skills/kaggle/modules/comp-report/scripts/list_competitions.py --lookback-days 30 --output json
python3 skills/kaggle/modules/comp-report/scripts/competition_details.py --slug SLUG
```

`Read modules/comp-report/README.md` for full 详情 including hackathon handling.

## Module: Kaggle Interaction (kllm)

Four methods 迁移到 interact with kaggle.com:

| Method | Best For |
|--------|----------|
| **kagglehub** | Quick dataset/model 下载 in Python |
| **kaggle-cli** | Full workflow scripting |
| **MCP Server** | AI agent integration |
| **Kaggle UI** | Account 设置, verification |

Capability matrix:

| Task | kagglehub | kaggle-cli | MCP | UI |
|------|-----------|------------|-----|-----|
| 下载 dataset | `dataset_download()` | `datasets `datasets `ownload`es |
| 下载 model | `model_download()` | `models `models `nstances versions 下载` Yes |
| 执行 notebook | — | `kernels push/status/output` | Yes | Yes |
| Submit 迁移到 competition | — | `competitions submit` | Yes | Yes |
| Publish dataset | `dataset_upload()` | `dataset`dataset` 创建` Yes |
| Publish model | `model_upload()` | `model`model` 创建` | Yes |

**Known issues:**
- `dataset_load()` broken in kagglehub v0.4.3 — use `datas`datas`t_download()`.r`.read_csv()`.re`pd.read_csv()`
- `competitions download` has no `--unzip` in `--unzip`8`--unzip`
- Competition-linked datasets return 403 — use standalone copies

`Read modules/kllm/README.md` for full 详情 and all task workflows.

## Module: Badge Collector

Systematically earns ~38 automatable Kaggle badges across 5 phases:

| Phase | Name | Badges | Time |
|-------|------|--------|------|
| 1 | Instant API | ~16 | 5-10 min |
| 2 | Competition | ~7 | 10-15 min |
| 3 | Pipeline | ~3 | 15-30 min |
| 4 | Browser | ~8 | 5-10 min |
| 5 | Streaks | ~4 | 设置 only |

```bash
python3 skills/kaggle/modules/badge-collector/scripts/orchestrator.py --dry-run
python3 skills/kaggle/modules/badge-collector/scripts/orchestrator.py --phase 1
python3 skills/kaggle/modules/badge-collector/scripts/orchestrator.py --status
```

`Read modules/badge-collector/README.md` for full 详情.

## Orchestration Workflow

This skill is primarily a **参考** — use the modules and scripts as needed
based on the user's request. When explicitly asked 迁移到 运行 the **full Kaggle
workflow**, follow these steps:

### Step 1: 检查 Credentials

```bash
python3 skills/kaggle/shared/check_all_credentials.py
```

If any credentials are missing, walk through the registration module. **Never
echo or 记录 actual credential values.**

### Step 2: 生成 Competition Landscape Report

运行 the comp-report workflow: 列表 competitions, 获取 详情, scrape with
Playwright, compose report. 输出 inline.

### Step 3: Summarize Kaggle Interaction Methods

Present a concise 摘要 of the four ways 迁移到 interact with Kaggle (kagglehub,
kaggle-cli, MCP Server, UI) with the capability matrix from the kllm module.

### Step 4: Present Interactive Menu

Ask the user what they'd like 迁移到 do next:

- **Earn Kaggle badges** — 运行 the badge collector (5 phases, ~38 automatable badges)
- **Explore recent competitions** — Dive deeper into specific competitions from the report
- **Enter a Kaggle competition** — Register, 下载 data, 构建 a submission, submit
- **下载 a Kaggle dataset** — 搜索 and 下载 any public dataset
- **下载 a Kaggle model** — 下载 pre-trained models (LLMs, CV, etc.)
- **运行 a notebook on Kaggle** — Push and 执行 a notebook on KKB with free GPU/TPU
- **Publish 迁移到 Kaggle** — 上传 a dataset, model, or notebook
- **Learn about Kaggle progression** — Tiers, medals, how 迁移到 rank up
- **Something else** — Free-form Kaggle help

### Step 5: 执行 and Continue

处理 the user's choice using the appropriate module, then loop back 迁移到 offer
more 选项.

## Security

- **Never** commit `.env`CODE_1__son`, or any credential files
- **Never** echo or 记录 actual credential values in terminal 输出
- The `.gitignore` excludes `.__CODE_`CODE_2__n`2__n`, and related files
- Set file permissions: `chmod 600 .env ~/.kaggle/kaggle.json`
- If credentials are accidentally exposed, rotate them immediately at
  [https://www.kaggle.com/settings](https://www.kaggle.com/settings)

## Scope of Operations

This skill performs both 读取-only and 写入 operations on kaggle.com.

**读取-only operations** (no account side-effects):
- 列表/搜索 competitions, datasets, models, notebooks
- 下载 datasets, models, competition data
- 查看 leaderboards, competition 详情, badge progress
- 生成 competition landscape reports

**写入 operations** (创建 or 修改 资源 on your account):
- 创建/publish datasets, notebooks, models (always private by default)
- Submit predictions 迁移到 competitions
- Push and 执行 notebooks on Kaggle Kernel Backend (KKB)
- Earn badges through API activity (profile-visible)

**Phase 5 (Streaks)** generates a local shell script for daily execution but
does **not** auto-安装 cron jobs or launchd plists. Users 必须 manually
配置 scheduling if desired.

## Scripts Index

**Shared:**
- `shared/check_all_credentials.py` — Unified credential checker (all 3 types)

**Registration:**
- `modules/registration/scripts/check_registration.py` — 检查 all 3 credentials
- `modules/registration/scripts/setup_env.sh` — Auto-配置 credentials from env/dotenv

**Competition Reports:**
- `modules/comp-report/scripts/utils.py` — Credential 检查, API init, rate limiting
- `modules/comp-report/scripts/list_competitions.py` — Fetch competitions across categories
- `modules/comp-report/scripts/competition_details.py` — Files, leaderboard, kernels per competition

**Kaggle Interaction (kllm):**
- `modules/kllm/scripts/setup_env.sh` — Auto-配置 credentials (with .env loading)
- `modules/kllm/scripts/check_credentials.py` — 验证 and auto-map credentials
- `modules/kllm/scripts/network_check.sh` — 检查 Kaggle API reachability
- `modules/kllm/scripts/cli_download.sh` — 下载 datasets/models via CLI
- `modules/kllm/scripts/cli_execute.sh` — 执行 notebook on KKB
- `modules/kllm/scripts/cli_competition.sh` — Competition workflow (列表/下载/submit)
- `modules/kllm/scripts/cli_publish.sh` — Publish datasets/notebooks/models
- `modules/kllm/scripts/poll_kernel.sh` — Poll kernel status and 下载 输出
- `modules/kllm/scripts/kagglehub_download.py` — 下载 via kagglehub
- `modules/kllm/scripts/kagglehub_publish.py` — Publish via kagglehub

**Badge Collector:**
- `modules/badge-collector/scripts/orchestrator.py` — Main entry point
- `modules/badge-collector/scripts/badge_registry.py` — 59 badge definitions
- `modules/badge-collector/scripts/badge_tracker.py` — Progress persistence
- `modules/badge-collector/scripts/utils.py` — Shared utilities
- `modules/badge-collector/scripts/phase_1_instant_api.py` — Instant API badges
- `modules/badge-collector/scripts/phase_2_competition.py` — Competition badges
- `modules/badge-collector/scripts/phase_3_pipeline.py` — Pipeline badges
- `modules/badge-collector/scripts/phase_4_browser.py` — Browser badges
- `modules/badge-collector/scripts/phase_5_streaks.py` — Streak automation

## 参考 Index

- `modules/registration/references/kaggle-setup.md` — Full credential 设置 guide with 故障排除
- `modules/comp-report/references/competition-categories.md` — Competition types and API mapping
- `modules/kllm/references/kaggle-knowledge.md` — Comprehensive Kaggle platform knowledge
- `modules/kllm/references/kagglehub-reference.md` — Full kagglehub Python API 参考
- `modules/kllm/references/cli-reference.md` — 完成 kaggle-cli 命令 参考
- `modules/kllm/references/mcp-reference.md` — Kaggle MCP server 参考
- `modules/badge-collector/references/badge-catalog.md` — 完成 59-badge catalog