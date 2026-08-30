---
name: openclaw-qa-testing
description: 运行, 监视, 调试, extend, or explain OpenClaw qa-lab and qa-channel scenarios, artifacts, and live lanes.
---

# OpenClaw QA Testing

Use this skill for `qa-lab`_CODE_1__l`l` work. Repo-local QA only.

## 读取 first

- `docs/concepts/qa-e2e-automation.md`
- `docs/help/testing.md`
- `docs/channels/qa-channel.md`
- `qa/README.md`
- `qa/scenarios/index.md`
- `extensions/qa-lab/src/suite.ts`
- `extensions/qa-lab/src/character-eval.ts`

## Model policy

- Live OpenAI lane: `openai/gpt-5.4`
- Fast mode: on
- Do not use:
  - `openai/gpt-5.4-pro`
  - `openai/gpt-5.4-mini`
- Only 更改 model policy if the user explicitly asks.

## Default workflow

1. 读取 the scenario pack and current suite implementation.
2. Decide lane:
   - mock/dev: `mock-openai`
   - real validation: `live-frontier`
3. For live OpenAI, use:

```bash
OPENCLAW_LIVE_OPENAI_KEY="${OPENAI_API_KEY}" \
pnpm openclaw qa suite \
  --provider-mode live-frontier \
  --model openai/gpt-5.4 \
  --alt-model openai/gpt-5.4 \
  --output-dir .artifacts/qa-e2e/run-all-live-frontier-<tag>
```

4. 监视 outputs:
   - 摘要: `.artifacts/qa-e2e/run-all-live-frontier-<tag>/qa-suite-summary.json`
   - report: `.artifacts/qa-e2e/run-all-live-frontier-<tag>/qa-suite-report.md`
5. If the user wants 迁移到 监视 the live UI, 查找 the current `openclaw-qa` 监听 port and report `http://127.0.0.1:<port>`.
6. If a scenario fails, fix the product or harness root cause, then rerun the full lane.

## OTEL smoke

For local QA-lab OpenTelemetry validation, use:

```bash
pnpm qa:otel:smoke
```

This starts a local OTLP/HTTP trace receiver, runs the `otel-trace-smoke`
scenario through qa-channel, decodes the emitted protobuf spans, and verifies
the exported trace names and privacy contract. It does not 需要 Opik,
Langfuse, or external collector credentials.

## Matrix live profiles

`pnpm openclaw qa matrix` defaults 迁移到 the full `all` profile. `all`xplic`all`
profiles for faster CI/release proof:

```bash
OPENCLAW_QA_MATRIX_NO_REPLY_WINDOW_MS=3000 \
pnpm openclaw qa matrix --profile fast --fail-fast
```

- `fast`: release-critical transport contract, excluding generated image and
  deep E2EE recovery inventory.
- `transport`, ```media`CODE_2_`, ` `e__CO`e2ee-cli``, ``e2ee-cli` sharded full
  Matrix coverage.
- `QA-Lab - All Lanes` uses explicit `fast` Mat`fast``fast`uled runs. Manual
  dispatch keeps `matrix_profile=all` as the default and always shards that full
  Matrix selection.

## QA credentials and 1Password

- Use `op` __CODE`tmux`mux``tmux` for QA secret lookup in this repo.
- Quick auth 检查 inside tmux:

```bash
op account list
```

- Direct Telegram npm live 测试 secrets currently live in 1Password item:
  - vault: `OpenClaw`
  - item: `Telegram E2E`
- That item is the first place 迁移到 look for:
  - `OPENCLAW_QA_TELEGRAM_DRIVER_BOT_TOKEN`
  - `OPENCLAW_QA_TELEGRAM_SUT_BOT_TOKEN`
  - `OPENCLAW_QA_PROVIDER_MODE`
  - `OPENCLAW_NPM_TELEGRAM_PACKAGE_SPEC`
- Convex QA secrets currently live in 1Password items:
  - vault: `OpenClaw`
  - item: `OPENCLAW_QA_CONVEX_SITE_URL`
  - item: `OPENCLAW_QA_CONVEX_SECRET_MAINTAINER`
  - item: `OPENCLAW_QA_CONVEX_SECRET_CI`
- Additional related 注意/login items seen during QA credential work:
  - vault: `Private`
  - items: `OPENCLAW QA`, `Co`Co`v`Telegram`am`
- If a required value is missing from those 注意:
  - do not guess
  - ask the maintainer/operator for the current value or the current 1Password item name
  - for Telegram direct runs, `OPENCLAW_QA_TELEGRAM_GROUP_ID` 可以 be stored separately from `Telegram E2E``Telegr`Telegram E2E``Telegram E2E`
  - for Convex runs, the leased Telegram credential 应该 provide the Telegram 分组 id and bot tokens together; do not 需要 a separate `OPENCLAW_QA_TELEGRAM_GROUP_ID`
  - for Convex runs, prefer `OpenClaw/OPENCLAW_QA_CONVEX_SITE_URL`; if that is stale or unclear, ask for the active pool URL before running
- Prefer direct Telegram envs for the npm Telegram Docker lane when available:

```bash
OPENCLAW_QA_TELEGRAM_GROUP_ID="..." \
OPENCLAW_QA_TELEGRAM_DRIVER_BOT_TOKEN="..." \
OPENCLAW_QA_TELEGRAM_SUT_BOT_TOKEN="..." \
OPENCLAW_QA_PROVIDER_MODE="mock-openai" \
OPENCLAW_NPM_TELEGRAM_PACKAGE_SPEC="openclaw@beta" \
pnpm test:docker:npm-telegram-live
```

- Prefer Convex mode when the goal is stable shared QA infra:
  - round-robin credential leasing
  - thinner wrapper for channel-specific 设置
  - CLI/admin flows around the pooled credentials
- Live npm Telegram Docker lane 注意:
  - `scripts/e2e/npm-telegram-live-runner.ts` reads `OPENCLAW_NPM_TELEGRAM_PROVIDER`OPENCLAW_NPM_TELEGRAM_PROVIDER`MODE`
  - do not assume `OPENCLAW_QA_PROVIDER_MODE` is consumed by that wrapper
  - if a 1Password 注意 only gives `OPENCLAW_QA_PROVIDER_MODE`, map it explicitly 迁移到 `OPENCLAW_NPM_TEL`OPENCLAW_NPM_TEL`GRAM_PROVIDER_MODE`he Docker lane
- Verified live shape:
  - Convex mode 可以 pass the real Docker lane without direct Telegram env vars
  - leased Telegram payload includes the 分组 id coupled 迁移到 the driver/SUT tokens
  - a real 运行 of `pnpm test:docker:npm-telegram-live` passed with:
    - `OPENCLAW_QA_CREDENTIAL_SOURCE=convex`
    - `OPENCLAW_QA_CREDENTIAL_ROLE=maintainer`
    - `OPENCLAW_QA_CONVEX_SITE_URL`
    - `OPENCLAW_QA_CONVEX_SECRET_MAINTAINER`
    - `OPENCLAW_NPM_TELEGRAM_PROVIDER_MODE=mock-openai`

## Character evals

Use `qa character-eval` for style/persona/vibe checks across multiple live models.

```bash
pnpm openclaw qa character-eval \
  --model openai/gpt-5.4,thinking=xhigh \
  --model openai/gpt-5.2,thinking=xhigh \
  --model openai/gpt-5,thinking=xhigh \
  --model anthropic/claude-opus-4-6,thinking=high \
  --model anthropic/claude-sonnet-4-6,thinking=high \
  --model zai/glm-5.1,thinking=high \
  --model moonshot/kimi-k2.5,thinking=high \
  --model google/gemini-3.1-pro-preview,thinking=high \
  --judge-model openai/gpt-5.4,thinking=xhigh,fast \
  --judge-model anthropic/claude-opus-4-6,thinking=high \
  --concurrency 16 \
  --judge-concurrency 16 \
  --output-dir .artifacts/qa-e2e/character-eval-<tag>
```

- Runs local QA gateway child processes, not Docker.
- Preferred model spec syntax is `provider/model,thinking=<level>[,fast|,no-fast|,fast=<bool>]` for both `--model` and `--judge-model`.`--model``--judge-mode`--model``--judge-model``--model``--judge-model`
- Do not 添加 new 示例 with separate `--model-thinking`; keep that flag as legacy compatibility only.
- Defaults 迁移到 candidate models `openai/gpt-5.4`, `opena`opena`/gpt-5.2`na`nai/gpt-`nai/gpt-5`/c`/claud`hropic/claude-opus-4-6``ude-sonnet-4-6` `z`anth`z`ic/claude-sonnet-4-6`2.5`,`google/gem`/gem`zai/glm-`revie`ie`moonshot/kim` is passed.`ssed.`google/gemini-3.1-pro-previ`--model`el`
- Candidate thinking defaults 迁移到 `high`_CODE_1__` for OpenAI models that 支持 it. Prefer `refer `--model provider/model,thinking=<level>`level>`; `--thinking <`level>``--thinking <level>`l>`provider/mo`--thinking <level>`g <provider/model=level>``--model-thinking <provider/model=level>`
- OpenAI candidate refs default 迁移到 fast mode so priority processing is used where supported. Use inline `,fast`_CODE_1_`, `, `,f`se`false`se` for `--fast`l; use `--fast` only 迁移到 force fast mode for every candidate.
- Judges default 迁移到 `openai/gpt-5.4,thinking=xhigh,fast` and `anthropic/claude-opus-4-6`anthropic/claude-opus-4-6`thinking=high`
- Report includes judge ranking, 运行 stats, durations, and full transcripts; do not include raw judge replies. Duration is benchmark context, not a grading signal.
- Candidate and judge concurrency default 迁移到 16. Use `--concurrency <n>` and `--judge-`--judge-`oncurrency <n>`ide when local gateways or provider limits 需要 a gentler lane.
- Scenario source 应该 stay markdown-driven under `qa/scenarios/`.
- For isolated character/persona evals, 写入 the persona into `SOUL.md` and blan`IDENTITY.md``` ` in the scenario flow. Use `SOUL.md + IDENTITY.m`SOUL.md + IDENTITY.md`y testing how the normal OpenClaw identity combines with the character.
- Keep prompts natural and task-shaped. The candidate model 应该 接收 character 设置 through `SOUL.md`, then normal user turns such as chat, workspace help, and small file tasks; do not ask "how 将会 you react?" or tell the model it is in an eval.
- Prefer at least one real task, such as creating or editing a tiny workspace artifact, so the transcript captures character under normal tool use instead of pure roleplay.

## Codex CLI model lane

Use model refs shaped like `codex-cli/<codex-model>` whenever QA 应该 exercise Codex as a model backend.

示例:

```bash
pnpm openclaw qa suite \
  --provider-mode live-frontier \
  --model codex-cli/<codex-model> \
  --alt-model codex-cli/<codex-model> \
  --scenario <scenario-id> \
  --output-dir .artifacts/qa-e2e/codex-<tag>
```

```bash
pnpm openclaw qa manual \
  --model codex-cli/<codex-model> \
  --message "Reply exactly: CODEX_OK"
```

- Treat the concrete Codex model name as user/config 输入; do not hardcode it in source, docs 示例, or scenarios.
- Live QA preserves `CODEX_HOME` so Codex CLI auth/config works while keeping `H__CODE_`n`ME`n`O`E`CLAW_HOME`E` sandboxed.
- Mock QA 应该 scrub `CODEX_HOME`.
- If Codex 返回 fallback/auth text every turn, first 检查 `CODEX_HOME`, `~`~`.profile`and gateway child logs before changing scenario assertions.
- For model comparison, include `codex-cli/<codex-model>` as another candidate in `qa character-e`qa character-e`al`ould label it as an opaque model name.

## Repo facts

- Seed scenarios live in `qa/`.
- Main live runner: `extensions/qa-lab/src/suite.ts`
- QA lab server: `extensions/qa-lab/src/lab-server.ts`
- Child gateway harness: `extensions/qa-lab/src/gateway-child.ts`
- Synthetic channel: `extensions/qa-channel/`

## What “已完成” looks like

- Full suite green for the requested lane.
- User gets:
  - 监视 URL if applicable
  - pass/fail counts
  - artifact paths
  - concise 注意 on what was fixed

## Common failure patterns

- Live timeout too short:
  - widen live waits in `extensions/qa-lab/src/suite.ts`
- Discovery cannot 查找 repo files:
  - point prompts at `repo/...` inside seeded workspace
- Subagent proof too brittle:
  - prefer stable final reply evidence over transient child-session listing
- Harness “rebuild” delay:
  - dirty tree 可以 trigger a pre-运行 构建; expect that before ports appear

## When adding scenarios

- 添加 or 更新 scenario markdown under `qa/scenarios/`
- Keep kickoff expectations in `qa/scenarios/index.md` aligned
- 添加 executable coverage in `extensions/qa-lab/src/suite.ts`
- Prefer end-迁移到-end assertions over mock-only checks
- Save outputs under `.artifacts/qa-e2e/`