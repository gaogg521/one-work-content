---
name: e2e-testing-patterns
model: standard
category: testing
description: 使用 Playwright 和 Cypress 构建可靠、快速的 E2E 测试套件。关键用户旅程覆盖、消除不稳定测试、CI/CD 集成。
version: 1.0
keywords:
- e2e
- end-to-end
- playwright
- cypress
- browser testing
- integration tests
- test automation
- flaky tests
- visual regression
tags:
- 测试
- ArgoCD
- CI/CD
- 模式
---

# E2E 测试模式

> 测试用户做什么，而不是代码如何工作。E2E 测试证明系统作为一个整体是有效的 — 它们是你发布信心的来源。


## 安装

### OpenClaw / Moltbot / Clawbot

```bash
npx clawhub@latest install e2e-testing-patterns
```


---

## 此技能的功能

提供用于构建端到端测试套件的模式，这些模式：
- 在用户之前捕获回归
- 对于 CI/CD 运行速度足够快
- 保持稳定（没有不稳定的失败）
- 覆盖关键用户旅程而不会过度测试

## 何时使用

- **为 Web 应用程序实现 E2E 测试自动化**
- **调试间歇性失败的不稳定测试**
- **使用浏览器测试设置 CI/CD 测试管道**
- **测试关键用户工作流**（认证、结账、注册）
- **选择使用 E2E 测试什么** 与单元/集成测试

---

## 测试金字塔 — 了解你的层

```
        /\
       /E2E\         ← 少量：仅关键路径（此技能）
      /─────\
     /Integr\        ← 更多：组件交互、API 契约
    /────────\
   /Unit Tests\      ← 大量：快速、隔离、覆盖边缘情况
  /────────────\
```

### E2E 测试的用途

| E2E 测试 ✓ | 不是 E2E 测试 ✗ |
|-------------|-----------------|
| 关键用户旅程（登录 → 仪表板 → 操作 → 登出） | 单元级逻辑（使用单元测试） |
| 多步骤流程（结账、入门向导） | API 契约（使用集成测试） |
| 跨浏览器兼容性 | 边缘情况（太慢，使用单元测试） |
| 真实 API 集成 | 内部实现细节 |
| 认证流程 | 组件视觉状态（使用 Storybook） |

**经验法则：** 如果它破坏了会摧毁你的业务，请使用 E2E 测试。如果只是不方便，请使用更快的单元/集成测试。

---

## 核心原则

| 原则 | 为什么 | 如何 |
|-----------|-----|-----|
| **测试行为，而非实现** | 在重构中幸存 | 断言用户可见的结果，而非 DOM 结构 |
| **独立的测试** | 可并行化、可调试 | 每个测试创建自己的数据，之后清理 |
| **确定性等待** | 没有不稳定性 | 等待条件，而非固定的超时 |
| **稳定的选择器** | 在 UI 更改中幸存 | 使用 `data-testid`、角色、标签 — 从不使用 CSS 类 |
| **快速反馈** | 开发人员运行它们 | 模拟外部服务、并行化、分片 |

---

## Playwright 模式

### 配置

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  timeout: 30000,
  expect: { timeout: 5000 },
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [["html"], ["junit", { outputFile: "results.xml" }]],
  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
    video: "retain-on-failure",
  },
  projects: [
    { name: "chromium", use: { ...devices["Desktop Chrome"] } },
    { name: "firefox", use: { ...devices["Desktop Firefox"] } },
    { name: "webkit", use: { ...devices["Desktop Safari"] } },
    { name: "mobile", use: { ...devices["iPhone 13"] } },
  ],
});
```

### 模式：页面对象模型

封装页面逻辑。测试读起来像用户故事。

```typescript
// pages/LoginPage.ts
import { Page, Locator } from "@playwright/test";

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel("Email");
    this.passwordInput = page.getByLabel("Password");
    this.loginButton = page.getByRole("button", { name: "Login" });
    this.errorMessage = page.getByRole("alert");
  }

  async goto() {
    await this.page.goto("/login");
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }
}

// tests/login.spec.ts
import { test, expect } from "@playwright/test";
import { LoginPage } from "../pages/LoginPage";

test("successful login redirects to dashboard", async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login("user@example.com", "password123");

  await expect(page).toHaveURL("/dashboard");
  await expect(page.getByRole("heading", { name: "Dashboard" })).toBeVisible();
});
```

### 模式：用于测试数据的 Fixtures

自动创建和清理测试数据。

```typescript
// fixtures/test-data.ts
import { test as base } from "@playwright/test";

export const test = base.extend<{ testUser: TestUser }>({
  testUser: async ({}, use) => {
    // Setup: Create user
    const user = await createTestUser({
      email: `test-${Date.now()}@example.com`,
      password: "Test123!@#",
    });

    await use(user);

    // Teardown: Clean up
    await deleteTestUser(user.id);
  },
});

// Usage — testUser is created before, deleted after
test("user can update profile", async ({ page, testUser }) => {
  await page.goto("/login");
  await page.getByLabel("Email").fill(testUser.email);
  // ...
});
```

### 模式：智能等待

从不使用固定的超时。等待特定条件。

```typescript
// ❌ FLAKY: Fixed timeout
await page.waitForTimeout(3000);

// ✅ STABLE: Wait for conditions
await page.waitForLoadState("networkidle");
await page.waitForURL("/dashboard");

// ✅ BEST: Auto-waiting assertions
await expect(page.getByText("Welcome")).toBeVisible();
await expect(page.getByRole("button", { name: "Submit" })).toBeEnabled();

// Wait for API response
const responsePromise = page.waitForResponse(
  (r) => r.url().includes("/api/users") && r.status() === 200
);
await page.getByRole("button", { name: "Load" }).click();
await responsePromise;
```

### 模式：网络模拟

将测试与真实外部服务隔离。

```typescript
test("shows error when API fails", async ({ page }) => {
  // Mock the API response
  await page.route("**/api/users", (route) => {
    route.fulfill({
      status: 500,
      body: JSON.stringify({ error: "Server Error" }),
    });
  });

  await page.goto("/users");
  await expect(page.getByText("Failed to load users")).toBeVisible();
});

test("handles slow network gracefully", async ({ page }) => {
  await page.route("**/api/data", async (route) => {
    await new Promise((r) => setTimeout(r, 3000)); // Simulate delay
    await route.continue();
  });

  await page.goto("/dashboard");
  await expect(page.getByText("Loading...")).toBeVisible();
});
```

---

## Cypress 模式

### 自定义命令

```typescript
// cypress/support/commands.ts
declare global {
  namespace Cypress {
    interface Chainable {
      login(email: string, password: string): Chainable<void>;
      dataCy(value: string): Chainable<JQuery<HTMLElement>>;
    }
  }
}

Cypress.Commands.add("login", (email, password) => {
  cy.visit("/login");
  cy.get('[data-testid="email"]').type(email);
  cy.get('[data-testid="password"]').type(password);
  cy.get('[data-testid="login-button"]').click();
  cy.url().should("include", "/dashboard");
});

Cypress.Commands.add("dataCy", (value) => {
  return cy.get(`[data-cy="${value}"]`);
});

// Usage
cy.login("user@example.com", "password");
cy.dataCy("submit-button").click();
```

### 网络拦截

```typescript
// Mock API
cy.intercept("GET", "/api/users", {
  statusCode: 200,
  body: [{ id: 1, name: "John" }],
}).as("getUsers");

cy.visit("/users");
cy.wait("@getUsers");
cy.get('[data-testid="user-list"]').children().should("have.length", 1);
```

---

## 选择器策略

| 优先级 | 选择器类型 | 示例 | 为什么 |
|----------|--------------|---------|-----|
| 1 | **Role + name** | `getByRole("button", { name: "Submit" })` | 可访问、面向用户 |
| 2 | **Label** | `getByLabel("Email address")` | 可访问、语义化 |
| 3 | **data-testid** | `getByTestId("checkout-form")` | 稳定、为测试显式设置 |
| 4 | **Text content** | `getByText("Welcome back")` | 面向用户 |
| ❌ | CSS classes | `.btn-primary` | 样式更改时中断 |
| ❌ | DOM structure | `div > form > input:nth-child(2)` | 任何重组时中断 |

```typescript
// ❌ BAD: Brittle selectors
cy.get(".btn.btn-primary.submit-button").click();
cy.get("div > form > div:nth-child(2) > input").type("text");

// ✅ GOOD: Stable selectors
page.getByRole("button", { name: "Submit" }).click();
page.getByLabel("Email address").fill("user@example.com");
page.getByTestId("email-input").fill("user@example.com");
```

---

## 视觉回归测试

```typescript
// Playwright visual comparisons
test("homepage looks correct", async ({ page }) => {
  await page.goto("/");
  await expect(page).toHaveScreenshot("homepage.png", {
    fullPage: true,
    maxDiffPixels: 100,
  });
});

test("button states", async ({ page }) => {
  const button = page.getByRole("button", { name: "Submit" });

  await expect(button).toHaveScreenshot("button-default.png");

  await button.hover();
  await expect(button).toHaveScreenshot("button-hover.png");
});
```

---

## 可访问性测试

```typescript
// npm install @axe-core/playwright
import AxeBuilder from "@axe-core/playwright";

test("page has no accessibility violations", async ({ page }) => {
  await page.goto("/");

  const results = await new AxeBuilder({ page })
    .exclude("#third-party-widget")  // Exclude things you can't control
    .analyze();

  expect(results.violations).toEqual([]);
});
```

---

## 调试失败的测试

```bash
# Run in headed mode (see the browser)
npx playwright test --headed

# Debug mode (step through)
npx playwright test --debug

# Show trace viewer for failed tests
npx playwright show-report
```

```typescript
// Add test steps for better failure reports
test("checkout flow", async ({ page }) => {
  await test.step("Add item to cart", async () => {
    await page.goto("/products");
    await page.getByRole("button", { name: "Add to Cart" }).click();
  });

  await test.step("Complete checkout", async () => {
    await page.goto("/checkout");
    // ... if this fails, you know which step
  });

  // Pause for manual inspection
  await page.pause();
});
```

---

## 不稳定测试检查清单

当测试间歇性失败时，请检查：

| 问题 | 修复 |
|-------|-----|
| 固定的 `waitForTimeout()` 调用 | 替换为 `waitForSelector()` 或 expect 断言 |
| 页面加载的竞争条件 | 等待 `networkidle` 或特定元素 |
| 测试数据污染 | 确保测试创建/清理自己的数据 |
| 动画计时 | 等待动画完成或禁用它们 |
| 视口不一致 | 在配置中设置显式视口 |
| 随机测试顺序问题 | 测试必须是独立的 |
| 第三方服务不稳定 | 模拟外部 API |

---

## CI/CD 集成

```yaml
# GitHub Actions example
name: E2E Tests
on: [push, pull_request]

jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run build
      - run: npm run start & npx wait-on http://localhost:3000
      - run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

---

## 永远不要做

1. **永远不要使用固定的 `waitForTimeout()` 或 `cy.wait(ms)`** — 它们会导致不稳定的测试并降低套件速度
2. **永远不要依赖 CSS 类或 DOM 结构作为选择器** — 使用角色、标签或 data-testid
3. **永远不要共享测试之间的状态** — 每个测试必须是完全独立的
4. **永远不要测试实现细节** — 测试用户看到和做什么，而非内部结构
5. **永远不要跳过清理** — 始终删除你创建的测试数据，即使在失败时也是如此
6. **永远不要使用 E2E 测试所有内容** — 保留给关键路径；对边缘情况使用更快的测试
7. **永远不要忽略不稳定的测试** — 立即修复它们或删除它们；不稳定的测试比没有测试更糟糕
8. **永远不要在选择器中硬编码测试数据** — 对变化的内容使用动态等待

---

## 快速参考

### Playwright 命令

```typescript
// Navigation
await page.goto("/path");
await page.goBack();
await page.reload();

// Interactions
await page.click("selector");
await page.fill("selector", "text");
await page.type("selector", "text");  // Types character by character
await page.selectOption("select", "value");
await page.check("checkbox");

// Assertions
await expect(page).toHaveURL("/expected");
await expect(locator).toBeVisible();
await expect(locator).toHaveText("expected");
await expect(locator).toBeEnabled();
await expect(locator).toHaveCount(3);
```

### Cypress 命令

```typescript
// Navigation
cy.visit("/path");
cy.go("back");
cy.reload();

// Interactions
cy.get("selector").click();
cy.get("selector").type("text");
cy.get("selector").clear().type("text");
cy.get("select").select("value");
cy.get("checkbox").check();

// Assertions
cy.url().should("include", "/expected");
cy.get("selector").should("be.visible");
cy.get("selector").should("have.text", "expected");
cy.get("selector").should("have.length", 3);
```
