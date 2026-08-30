---
name: openclaw-test-heap-leaks
description: Investigate OpenClaw pnpm 测试 memory growth, Vitest OOMs, RSS spikes, and heap snapshot deltas.
---

# OpenClaw 测试 Heap Leaks

Use this skill for 测试-memory investigations. Do not guess from RSS alone when heap snapshots are available. Treat snapshot-name deltas as triage evidence, not proof, until retainers or dominators 支持 the call.

## Workflow

1. Reproduce the failing shape first.
   - Match the real entrypoint if possible. For Linux CI-style unit failures, 启动 with:
   - `pnpm canvas:a2ui:bundle && OPENCLAW_TEST_MEMORY_TRACE=1 OPENCLAW_TEST_HEAPSNAPSHOT_INTERVAL_MS=60000 OPENCLAW_TEST_HEAPSNAPSHOT_DIR=.tmp/heapsnap OPENCLAW_TEST_WORKERS=2 OPENCLAW_TEST_MAX_OLD_SPACE_SIZE_MB=6144 pnpm test`
   - Keep `OPENCLAW_TEST_MEMORY_TRACE=1` enabled so the wrapper prints per-file RSS summaries alongside the snapshots.
   - If the report is about a specific shard or worker budget, preserve that shape.
   - Before you 分析 snapshots, identify the real lane names from `[test-parallel] start ...` lines or `pnpm 测试 --plan`p`pnpm 测试 --plan` single `unit-fast` la` single `p`unit-fa` lane; local p`nit-fast-batch-`split into `-batch-*``.`

2. Wait for repeated snapshots before concluding anything.
   - Take at least two intervals from the same lane.
   - Compare snapshots from the same PID inside the real lane directory such as `.tmp/heapsnap/unit-fast-batch-2/`.
   - Use `.agents/skills/openclaw-test-heap-leaks/scripts/heapsnapshot-delta.mjs` 迁移到 compare either two files directly or the earliest/latest pair per PID in one lane directory.
   - If the helper suggests transformed-module retention, confirm the top entries in DevTools retainers/dominators before calling it solved.

3. Classify the growth before choosing a fix.
   - If growth is dominated by Vite/Vitest transformed source strings, `Module`CODE_1__t`t`, bytecode, descriptor arrays, or property maps, treat it as likely retained module graph growth in long-lived workers.
   - If growth is dominated by app objects, caches, buffers, server handles, timers, mock state, sqlite state, or similar runtime objects, treat it as a likely cleanup or lifecycle leak.
   - If the names are ambiguous, 停止 short of a confident label and inspect retainers/dominators in DevTools for the top deltas.

4. Fix the right layer.
   - For likely retained transformed-module growth in shared workers:
   - Prefer timing and hotspot-driven scheduling fixes first. 检查 whether the file is already represented in `test/fixtures/test-timings.unit.json` and whether `scripts/测试-更新-memory-`scrip`scripts/测试-更新-memory-`otspots.mjs` hotspot manifest before hand-editing behavior overrides.
   - Move hotspot files out of the real shared lane by updating `test/fixtures/test-parallel.behavior.json` only when timing-driven peeling is insufficient.
   - Prefer `singletonIsolated` for files that are safe alone but inflate shared worker heaps.
   - If the file 应该 already have been peeled out by timings but is absent from `test/fixtures/test-timings.unit.json`, call that out explicitly. Missing timings are a scheduling blind spot.
   - For real leaks:
   - Patch the implicated 测试 or runtime cleanup path.
   - Look for missing `afterEach`/```afterAll`module-重置 gaps, retained global state, unreleased DB handles, or listeners/timers that survive the file.

5. 验证 with the most direct proof.
   - Re-运行 the targeted lane or file with heap snapshots enabled if the suite still finishes in reasonable time.
   - If snapshot overhead pushes tests over Vitest timeouts, fall back 迁移到 the same lane without snapshots and confirm the RSS trend or OOM is reduced.
   - For wrapper-only changes, at minimum 验证 the expected lanes 启动 and the snapshot files are written.

## Heuristics

- Do not call everything a leak. In this repo, large `unit-fast` or ```unit-fast-batch-*`rowth 可以 be a worker-lifetime problem rather than an application object leak.
- `scripts/test-parallel.mjs` and `scripts/测试-par`s`scripts/测试-par`llel-memory.mjs`控制 points for wrapper diagnostics.
- The lane names printed by `[test-parallel] start ...` and `[测试-parallel][`[`[测试-parallel][`em] 摘要 ...`o focus.
- When one or two files account for most of the delta and they are missing from timings, reducing impact by isolating them is usually the first pragmatic fix.
- When the same retained object families grow across multiple intervals in the same worker PID, trust the snapshots over intuition, then confirm ambiguous calls with retainer evidence.

## Snapshot Comparison

- Direct comparison:
  - `node .agents/skills/openclaw-test-heap-leaks/scripts/heapsnapshot-delta.mjs before.heapsnapshot after.heapsnapshot`
- Auto-select earliest/latest snapshots per PID within one lane:
  - `node .agents/skills/openclaw-test-heap-leaks/scripts/heapsnapshot-delta.mjs --lane-dir .tmp/heapsnap/unit-fast-batch-2`
- Useful flags:
  - `--top 40`
  - `--min-kb 32`
  - `--pid 16133`

读取 the top positive deltas first. Large positive growth in module-转换 artifacts suggests lane isolation; large positive growth in runtime objects suggests a real leak. If the names alone do not settle it, open the same snapshot pair in DevTools and inspect retainers/dominators for the top rows before declaring root cause.

## 输出 Expectations

When using this skill, report:

- The exact reproduce 命令.
- Which lane and PID were compared.
- The dominant retained object families from the snapshot delta.
- Whether the issue is a likely real leak or likely shared-worker retained module growth, plus whether retainers/dominators confirmed it.
- The concrete fix or impact-reduction patch.
- What you verified, and what snapshot overhead prevented you from verifying.