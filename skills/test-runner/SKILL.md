---
name: test-runner
description: | 语言 | 单元测试 | 集成测试 | 端到端测试 |
tags:
- 测试
- 文档
---

# test-runner

跨语言和框架编写和运行测试。

## 框架选择

| 语言 | 单元测试 | 集成测试 | 端到端测试 |
|----------|-----------|-------------|-----|
| TypeScript/JS | Vitest (preferred), Jest | Supertest | Playwright |
| Python | pytest | pytest + httpx | Playwright |
| Swift | XCTest | XCTest | XCUITest |

## 按框架快速开始

### Vitest (TypeScript / JavaScript)
```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom
```

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './tests/setup.ts',
  },
})
```

```bash
npx vitest              # 监听模式
npx vitest run          # 单次运行
npx vitest --coverage   # 带覆盖率
```

### Jest
```bash
npm install -D jest @types/jest ts-jest
```

```bash
npx jest                # 运行全部
npx jest --watch        # 监听模式
npx jest --coverage     # 带覆盖率
npx jest path/to/test   # 单个文件
```

### pytest (Python)
```bash
uv pip install pytest pytest-cov pytest-asyncio httpx
```

```bash
pytest                          # 运行全部
pytest -v                       # 详细输出
pytest -x                       # 第一次失败时停止
pytest --cov=app                # 带覆盖率
pytest tests/test_api.py -k "test_login"  # 指定测试
pytest --tb=short               # 短回溯
```

### XCTest (Swift)
```bash
swift test                      # 运行全部测试
swift test --filter MyTests     # 指定测试套件
swift test --parallel           # 并行执行
```

### Playwright (E2E)
```bash
npm install -D @playwright/test
npx playwright install
```

```bash
npx playwright test                    # 运行全部
npx playwright test --headed           # 浏览器可见
npx playwright test --debug            # 调试模式
npx playwright test --project=chromium # 指定浏览器
npx playwright show-report             # 查看 HTML 报告
```

## TDD 工作流

1. **Red** — 编写一个描述期望行为的失败测试。
2. **Green** — 编写最小代码使测试通过。
3. **Refactor** — 在保持测试通过的情况下清理代码。

```
┌─────────┐     ┌─────────┐     ┌──────────┐
│  Write   │────▶│  Write  │────▶│ Refactor │──┐
│  Test    │     │  Code   │     │  Code    │  │
│  (Red)   │     │ (Green) │     │          │  │
└─────────┘     └─────────┘     └──────────┘  │
     ▲                                          │
     └──────────────────────────────────────────┘
```

## 测试模式

### Arrange-Act-Assert
```typescript
test('calculates total with tax', () => {
  // Arrange
  const cart = new Cart([{ price: 100, qty: 2 }]);

  // Act
  const total = cart.totalWithTax(0.08);

  // Assert
  expect(total).toBe(216);
});
```

### 测试异步代码
```typescript
test('fetches user data', async () => {
  const user = await getUser('123');
  expect(user.name).toBe('Colt');
});
```

### Mocking
```typescript
import { vi } from 'vitest';

const mockFetch = vi.fn().mockResolvedValue({
  json: () => Promise.resolve({ id: 1, name: 'Test' }),
});
vi.stubGlobal('fetch', mockFetch);
```

### 测试 API 端点 (Python)
```python
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.mark.asyncio
async def test_get_users():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.get("/users")
    assert response.status_code == 200
    assert isinstance(response.json(), list)
```

### 测试 React 组件
```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

test('calls onClick when clicked', () => {
  const handleClick = vi.fn();
  render(<Button onClick={handleClick}>Click me</Button>);
  fireEvent.click(screen.getByText('Click me'));
  expect(handleClick).toHaveBeenCalledOnce();
});
```

## 覆盖率命令

```bash
# JavaScript/TypeScript
npx vitest --coverage          # Vitest (uses v8 or istanbul)
npx jest --coverage            # Jest

# Python
pytest --cov=app --cov-report=html    # HTML 报告
pytest --cov=app --cov-report=term    # 终端输出
pytest --cov=app --cov-fail-under=80  # 如果 < 80% 则失败

# 查看 HTML 覆盖率报告
open coverage/index.html       # macOS
open htmlcov/index.html        # Python
```

## 测试什么

**始终测试:**
- Public API / exported functions
- Edge cases: empty input, null, boundary values
- Error handling: invalid input, network failures
- Business logic: calculations, state transitions

**无需测试:**
- Private implementation details
- Framework internals (React rendering, Express routing)
- Trivial getters/setters
- Third-party library behavior
