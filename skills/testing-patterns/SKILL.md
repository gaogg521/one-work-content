---
name: testing-patterns
model: standard
category: testing
description: 单元测试、集成测试与端到端测试模式，含框架特定指导。适用于编写测试、添加覆盖率、制定测试策略、测试函数、创建测试套件、修复不稳定测试或提升测试质量等场景。
version: 1.0
tags:
- 模式
- 测试
---

# Testing Patterns

> **编写能发现 bug 的测试，而不是仅仅通过的测试。** —— 通过覆盖率获得信心，通过隔离获得速度。

---

## 测试金字塔

| 层级 | 比例 | 速度 | 成本 | 信心 | 范围 |
|-------|-------|-------|------|------------|-------|
| **Unit** | ~70% | 毫秒 | 低 | 低（隔离） | 单个函数/类 |
| **Integration** | ~20% | 秒 | 中 | 中 | 模块边界、API、数据库 |
| **E2E** | ~10% | 分钟 | 高 | 高（真实） | 完整用户工作流 |

> **规则：** 如果你的端到端测试数量超过单元测试，翻转金字塔。

---

## 单元测试模式

### 核心模式

| 模式 | 使用时机 | 结构 |
|---------|------------|-----------|
| **Arrange-Act-Assert** | 所有单元测试的默认结构 | 设置、执行、验证 |
| **Given-When-Then** | BDD 风格，聚焦行为 | 前置条件、动作、结果 |
| **Parameterized** | 相同逻辑，多个输入 | 数据驱动测试用例 |
| **Snapshot** | UI 组件、序列化输出 | 与保存的基线对比 |
| **Property-Based** | 数学不变量 | 生成随机输入，断言属性 |

### Arrange-Act-Assert (AAA)

每个单元测试的默认结构。设置、执行和验证的清晰分离使测试可读且可维护。

```typescript
// 清晰的 AAA 结构
test('calculates order total with tax', () => {
  // Arrange
  const items = [{ price: 10, qty: 2 }, { price: 5, qty: 1 }];
  const taxRate = 0.08;

  // Act
  const total = calculateTotal(items, taxRate);

  // Assert
  expect(total).toBe(27.0);
});
```

### 测试替身

为不同情况使用正确类型的测试替身。每种都有不同的用途。

| 替身 | 用途 | 使用时机 | 示例 |
|--------|---------|-------------|---------|
| **Stub** | 返回固定数据 | 控制间接输入 | `jest.fn().mockReturnValue(42)` |
| **Mock** | 验证交互 | 断言某物被调用 | `expect(mock).toHaveBeenCalledWith('arg')` |
| **Spy** | 包装真实实现 | 观察而不替换 | `jest.spyOn(service, 'save')` |
| **Fake** | 工作的简化实现 | 需要真实行为 | 内存数据库、假 HTTP 服务器 |

```typescript
// Stub —— 控制间接输入
const getUser = jest.fn().mockResolvedValue({ id: 1, name: 'Alice' });

// Spy —— 观察而不替换
const spy = jest.spyOn(logger, 'warn');
processInvalidInput(data);
expect(spy).toHaveBeenCalledWith('Invalid input received');

// Fake —— 轻量级替代
class FakeUserRepo implements UserRepository {
  private users = new Map<string, User>();
  async save(user: User) { this.users.set(user.id, user); }
  async findById(id: string) { return this.users.get(id) ?? null; }
}
```

### 参数化测试

当相同逻辑需要多个输入验证时使用参数化测试。这消除了复制粘贴测试，同时提供全面覆盖。

```typescript
// Vitest/Jest
test.each([
  ['hello', 'HELLO'],
  ['world', 'WORLD'],
  ['', ''],
  ['123abc', '123ABC'],
])('toUpperCase(%s) returns %s', (input, expected) => {
  expect(input.toUpperCase()).toBe(expected);
});
```

```python
# pytest
@pytest.mark.parametrize("input,expected", [
    ("hello", "HELLO"),
    ("world", "WORLD"),
    ("", ""),
])
def test_to_upper(input, expected):
    assert input.upper() == expected
```

```go
// Go —— 表驱动测试（惯用法）
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 2, 3, 5},
        {"zero", 0, 0, 0},
        {"negative", -1, -2, -3},
    }
    for _, tc := range tests {
        t.Run(tc.name, func(t *testing.T) {
            if got := Add(tc.a, tc.b); got != tc.expected {
                t.Errorf("Add(%d,%d) = %d, want %d", tc.a, tc.b, got, tc.expected)
            }
        })
    }
}
```

---

## 集成测试模式

### 数据库测试策略

| 策略 | 方法 | 权衡 |
|----------|----------|-----------|
| **Transaction rollback** | 将每个测试包装在事务中，之后回滚 | 快速，但隐藏提交 bug |
| **Fixtures/seeds** | 在套件之前加载已知数据 | 可预测，但 schema 变更时脆弱 |
| **Factory functions** | 以编程方式生成数据 | 灵活，但更多设置代码 |
| **Testcontainers** | 在 Docker 中启动真实数据库 | 真实，但启动更慢 |

```typescript
// 事务回滚模式 (Prisma)
beforeEach(async () => {
  await prisma.$executeRaw`BEGIN`;
});
afterEach(async () => {
  await prisma.$executeRaw`ROLLBACK`;
});

test('creates user in database', async () => {
  const user = await createUser({ name: 'Alice', email: 'a@b.com' });
  const found = await prisma.user.findUnique({ where: { id: user.id } });
  expect(found?.name).toBe('Alice');
});
```

### API 测试

```typescript
// Supertest (Node.js)
import request from 'supertest';
import { app } from '../src/app';

describe('POST /api/users', () => {
  it('creates a user and returns 201', async () => {
    const res = await request(app)
      .post('/api/users')
      .send({ name: 'Alice', email: 'alice@test.com' })
      .expect(201);

    expect(res.body).toMatchObject({
      id: expect.any(String),
      name: 'Alice',
    });
  });

  it('returns 400 for invalid email', async () => {
    await request(app)
      .post('/api/users')
      .send({ name: 'Alice', email: 'not-an-email' })
      .expect(400);
  });
});
```

---

## Mock 最佳实践

### Mock 边界，而不是实现

基本规则：在系统边界（外部 API、数据库、文件系统）进行 mock，永远不要 mock 内部领域逻辑。

```typescript
// BAD —— mock 内部实现
jest.mock('./utils/formatDate');  // 重构时中断

// GOOD —— mock 外部边界
jest.mock('./services/paymentGateway');  // 第三方 API 是边界
```

### 何时 Mock 与何时不 Mock

| Mock | 不要 Mock |
|------|-----------|
| HTTP API、外部服务 | 纯函数 |
| 数据库（单元测试中） | 你自己的领域逻辑 |
| 文件系统、网络 | 数据转换 |
| 时间/日期 (`Date.now`) | 简单计算 |
| 环境变量 | 内部类方法 |

### 为可测试性进行依赖注入

构建代码以便依赖可以在测试中替换。这是对可测试代码影响最大的单一模式。

```typescript
// 可注入依赖 —— 易于测试
class OrderService {
  constructor(
    private paymentGateway: PaymentGateway,
    private inventory: InventoryService,
    private notifier: NotificationService,
  ) {}

  async placeOrder(order: Order): Promise<OrderResult> {
    const stock = await this.inventory.check(order.items);
    if (!stock.available) return { status: 'out_of_stock' };

    const payment = await this.paymentGateway.charge(order.total);
    if (!payment.success) return { status: 'payment_failed' };

    await this.notifier.send(order.userId, 'Order confirmed');
    return { status: 'confirmed', id: payment.transactionId };
  }
}

// 在测试中 —— 注入 fakes
const service = new OrderService(
  new FakePaymentGateway(),
  new FakeInventory({ available: true }),
  new FakeNotifier(),
);
```

---

## 框架快速参考

| 框架 | 语言 | 类型 | 测试运行器 | 断言 |
|-----------|----------|------|-------------|-----------|
| **Jest** | JS/TS | 单元/集成 | 内置 | `expect()` |
| **Vitest** | JS/TS | 单元/集成 | Vite-native | `expect()` (Jest-compatible) |
| **Playwright** | JS/TS/Python | E2E | 内置 | `expect()` / locators |
| **Cypress** | JS/TS | E2E | 内置 | `cy.should()` |
| **pytest** | Python | 单元/集成 | 内置 | `assert` |
| **Go testing** | Go | 单元/集成 | `go test` | `t.Error()` / testify |
| **Rust** | Rust | 单元/集成 | `cargo test` | `assert!()` / `assert_eq!()` |
| **JUnit 5** | Java/Kotlin | 单元/集成 | 内置 | `assertEquals()` |
| **RSpec** | Ruby | 单元/集成 | 内置 | `expect().to` |
| **PHPUnit** | PHP | 单元/集成 | 内置 | `$this->assert*()` |
| **xUnit** | C# | 单元/集成 | 内置 | `Assert.Equal()` |

---

## 测试质量检查清单

| 质量 | 规则 | 原因 |
|---------|------|-----|
| **Deterministic** | 相同输入每次产生相同结果 | 不稳定测试侵蚀信任 |
| **Isolated** | 测试之间没有共享可变状态 | 依赖顺序的测试在 CI 中中断 |
| **Fast** | 单元: < 10ms, 集成: < 1s, E2E: < 30s | 慢测试不会被运行 |
| **Readable** | 测试名称描述场景和预期 | 测试即文档 |
| **Maintainable** | 改变一个行为，改变一个测试 | 脆弱测试拖慢开发 |
| **Focused** | 每个测试一个逻辑断言 | 失败精确定位问题 |

> **命名约定：** `test_[unit]_[scenario]_[expected result]` 或 `should [do X] when [condition Y]`

---

## 覆盖率策略

### 何时追求何种目标

| 目标 | 时机 | 理由 |
|--------|------|-----------|
| **80%+ line coverage** | 业务逻辑、工具、核心领域 | 高 ROI —— 捕获大多数回归 |
| **90%+ branch coverage** | 支付处理、认证、安全关键路径 | 边缘情况在这里很重要 |
| **100% coverage** | 几乎从不 —— 收益递减 | Getter/setter 测试增加噪音，而非信心 |
| **Mutation testing** | 覆盖率高后的关键路径 | 验证测试确实能发现 bug |

### 什么不应该测试

| 跳过 | 原因 |
|------|--------|
| 生成代码 (Prisma client, protobuf) | 由工具维护 |
| 第三方库内部 | 不是你的责任 |
| 简单 getter/setter | 没有逻辑可验证 |
| 配置文件 | 改为测试它们配置的行为 |
| Console.log / print 语句 | 没有业务价值的副作用 |

---

## 测试组织

```
src/
├── services/
│   ├── order.service.ts
│   └── order.service.test.ts      # 同位置单元测试
├── api/
│   └── routes/
│       └── orders.ts
tests/
├── integration/
│   ├── api/
│   │   └── orders.test.ts         # API 集成测试
│   └── db/
│       └── order.repo.test.ts     # 数据库集成测试
├── e2e/
│   ├── pages/                     # 页面对象
│   │   └── checkout.page.ts
│   └── specs/
│       └── checkout.spec.ts       # 端到端规格
└── helpers/
    ├── factories.ts               # 测试数据工厂
    └── setup.ts                   # 全局测试设置
```

> **规则：** 单元测试与源码同位置。集成和端到端测试分离到专用目录。

---

## 反模式

| 反模式 | 问题 | 修复 |
|--------------|---------|-----|
| **Testing implementation** | 测试在重构时中断，而不是在出现 bug 时 | 测试行为和输出，而不是内部 |
| **Flaky tests** | 非确定性失败侵蚀 CI 信任 | 移除时间/顺序/网络依赖 |
| **Test pollution** | 共享可变状态在测试间泄漏 | 在 `beforeEach` / `setUp` 中重置状态 |
| **Sleeping in tests** | `sleep(2000)` 慢且不可靠 | 使用显式等待、轮询或事件 |
| **Giant arrange** | 50 行设置掩盖意图 | 提取工厂/构建器/fixtures |
| **Assert-free tests** | 测试运行但不验证任何内容 | 每个测试必须 assert 或 expect |
| **Overmocking** | Mock 一切等于没有测试真实内容 | 只 mock 外部边界 |
| **Copy-paste tests** | 重复测试发散和腐烂 | 使用参数化测试或辅助函数 |
| **Testing the framework** | 验证库代码是否工作 | 测试*你的*逻辑，信任依赖 |
| **Ignoring test failures** | `skip`, `xit`, `@Disabled` 累积 | 修复或删除 —— 永远不要囤积跳过的测试 |
| **Tight coupling to DB** | schema 变更时测试失败 | 单元测试使用仓库模式 + fakes |
| **One giant test** | 单个测试覆盖 10 个场景 | 拆分为聚焦的、命名的测试 |
| **No test for bug fix** | 回归稍后重新出现 | 每个 bug 修复都要有一个回归测试 |

---

## 永远不要做

1. **永远不要测试实现细节而不是行为** —— 测试必须验证代码做什么，而不是怎么做
2. **永远不要在测试中使用 `sleep()`** —— 使用显式等待、轮询、事件或自动重试的断言
3. **永远不要共享测试间的可变状态** —— 每个测试设置和拆除自己的状态
4. **永远不要编写无断言的测试** —— 没有断言的测试什么都不能证明
5. **永远不要 mock 内部领域逻辑** —— 只在系统边界 mock（网络、数据库、文件系统、时钟）
6. **永远不要在没有关联问题和重新启用计划的情况下跳过测试** —— 跳过的测试腐烂成永久缺口
7. **永远不要让测试套件处于失败状态** —— 在继续之前修复它或删除它并说明理由
8. **永远不要以 100% 覆盖率为目标** —— 覆盖率百分比是工具，不是目标；关键路径上的强断言胜过到处弱断言

---

## 总结

| 做 | 不做 |
|----|-------|
| 测试行为，而不是实现 | 到处 mock |
| 在修复 bug 之前编写测试 | 跳过测试以更快发布 |
| 保持测试快速和确定性 | 使用 `sleep()` 或共享状态 |
| 使用工厂生成测试数据 | 跨测试复制粘贴设置 |
| 在系统边界 mock | Mock 内部函数 |
| 用描述性名称命名测试 | 将测试命名为 `test1`, `test2` |
| 每次推送都在 CI 中运行测试 | 只在本地运行测试 |
| 删除或修复跳过的测试 | 让 `@skip` 永远累积 |
| 对变体使用参数化测试 | 重复测试代码 |
| 注入依赖以获得可测试性 | 硬编码依赖 |

> **记住：** 测试是安全网 —— 一个快速、可信的套件让你可以无畏地重构并充满信心地发布。
