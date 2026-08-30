---
name: test-specialist
description: 编写测试用例、修复缺陷(bug)、分析代码潜在问题或提升 JavaScript/TypeScript 应用测试覆盖率时调用。适用于单元测试、集成测试、端到端测试、调试运行时错误、逻辑缺陷、性能问题与系统化代码分析。
tags:
- Java
---

# 测试专家

## 概述

对 JavaScript/TypeScript 应用应用系统化测试方法和调试技术。此技能提供全面的测试策略、bug 分析框架以及用于识别覆盖率缺口和未测试代码的自动化工具。

## 核心能力

### 1. 编写测试用例

编写覆盖单元、集成和端到端场景的全面测试。

#### 单元测试方法

使用 AAA 模式（Arrange-Act-Assert）构建测试：

```typescript
describe('ExpenseCalculator', () => {
  describe('calculateTotal', () => {
    test('sums expense amounts correctly', () => {
      // Arrange
      const expenses = [
        { amount: 100, category: 'food' },
        { amount: 50, category: 'transport' },
        { amount: 25, category: 'entertainment' }
      ];

      // Act
      const total = calculateTotal(expenses);

      // Assert
      expect(total).toBe(175);
    });

    test('handles empty expense list', () => {
      expect(calculateTotal([])).toBe(0);
    });

    test('handles negative amounts', () => {
      const expenses = [
        { amount: 100, category: 'food' },
        { amount: -50, category: 'refund' }
      ];
      expect(calculateTotal(expenses)).toBe(50);
    });
  });
});
```

**关键原则：**
- 每个测试只测试一个行为
- 覆盖正常路径、边界情况和错误条件
- 使用描述性的测试名称来解释场景
- 保持测试独立和隔离

#### 集成测试方法

测试组件如何协同工作，包括数据库、API 和服务交互：

```typescript
describe('ExpenseAPI Integration', () => {
  beforeAll(async () => {
    await database.connect(TEST_DB_URL);
  });

  afterAll(async () => {
    await database.disconnect();
  });

  beforeEach(async () => {
    await database.clear();
    await seedTestData();
  });

  test('POST /expenses creates expense and updates total', async () => {
    const response = await request(app)
      .post('/api/expenses')
      .send({
        amount: 50,
        category: 'food',
        description: 'Lunch'
      })
      .expect(201);

    expect(response.body).toMatchObject({
      id: expect.any(Number),
      amount: 50,
      category: 'food'
    });

    // 验证数据库状态
    const total = await getTotalExpenses();
    expect(total).toBe(50);
  });
});
```

#### 端到端测试方法

使用 Playwright 或 Cypress 等工具测试完整的用户工作流：

```typescript
test('user can track expense from start to finish', async ({ page }) => {
  // 导航到应用
  await page.goto('/');

  // 添加新支出
  await page.click('[data-testid="add-expense-btn"]');
  await page.fill('[data-testid="amount"]', '50.00');
  await page.selectOption('[data-testid="category"]', 'food');
  await page.fill('[data-testid="description"]', 'Lunch');
  await page.click('[data-testid="submit"]');

  // 验证支出出现在列表中
  await expect(page.locator('[data-testid="expense-item"]')).toContainText('Lunch');
  await expect(page.locator('[data-testid="total"]')).toContainText('$50.00');
});
```

### 2. 系统化 Bug 分析

应用结构化的调试方法来识别和修复问题。

#### 五步分析流程

1. **复现**: 可靠地复现 bug
   - 记录触发的确切步骤
   - 识别所需的环境/状态
   - 记录预期行为与实际行为

2. **隔离**: 缩小问题范围
   - 对代码路径进行二分查找
   - 创建最小复现用例
   - 移除无关的依赖

3. **根因分析**: 确定根本原因
   - 追踪执行流程
   - 检查假设和前置条件
   - 审查最近的更改（git blame）

4. **修复实现**: 实现解决方案
   - 先编写失败的测试（TDD）
   - 实现修复
   - 验证测试通过

5. **验证**: 确保完整性
   - 运行完整测试套件
   - 测试边界情况
   - 验证没有回归

#### 常见 Bug 模式

**竞态条件：**
```typescript
// 测试并发操作
test('handles concurrent updates correctly', async () => {
  const promises = Array.from({ length: 100 }, () =>
    incrementExpenseCount()
  );

  await Promise.all(promises);
  expect(getExpenseCount()).toBe(100);
});
```

**Null/Undefined 错误：**
```typescript
// 测试空安全性
test.each([null, undefined, '', 0, false])
  ('handles invalid input: %p', (input) => {
    expect(() => processExpense(input)).toThrow('Invalid expense');
  });
```

**差一错误：**
```typescript
// 显式测试边界
describe('pagination', () => {
  test('handles empty list', () => {
    expect(paginate([], 1, 10)).toEqual([]);
  });

  test('handles single item', () => {
    expect(paginate([item], 1, 10)).toEqual([item]);
  });

  test('handles last page with partial items', () => {
    const items = Array.from({ length: 25 }, (_, i) => i);
    expect(paginate(items, 3, 10)).toHaveLength(5);
  });
});
```

### 3. 识别潜在问题

在问题成为 bug 之前主动识别它们。

#### 安全漏洞

测试常见的安全问题：

```typescript
describe('security', () => {
  test('prevents SQL injection', async () => {
    const malicious = "'; DROP TABLE expenses; --";
    await expect(
      searchExpenses(malicious)
    ).resolves.not.toThrow();
  });

  test('sanitizes XSS in descriptions', () => {
    const xss = '<script>alert("xss")</script>';
    const expense = createExpense({ description: xss });
    expect(expense.description).not.toContain('<script>');
  });

  test('requires authentication for expense operations', async () => {
    await request(app)
      .post('/api/expenses')
      .send({ amount: 50 })
      .expect(401);
  });
});
```

#### 性能问题

测试性能问题：

```typescript
test('processes large expense list efficiently', () => {
  const largeList = Array.from({ length: 10000 }, (_, i) => ({
    amount: i,
    category: 'test'
  }));

  const start = performance.now();
  const total = calculateTotal(largeList);
  const duration = performance.now() - start;

  expect(duration).toBeLessThan(100); // 应在 <100ms 内完成
  expect(total).toBe(49995000);
});
```

#### 逻辑错误

使用参数化测试来捕获边界情况：

```typescript
test.each([
  // [input, expected, description]
  [[10, 20, 30], 60, 'normal positive values'],
  [[0, 0, 0], 0, 'all zeros'],
  [[-10, 20, -5], 5, 'mixed positive and negative'],
  [[0.1, 0.2], 0.3, 'decimal precision'],
  [[Number.MAX_SAFE_INTEGER], Number.MAX_SAFE_INTEGER, 'large numbers'],
])('calculateTotal(%p) = %p (%s)', (amounts, expected, description) => {
  const expenses = amounts.map(amount => ({ amount, category: 'test' }));
  expect(calculateTotal(expenses)).toBeCloseTo(expected);
});
```

### 4. 测试覆盖率分析

使用自动化工具来识别测试覆盖率中的缺口。

#### 查找未测试的代码

运行提供的脚本来识别没有测试的源文件：

```bash
python3 scripts/find_untested_code.py src
```

该脚本将：
- 扫描源目录中的所有代码文件
- 识别哪些文件缺少对应的测试文件
- 按类型（组件、服务、工具等）对未测试文件进行分类
- 优先处理最需要测试的文件

**解读：**
- **API/Services**: 高优先级 - 测试业务逻辑和数据操作
- **Models**: 高优先级 - 测试数据验证和转换
- **Hooks**: 中优先级 - 测试有状态行为
- **Components**: 中优先级 - 测试复杂的 UI 逻辑
- **Utils**: 低优先级 - 根据需要测试复杂函数

#### 分析覆盖率报告

生成覆盖率后运行覆盖率分析脚本：

```bash
# 生成覆盖率（使用 Jest 示例）
npm test -- --coverage

# 分析覆盖率缺口
python3 scripts/analyze_coverage.py coverage/coverage-final.json
```

该脚本识别：
- 低于覆盖率阈值的文件（默认 80%）
- Statement、branch 和 function 覆盖率百分比
- 需要改进的优先文件

**覆盖率目标：**
- 关键路径: 90%+ 覆盖率
- 业务逻辑: 85%+ 覆盖率
- UI 组件: 75%+ 覆盖率
- 工具函数: 70%+ 覆盖率

### 5. 测试维护和质量

确保测试保持有价值和可维护。

#### 测试代码质量原则

**DRY (Don't Repeat Yourself):**
```typescript
// 提取通用设置
function createTestExpense(overrides = {}) {
  return {
    amount: 50,
    category: 'food',
    description: 'Test expense',
    date: new Date('2024-01-01'),
    ...overrides
  };
}

test('filters by category', () => {
  const expenses = [
    createTestExpense({ category: 'food' }),
    createTestExpense({ category: 'transport' }),
  ];
  // ...
});
```

**清晰的测试数据：**
```typescript
// 差: 魔法数字
expect(calculateDiscount(100, 0.15)).toBe(85);

// 好: 命名常量
const ORIGINAL_PRICE = 100;
const DISCOUNT_RATE = 0.15;
const EXPECTED_PRICE = 85;
expect(calculateDiscount(ORIGINAL_PRICE, DISCOUNT_RATE)).toBe(EXPECTED_PRICE);
```

**避免测试相互依赖：**
```typescript
// 差: 测试依赖于执行顺序
let sharedState;
test('test 1', () => {
  sharedState = { value: 1 };
});
test('test 2', () => {
  expect(sharedState.value).toBe(1); // 依赖于 test 1
});

// 好: 独立的测试
test('test 1', () => {
  const state = { value: 1 };
  expect(state.value).toBe(1);
});
test('test 2', () => {
  const state = { value: 1 };
  expect(state.value).toBe(1);
});
```

## 工作流决策树

遵循此决策树来确定测试方法：

1. **添加新功能？**
   - 是 → 先编写测试（TDD）
     - 编写失败的测试
     - 实现功能
     - 验证测试通过
     - 重构
   - 否 → 进入步骤 2

2. **修复 bug？**
   - 是 → 应用 bug 分析流程
     - 复现 bug
     - 编写展示 bug 的失败测试
     - 修复实现
     - 验证测试通过
   - 否 → 进入步骤 3

3. **提升测试覆盖率？**
   - 是 → 使用覆盖率工具
     - 运行 `find_untested_code.py` 来识别缺口
     - 在覆盖率报告上运行 `analyze_coverage.py`
     - 优先处理关键路径
     - 为未测试的代码编写测试
   - 否 → 进入步骤 4

4. **分析代码质量？**
   - 是 → 系统化审查
     - 检查安全漏洞
     - 测试边界情况和错误处理
     - 验证性能特征
     - 审查错误处理

## 测试框架和工具

### 推荐技术栈

**单元/集成测试：**
- Jest 或 Vitest 作为测试运行器
- Testing Library 用于 React 组件
- Supertest 用于 API 测试
- MSW (Mock Service Worker) 用于 API mocking

**E2E 测试：**
- Playwright 或 Cypress
- Page Object Model 模式

**覆盖率：**
- Istanbul（内置于 Jest/Vitest）
- JSON 格式的覆盖率报告

### 运行测试

```bash
# 运行所有测试
npm test

# 带覆盖率运行
npm test -- --coverage

# 运行特定测试文件
npm test -- ExpenseCalculator.test.ts

# 在 watch 模式下运行
npm test -- --watch

# 运行 E2E 测试
npm run test:e2e
```

## 参考文档

有关详细的模式和技术，请参阅：

- `references/testing_patterns.md` - 全面的测试模式、最佳实践和代码示例
- `references/bug_analysis.md` - 深入的 bug 分析框架、常见 bug 模式和调试技术

在以下情况下加载这些参考：
- 处理复杂的测试场景
- 需要特定的模式实现
- 调试不寻常的问题
- 寻求特定情况的最佳实践

## 脚本

### analyze_coverage.py

分析 Jest/Istanbul 覆盖率报告以识别缺口：

```bash
python3 scripts/analyze_coverage.py [coverage-file]
```

如果未指定，自动查找常见的覆盖率文件位置。

**输出：**
- 低于覆盖率阈值的文件
- Statement、branch 和 function 覆盖率百分比
- 需要改进的优先文件

### find_untested_code.py

查找没有对应测试文件的源文件：

```bash
python3 scripts/find_untested_code.py [src-dir] [--pattern test|spec]
```

**输出：**
- 源文件和测试文件总数
- 测试文件覆盖率百分比
- 按类型分类的未测试文件（API、services、components 等）
- 优先级建议

## 最佳实践总结

1. **先编写测试**（TDD）当添加新功能时
2. **测试行为，而不是实现** - 测试应在重构后仍然有效
3. **保持测试独立** - 测试之间没有共享状态
4. **使用描述性名称** - 测试名称应解释场景
5. **覆盖边界情况** - null、空值、边界值、错误条件
6. **Mock 外部依赖** - 测试应快速且可靠
7. **保持高覆盖率** - 关键代码 80%+
8. **立即修复失败的测试** - 永远不要提交损坏的测试
9. **重构测试** - 对测试应用与生产代码相同的质量标准
10. **使用工具** - 自动化覆盖率分析和缺口识别
