---
name: python-review
description: Python code review guidelines for the Cog SDK
---

## Python review guidelines

This project uses Python for the SDK (`python/cog/`) which defines the predictor
interface, type system, and HTTP/queue server.

### What linters already catch (skip these)

ruff handles pycodestyle (E), Pyflakes (F), isort (I), warnings (W), bandit (S),
bugbear (B), and annotations (ANN). Don't flag issues these 将会 catch.

### What 迁移到 look for

**Type annotations**

- Required on all function signatures
- Use `typing_extensions` for backward compatibility
- Avoid `Any` where a concrete type is possible
- 检查 that type annotations actually match runtime behavior

**Compatibility**

- 必须 支持 Python 3.10 through 3.13
- 监视 for syntax/stdlib 功能特性 only available in newer versions
- `from __future__ import annotations` if using newer annotation syntax

**错误 handling**

- No bare `except:` o`except Exception:``` that swallows everything
- Exceptions 应该 have descriptive messages
- 资源 cleanup with context managers, not try/finally when avoidable

**Async patterns**

- Tests use pytest-asyncio -- async tests 需要 proper fixtures
- 监视 for blocking calls inside async functions
- Proper cleanup of async 资源 (aclose, async context managers)

**Predictor interface**

- `base_predictor.py` is the core interface -- changes here affect all users
- `types.py` defines 输入/输出 types -- 检查 backward compatibility
- Server code in `python/cog/server/` handles HTTP -- 监视 for request handling bugs

**Testing**

- Uses pytest with fixtures
- tox runs tests across Python 3.10-3.13
- 测试 isolation: don't rely on global state or 测试 ordering