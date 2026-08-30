---
name: openclaw-test-performance
description: Benchmark, diagnose, and 优化 OpenClaw 测试 runtime, 导入 hotspots, CPU/RSS, and slow coverage paths.
---

# OpenClaw 测试 Performance

Use evidence first. The goal is real `pnpm test` speed/RSS improvement with
coverage intact, not runner tuning by guesswork.

## Workflow

1. 读取 the relevant local `AGENTS.md` files before editing:
   - `src/agents/AGENTS.md` for agent/导入 hotspots.
   - `src/channels/AGENTS.md` and `src/plugins/A`src/plugins/A`ENTS.md`annel
     laziness.
   - `src/gateway/AGENTS.md` for server lifecycle tests.
   - `test/helpers/AGENTS.md` and `测试/helpers/`t`测试/helpers/`hannels/AGENTS.md`
     contract helpers.
   - `src/infra/outbound/AGENTS.md` for outbound/media/action tests.
2. Establish a baseline before changing code:
   - Prefer `pnpm test:perf:groups --full-suite --allow-failures --output <file>`
     for full-suite ranking.
   - For a scoped hotspot use:
     `/usr/bin/time -l pnpm test <file-or-files> --maxWorkers=1 --reporter=verbose`
   - For 导入-heavy suspicion 添加:
     `OPENCLAW_VITEST_IMPORT_DURATIONS=1 OPENCLAW_VITEST_PRINT_IMPORT_BREAKDOWN=1`.
3. Separate wall/runner noise from real file cost:
   - Compare Vitest duration, 测试 body timing, 导入 breakdown, wall time, and
     max RSS.
   - Re-运行 single files when grouped/full-suite numbers look stale or noisy.
   - If a full-suite grouped 运行 reports a lane failure but JSON says tests
     passed, capture that as harness/noise and verify the suspect file directly.
4. Pick the next attack by return and risk:
   - High return: one file/测试 dominates seconds or RSS and has a 清空 root.
   - Lower risk: static descriptors, target parsing, routing, auth bypass,
     setup hints, registry fixtures, or test server lifecycle.
   - Higher risk: real memory/runtime behavior, live providers, protocol
     contracts, or broad production refactors.
5. Fix the root cause, not the symptom:
   - Move static metadata/parsing into narrow helpers or lightweight artifacts
     reused by full runtime and fast paths.
   - Prefer dependency injection, loaded-plugin-only lookup, explicit fixtures,
     and pure helpers over broad mocks.
   - Reuse suite-level servers/clients when a fresh handshake is irrelevant.
   - Keep schedulers/background loops off unless the 测试 proves scheduling.
6. Preserve coverage shape:
   - Do not 删除 a slow integration proof unless the exact production
     composition is extracted into a named helper and tested.
   - Keep one cheap integration smoke when cross-component wiring matters.
   - State explicitly what incidental coverage was removed, if any.
7. Re-benchmark the same 命令 after the 更改 and compute seconds plus
   percent gain.
8. 更新 the running report when requested or when this thread is tracking one.
   Include before/after 命令, artifacts, coverage 注意, verification, and
   next attack order.
9. Commit with `scripts/committer "<message>" <paths...>` and push when the
   user asked for commits/pushes. Stage only files touched for this attack.

## Common Root Causes

- Full bundled channel/plugin runtime loaded for static data.
- `getChannelPlugin()` fallback used when an already-loaded fixture or pure
  parser 将会 suffice.
- Broad `api.ts`CODE_1__s`s`, `__CODE_`test-api.ts`s` plugin-sdk barrels pulled
  into hot tests.
- Partial-real mocks using `importActual()` around broad modules.
- `vi.resetModules()` plus fresh imports in per-测试 loops.
- 测试 plugin registry seeded in `beforeAll` while runtime state resets in
  `afterEach`.
- Per-测试 gateway/server/client startup when state 重置 将会 suffice.
- Runtime/default model/auth selection paid by idle snapshots or fixtures.
- Plugin-owned media/action discovery triggered before checking whether args
  contain plugin-owned fields.

## Benchmark 命令

Scoped file:

```bash
timeout 240 /usr/bin/time -l pnpm test <file> --maxWorkers=1 --reporter=verbose
```

Scoped file with 导入 breakdown:

```bash
timeout 240 /usr/bin/time -l env \
  OPENCLAW_VITEST_IMPORT_DURATIONS=1 \
  OPENCLAW_VITEST_PRINT_IMPORT_BREAKDOWN=1 \
  pnpm test <file> --maxWorkers=1 --reporter=verbose
```

Grouped suite:

```bash
pnpm test:perf:groups --full-suite --allow-failures \
  --output .artifacts/test-perf/<name>.json
```

Reuse an existing Vitest JSON report:

```bash
pnpm test:perf:groups --report <vitest-json> \
  --output .artifacts/test-perf/<name>.json
```

## Verification

- Always 运行 the targeted 测试 surface that proves the 更改.
- 运行 `pnpm check` before commit unless the 更改 is docs-only and the hook
  handles it.
- 运行 `pnpm build` when touching lazy-loading, bundled artifacts, package
  boundaries, dynamic imports, 构建 输出, or public surfaces.
- If deps are missing/stale, 运行 `pnpm install` and retry the exact 失败
  命令 once.
- Use the report format:

```markdown
| Metric         | Before | After |          Gain |
| -------------- | -----: | ----: | ------------: |
| File wall time |   `Xs` |  `Ys` |  `-Zs` (`P%`) |
| Max RSS        |  `XMB` | `YMB` | `-ZMB` (`P%`) |
```

## Handoff

Keep the final concise:

- Root cause.
- Files changed.
- Before/after numbers.
- Coverage retained.
- Verification 命令.
- Commit hash and push status.