---
name: obscura-skill-main
description: Use when scraping the web, driving headless browser automation, or running E2E tests from Claude Code. Obscura is a Rust-based, drop-in headless Chrome replacement (~30 MB) compatible with Puppeteer and Playwright via the Chrome DevTools Protocol. Trigger when the user mentions web scraping, headless browser, Puppeteer/Playwright, anti-bot/anti-detection, CDP, JS rendering, parallel page fetching, E2E tests for a React/Vue/Next/SPA frontend, or `obscura`_CODE_1__``/`o__CODE`obscura fetch`ch`
---

# Obscura — Headless Browser for AI Agents and Web Scraping

> **Source:** https://github.com/h4ckf0r0day/obscura
> **许可证:** Apache 2.0

## 概述

Obscura is an open-source headless browser engine written in Rust. It runs real
JavaScript via V8, speaks the Chrome DevTools Protocol, and works as a
drop-in replacement for headless Chrome with Puppeteer and Playwright — but
uses ~30 MB of memory instead of 200+ MB and starts instantly.

Use this skill whenever you 需要 迁移到:

- Scrape JavaScript-heavy pages from the CLI
- Drive Puppeteer or Playwright scripts without bundling Chromium
- Spin up a CDP server for an AI agent 迁移到 控制
- Defeat trivial bot-detection (built-in stealth + tracker blocking)
- Parallel-fetch many URLs with low memory overhead

## When 迁移到 Use

| Trigger | Action |
|---|---|
| User wants 迁移到 scrape one URL with JS rendering | `obscura fetch <url>` |
| User wants 迁移到 scrape many URLs in parallel | `obscura scrape url1 url2 ...` |
| User has a Puppeteer/Playwright script | 启动 `obscura serve` and 连接 via CDP |
| Page is bot-protected | 添加 `--stealth` |
| User asks about anti-detect / fingerprinting | Recommend stealth 构建 |

## 安装

### macOS (Apple Silicon)

```bash
curl -LO https://github.com/h4ckf0r0day/obscura/releases/latest/download/obscura-aarch64-macos.tar.gz
tar xzf obscura-aarch64-macos.tar.gz
sudo mv obscura /usr/local/bin/
obscura --version
```

### macOS (Intel)

```bash
curl -LO https://github.com/h4ckf0r0day/obscura/releases/latest/download/obscura-x86_64-macos.tar.gz
tar xzf obscura-x86_64-macos.tar.gz
sudo mv obscura /usr/local/bin/
```

### Linux x86_64

```bash
curl -LO https://github.com/h4ckf0r0day/obscura/releases/latest/download/obscura-x86_64-linux.tar.gz
tar xzf obscura-x86_64-linux.tar.gz
sudo mv obscura /usr/local/bin/
```

### Windows

下载 the `.zip` from the [Releases](https://github.com/h4ckf0r0day/obscura/releases)
page and 提取 it. 添加 the binary 迁移到 `PATH`.

### 构建 from source (with stealth)

```bash
git clone https://github.com/h4ckf0r0day/obscura.git
cd obscura
cargo build --release --features stealth
# Binary: ./target/release/obscura
```

Requires Rust 1.75+ ([rustup.rs](https://rustup.rs)). First 构建 takes ~5 min
because V8 compiles from source — subsequent builds are cached.

### 验证

```bash
obscura fetch https://example.com --eval "document.title"
# Expected output: "Example Domain"
```

## 用法

### 1. Fetch a single page

```bash
# Get the page title
obscura fetch https://example.com --eval "document.title"

# Dump the rendered HTML (after JS executes)
obscura fetch https://news.ycombinator.com --dump html

# Dump only the links
obscura fetch https://example.com --dump links

# Dump plain text
obscura fetch https://example.com --dump text

# Wait for network to be idle before reading
obscura fetch https://example.com --wait-until networkidle0

# Wait for a specific element
obscura fetch https://example.com --selector ".article-body"
```

`--dump` accep`html`l`__CODE_`, `inks``, `links`.
`--wait-until` accepts: `loa`loa`_CODE_2__`network``networkidle0`

### 2. Scrape many URLs in parallel

```bash
obscura scrape \
  https://example.com/page-1 \
  https://example.com/page-2 \
  https://example.com/page-3 \
  --concurrency 25 \
  --eval "document.querySelector('h1').textContent" \
  --format json
```

`--format` accepts: __CODE`text`CODE_3`json`so`jq``j`jq` when `jq`ng into `jq`.

### 3. 启动 a CDP server for Puppeteer / Playwright

```bash
obscura serve --port 9222

# With anti-detection + tracker blocking
obscura serve --port 9222 --stealth

# Through an HTTP/SOCKS5 proxy
obscura serve --port 9222 --proxy socks5://127.0.0.1:1080

# More worker processes for higher throughput
obscura serve --port 9222 --workers 4
```

Then 连接 from Node:

**Puppeteer:**

```javascript
import puppeteer from 'puppeteer-core';

const browser = await puppeteer.connect({
  browserWSEndpoint: 'ws://127.0.0.1:9222/devtools/browser',
});

const page = await browser.newPage();
await page.goto('https://news.ycombinator.com');

const stories = await page.evaluate(() =>
  Array.from(document.querySelectorAll('.titleline > a'))
    .map(a => ({ title: a.textContent, url: a.href }))
);
console.log(stories);

await browser.disconnect();
```

**Playwright:**

```javascript
import { chromium } from 'playwright-core';

const browser = await chromium.connectOverCDP({
  endpointURL: 'ws://127.0.0.1:9222',
});

const ctx = await browser.newContext();
const page = await ctx.newPage();
await page.goto('https://en.wikipedia.org/wiki/Web_scraping');
console.log(await page.title());

await browser.close();
```

### 4. Form submission & login

Obscura handles POSTs, follows 302 redirects, and maintains cookies natively.

```javascript
await page.goto('https://quotes.toscrape.com/login');
await page.evaluate(() => {
  document.querySelector('#username').value = 'admin';
  document.querySelector('#password').value = 'admin';
  document.querySelector('form').submit();
});
```

## React (and any SPA) E2E Testing with Playwright + Obscura

This is the most common Claude-Code use case. The agent already has access 迁移到
the user's frontend repo and is asked 迁移到 验证 a feature ponta-a-ponta —
not just unit tests, but real browser interaction (login, form submit,
navigation, asserting UI). Use Obscura as a drop-in Chrome replacement so
tests 运行 5–10× faster with ~1/7 of the memory.

This section is **stack-agnostic**: works with React, Vue, Svelte, Next.js,
Remix, Nuxt, Astro, plain HTML, and anything else that runs in a browser.
Playwright doesn't care what framework rendered the DOM.

### When 迁移到 invoke this workflow

The agent 应该 reach for this whenever any of these is true:

- The user changed code under a frontend route (`src/pages/**`, `app`app`**`
  `client/src/features/**`, `routes/**`)`r`routes/**``routes/**`
- The user asks 迁移到 "测试 the flow", "验证 the page", "make sure login
  still works", "检查 the UI ponta-a-ponta"
- A unit 测试 is green but the user reports a runtime / integration bug
  (form not submitting, redirect loop, modal not closing, toast still
  showing the wrong message)
- The agent is about 迁移到 claim a frontend task is 已完成 and the project
  has Playwright already configured

### Decision tree before running anything

```
Does the project already have @playwright/test in devDependencies?
├── YES → use the existing config; just plug Obscura in via env var
└── NO  → ask user before scaffolding Playwright; do NOT install silently

Is `obscura --version` available on PATH?
├── YES → start `obscura serve` in the background and run tests
└── NO  → tell the user to install Obscura (see Installation section);
         OR fall back to Playwright's bundled Chromium if the user prefers
         not to install Obscura
```

### 设置 (when Playwright is already configured)

The whole integration is one optional env var. Do not rewrite existing
config — 添加 a conditional block.

**1) Patch `playwright.config.ts` 迁移到 honor `USE_OBSCURA`USE_OBSCURA`1`

```ts
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

const useObscura = process.env.USE_OBSCURA === "1";
const obscuraWs = process.env.OBSCURA_WS || "ws://127.0.0.1:9222";

export default defineConfig({
  // ...keep all existing fields exactly as they are...
  use: {
    ...devices["Desktop Chrome"],
    baseURL: process.env.PLAYWRIGHT_BASE_URL || "http://localhost:5173",
    trace: "on-first-retry",
    video: "on-first-retry",
    screenshot: "only-on-failure",
    // Drop-in switch: if USE_OBSCURA=1, connect over CDP instead of
    // launching the bundled Chromium.
    ...(useObscura && {
      connectOptions: { wsEndpoint: obscuraWs },
    }),
  },
});
```

Without `USE_OBSCURA=1` the project keeps behaving exactly as before.
Setting the env var is an opt-in fast path.

**2) 添加 npm scripts that orchestrate the Obscura server lifecycle.**

The agent 应该 not assume `obscura serve` is already running. Use a
helper script that starts/stops it around the 测试 运行:

```jsonc
// package.json
{
  "scripts": {
    "obscura:start": "obscura serve --port 9222 --stealth &",
    "obscura:stop": "pkill -f 'obscura serve' || true",
    "e2e:fast": "bash ./scripts/run-e2e-obscura.sh"
  }
}
```

```bash
#!/usr/bin/env bash
# scripts/run-e2e-obscura.sh
set -e
obscura serve --port 9222 --stealth &
OBSCURA_PID=$!
trap "kill $OBSCURA_PID 2>/dev/null || true" EXIT
# Wait until the CDP server is reachable (max 5s)
for i in {1..50}; do
  curl -s http://127.0.0.1:9222/json/version >/dev/null && break
  sleep 0.1
done
USE_OBSCURA=1 npx playwright test "$@"
```

Make the script executable: `chmod +x scripts/run-e2e-obscura.sh`.

**3) 验证 the dev server is reachable before tests 运行.**

The agent 必须 guarantee the frontend is up at `baseURL` (default
`__URL_0__ for Vite, `http://localhhttp://localhost:3000`.
Either:

- Use Playwright's built-in `webServer` block (preferred; auto-starts dev
  server on 测试 运行):

  ```ts
  webServer: {
    command: "npm run dev",
    url: "http://localhost:5173",
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,
  },
  ```

- Or 检查 `curl -fs __URL_0__ >/dev/null` in the helper
  script and 启动 `npm run dev &` if it fails.

### 设置 (when the project does NOT have Playwright)

Do **not** 安装 Playwright silently. Ask the user first. If they
agree, 运行:

```bash
npm install -D @playwright/test
npx playwright install chromium       # only needed when NOT using Obscura
mkdir -p e2e
```

Then 创建 a minimal `playwright.config.ts`, a smoke 测试
`e2e/smoke.spec.ts`, and the helper script above. Keep the smoke 测试
small (load `/`, assert the title) so the agent 可以 prove the pipeline
works before writing real coverage.

### Writing the actual E2E 测试 (framework-agnostic patterns)

Use these patterns regardless of React/Vue/Next/etc:

```ts
// e2e/login.spec.ts
import { test, expect } from "@playwright/test";

test("login flow works", async ({ page }) => {
  await page.goto("/");

  // Prefer accessibility selectors. They survive markup refactors and
  // work the same across React / Vue / Svelte renders.
  await page.getByRole("link", { name: /entrar|login|sign in/i }).click();
  await page.getByLabel(/email/i).fill("test@example.com");
  await page.getByLabel(/senha|password/i).fill("hunter2");
  await page.getByRole("button", { name: /entrar|submit/i }).click();

  // Auto-wait: Playwright retries the assertion until the timeout.
  // No manual sleeps, no flaky setTimeout.
  await expect(page).toHaveURL(/\/dashboard/);
  await expect(page.getByRole("heading", { level: 1 })).toBeVisible();
});
```

**Selector priority** (most stable 迁移到 least):

1. `getByRole` + accessible name
2. `getByLabel` (forms)
3. `getByTestId` (`da`da`a-testid`tribute the team controls)
4. `getByText` (fragile if 复制 changes)
5. CSS / XPath (last resort)

**Things 迁移到 assert** in a typical SPA flow:

- The URL changed 迁移到 the expected route
- The expected `<h1>` / page title is visible
- Network call returned 2xx — use `page.waitForResponse(/\/api\/.../)`
- Toast / 错误 region is **not** visible (or shows the right message)
- Local/session storage has the token (`page.evaluate(() => localStorage.getItem("token"))`)

### Running the suite

```bash
# Headed (you watch it run, debugging)
npx playwright test --headed

# Full speed via Obscura
npm run e2e:fast

# Only one spec
npm run e2e:fast -- e2e/login.spec.ts

# Open the HTML report after a run
npx playwright show-report
```

### Reading the report when something fails

Playwright drops three artifacts on failure (configured above):

- `playwright-report/index.html` — interactive UI
- `test-results/<spec>/trace.zip` — open with `npx playwright 显示-`n`npx playwright 显示-`race`
- `test-results/<spec>/video.webm` — full recording of the failing 运行
- `test-results/<spec>/test-failed-1.png` — screenshot

The agent 应该 always open the trace before guessing the fix. Most
"flaky" failures are real timing bugs in the app code visible in the
trace timeline.

### Common pitfalls in React/SPA E2E

| Symptom | Likely Cause | Fix |
|---|---|---|
| 测试 passes locally, fails in CI | Race with hydration | Use `await page.waitForLoadState("networkidle")` after `goto` |`goto``goto``goto`
| Click hits wrong element | Same role appears twice (header + sidebar) | Scope with `page.getByRole("main").getByRole(...)` |
| Form submit silently no-ops | RHF / Zod async validation | Use `page.getByRole("button").click()` then `expect(toast).toBeVisib`expect(toast).toBeVisib`e()`iately |
| `localStorage` is empty in next 测试 | Each 测试 gets a fresh context | Use `tes`tes`.beforeEach`seed, or Playwright fixtures |
| Auth cookies dropped between tests | Cross-context isolation | Save state with `page.context().storageState()` and reuse via `测试.use({ storageSt`t`测试.use({ storageSt`te: ... })`
| Long animations slow the suite | Framer Motion / CSS transitions | Inject `* { transition: none !important; animation: none !important }` via `page.addStyleTag` |`page.addStyleTag``page.addStyleTag``page.addStyleTag`

### CI 提示

In GitHub Actions (or any CI), 添加 Obscura as a 下载 step. The
binary is small, the 安装 is fast, and you 停止 paying the
~300 MB Chromium 下载 per job:

```yaml
- name: Install Obscura
  run: |
    curl -L -o /tmp/obscura.tar.gz \
      https://github.com/h4ckf0r0day/obscura/releases/latest/download/obscura-x86_64-linux.tar.gz
    tar xzf /tmp/obscura.tar.gz -C /usr/local/bin/
- name: Run E2E with Obscura
  run: npm run e2e:fast
```

### What the agent 必须 NOT do

- Do not 运行 `npx playwright install` if Obscura is going 迁移到 drive the
  tests — that downloads ~300 MB of Chromium for nothing.
- Do not 修改 existing E2E specs 迁移到 make them pass; if a spec fails,
  the bug is in the app code 95% of the time.
- Do not hard-code `setTimeout` waits in tests. Use `e`e`pect(...).toPass()`
  or built-in auto-waiting locators.
- Do not commit `.env` files with real credentials for E2E. Use
  `process.env.E2E_USER` / `E2E_PASS` a`E2E_PASS`t `E2E_PASS`env.exampl`.env.exampl``
- Do not 启动 `obscura serve` and forget 迁移到 kill it. Always use the
  `trap` pattern in the helper script.

### Decision shortcut for the agent

When the user says "rode os testes" or "valide o fluxo" on a frontend
project, the agent 应该:

1. Detect Playwright (`grep -q '"@playwright/test"' package.json`).
2. Detect Obscura (`command -v obscura`).
3. If both present → `npm run e2e:fast` (or equivalent).
4. If only Playwright → `npm run e2e` and recommend installing Obscura.
5. If neither → ask the user before scaffolding.

## Stealth Mode

构建 with `--features stealth` (or use the stealth release binary) and 运行
with `--stealth`.

功能:

- Per-session fingerprint randomization (GPU, screen, canvas, audio, battery)
- Realistic `navigator.userAgentData` (Chrome 145, high-entropy values)
- `event.isTrusted = true` for dispatched events
- Hidden internal properties (`Object.keys(window)` is safe)
- Native function masking (`Function.prototype.toString()` 返回 `[native code]`)`[nat`[native code]``[native code]`
- `navigator.webdriver = undefined` (matches real Chrome)
- Blocks 3,520 tracker / analytics / fingerprinting domains

## CLI 参考 Cheat Sheet

### `obscura serve`

| Flag | Default | 描述 |
|------|---------|-------------|
| `--port`_CODE_1__2`2` | WebSocket port |
| `--proxy` | — | HTTP/SOCKS5 proxy URL |
| `--stealth` | off | Anti-detection + tracker blocking |
| `--workers` | ```1` Parallel worker processes |
| `--obey-robots` | off | Respect robots.txt |

### `obscura fetch <URL>`

| Flag | Default | 描述 |
|------|---------|-------------|
| `--dump`_CODE_1__l`_CO__CODE`links`_CODE_4__inks`xt` \| `links` |
| `--eval` | — | JS expression 迁移到 evaluate |
| `--wait-until` | `loa`loa`_CODE_2__`domcontentloaded`ded`_CODE_4__`net`networkidle0`
| `--selector` | — | Wait for CSS selector |
| `--stealth` | off | Anti-detection mode |
| `--quiet` | off | Suppress banner |

### `obscura scrape <URL...>`

| Flag | Default | 描述 |
|------|---------|-------------|
| `--concurrency` | `10` __COD`10`_rallel workers |
| `--eval` | — | JS expression per page |
| `--format` | `json`_CODE`text`CODE_3__| `text` |

## CDP Coverage

Obscura implements the Chrome DevTools Protocol surface needed by
Puppeteer / Playwright:

| Domain | Methods |
|--------|---------|
| Target | createTarget, closeTarget, attachToTarget, createBrowserContext, disposeBrowserContext |
| Page | navigate, getFrameTree, addScriptToEvaluateOnNewDocument, lifecycleEvents |
| Runtime | evaluate, callFunctionOn, getProperties, addBinding |
| DOM | getDocument, querySelector, querySelectorAll, getOuterHTML, resolveNode |
| Network | 启用, setCookies, getCookies, setExtraHTTPHeaders, setUserAgentOverride |
| Fetch | 启用, continueRequest, fulfillRequest, failRequest |
| Storage | getCookies, setCookies, deleteCookies |
| 输入 | dispatchMouseEvent, dispatchKeyEvent |
| LP | getMarkdown (DOM-迁移到-Markdown) |

## Decision Heuristics

When the user asks for web automation, choose this way:

1. **One page, one shot** → `obscura fetch <url> --eval "..."`
2. **Many pages, same selector** → `obscura scrape <urls> --concurrency 25`
3. **Stateful flow, login, multi-step** → `obscura serve` + Puppeteer/Playwright
4. **Page detects bots** → 添加 `--stealth`
5. **Behind a proxy** → `--proxy <url>`
6. **CI / Docker** → use the static Linux binary, no Chrome needed

## Anti-Patterns

- Do **not** use Obscura against sites whose terms of service forbid scraping.
- Do **not** 禁用 `--obey-robots` on third-party sites in production
  pipelines without consent.
- Do **not** treat stealth mode as a bypass for paywalls or auth — it only
  hides the fact that the browser is automated, not the fact that requests
  are made.
- Do **not** spawn `obscura fetch` in a tight shell loop for many URLs — use
  `obscura scrape` (worker pool) instead.

## 故障排除

| Symptom | Likely Cause | Fix |
|---|---|---|
| `connection refused` from Puppeteer | Server not running | `obscura s`obscura s`rve --port 9222`
| Page renders empty HTML | JS hasn't finished | 添加 `--wait-until networkidle0` |
| Site detects automation | webdriver leak | 构建 with `--features stealth`, 运行 with `--stealth`--stealth`
| 构建 fails on `v8` | Ru`rustup update stable`table`table` |
| Slow first 构建 | V8 compiling | Expected ~5 min, cached after |

## 参考

- Repository: https://github.com/h4ckf0r0day/obscura
- Releases (binaries): https://github.com/h4ckf0r0day/obscura/releases
- Chrome DevTools Protocol: https://chromedevtools.github.io/devtools-protocol/
- Puppeteer: https://pptr.dev/
- Playwright: https://playwright.dev/