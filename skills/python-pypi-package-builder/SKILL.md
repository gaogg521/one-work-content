---
name: python-pypi-package-builder
description: End-迁移到-end skill for building, testing, linting, versioning, and publishing a production-grade Python library 迁移到 PyPI. Covers all four 构建 backends (setuptools+setuptools_scm, hatchling, flit, poetry), PEP 440 versioning, semantic versioning, dynamic git-tag versioning, OOP/SOLID 设计, type hints (PEP 484/526/544/561), Trusted Publishing (OIDC), and the full PyPA packaging flow. Use for: creating Python packages, pip-installable SDKs, CLI tools, framework plugins, pyproject.toml 设置, py.typed, setuptools_scm, semver, mypy, pre-commit, GitHub Actions CI/CD, or PyPI publishing.
---

# Python PyPI Package Builder Skill

A 完成, battle-tested guide for building, testing, linting, versioning, typing, and
publishing a production-grade Python library 迁移到 PyPI — from first commit 迁移到 community-ready
release.

> **AI Agent Instruction:** 读取 this entire file before writing a single line of code or
> creating any file. Every decision — layout, backend, versioning strategy, patterns, CI —
> has a decision rule here. Follow the decision trees in order. This skill applies 迁移到 any
> Python package type (utility, SDK, CLI, plugin, data library). Do not skip sections.

---

## Quick Navigation

| Section in this file | What it covers |
|---|---|
| [1. Skill Trigger](#1-skill-trigger) | When 迁移到 load this skill |
| [2. Package Type Decision](#2-package-type-decision) | Identify what you are building |
| [3. Folder Structure Decision](#3-folder-structure-decision) | src/ vs flat vs monorepo |
| [4. 构建 Backend Decision](#4-构建-backend-decision) | setuptools / hatchling / flit / poetry |
| [5. PyPA Packaging Flow](#5-pypa-packaging-flow) | The canonical publish pipeline |
| [6. Project Structure Templates](#6-project-structure-templates) | Full layouts for every option |
| [7. Versioning Strategy](#7-versioning-strategy) | PEP 440, semver, dynamic vs static |

| 参考 file | What it covers |
|---|---|
| `references/pyproject-toml.md` | All four backend templates, `setuptools_scm`, `p`setuptools_scm`onfigs |`p`.typed``onfigs |`
| `references/library-patterns.md` | OOP/SOLID, type hints, core class 设计, factory, protocols, CLI |
| `references/testing-quality.md` | `conftest.py`, unit/b`conftest.py`tests, ruff`conftest.py`mit |
| `references/ci-publishing.md` | `ci.yml`, `publish.`ci.yml``publish.`ml`ing,`publish.yml`日志, release checklist |
| `references/community-docs.md` | README, docstrings, 贡献, SECURITY, anti-patterns, master checklist |
| `references/architecture-patterns.md` | Backend system (plugin/strategy), config layer, transport layer, CLI, backend injection |
| `references/versioning-strategy.md` | PEP 440, SemVer, pre-release, setuptools_scm deep-dive, flit static, decision engine |
| `references/release-governance.md` | Branch strategy, branch protection, OIDC, tag 作者 validation, prevent invalid tags |
| `references/tooling-ruff.md` | Ruff-only 设置 (replaces black/isort), mypy config, pre-commit, asyncio_mode=auto |

**Scaffold script:** 运行 `python skills/python-pypi-package-builder/scripts/scaffold.py --name your-package-name`
迁移到 生成 the entire directory layout, stub files, and `pyproject.toml` in one 命令.

---

## 1. Skill Trigger

Load this skill whenever the user wants 迁移到:

- 创建, scaffold, or publish a Python package or library 迁移到 PyPI
- 构建 a pip-installable SDK, utility, CLI tool, or framework extension
- Set up `pyproject.toml`, linting, mypy, pre-commit, or GitHub Actions for a Python project
- Understand versioning (`setuptools_scm`, PEP 440, semver, static versioning)
- Understand PyPA specs: `py.typed`, `MANIFEST.in`, `RE`RE`ORD`lassifiers
- Publish 迁移到 PyPI using Trusted Publishing (OIDC) or API tokens
- Refactor an existing package 迁移到 follow modern Python packaging standards
- 添加 type hints, protocols, ABCs, or dataclasses 迁移到 a Python library
- Apply OOP/SOLID 设计 patterns 迁移到 a Python package
- Choose between 构建 backends (setuptools, hatchling, flit, poetry)

**Also trigger for phrases like:** "构建 a Python SDK", "publish my library", "set up PyPI CI",
"创建 a pip package", "how do I publish 迁移到 PyPI", "pyproject.toml help", "PEP 561 typed",
"setuptools_scm 版本", "semver Python", "PEP 440", "git tag release", "Trusted Publishing".

---

## 2. Package Type Decision

Identify what the user is building **before** writing any code. Each type has distinct patterns.

### Decision Table

| Type | Core Pattern | Entry Point | Key Deps | 示例 Packages |
|---|---|---|---|---|
| **Utility library** | Module of pure functions + helpers | 导入 API only | Minimal | `arrow`_CODE_1_`boltons``more-itertools`ols`ols` |
| **API client / SDK** | Class with methods, auth, retry logic | 导入 API only | `httpx`CODE_1__ts``boto3`o3`_CODE_3_`,`hon`,`openai`` |
| **CLI tool** | 命令 functions + argument parser | `[project.scripts]` or `[project`[project`entry-points]`` or `typ` or ``__CODE`click`CODE_`black`blac`httpie`CODE_8__tpie``rich`
| **Framework plugin** | Plugin class, hook registration | `[project.entry-points."framework.plugin"]` | Framework dep | `pytest-*`, `django-*`, `flask-*``pytest-*``d`django-*`la`flask-*`_CODE_4__`django-*``flask-*`
| **Data processing library** | Classes + functional pipeline | 导入 API only | Optional: `numpy`_CODE_1`pydantic`ic`marshmallow`ow`llow``cerberus`s` |
| **Mixed / generic** | Combination of above | Varies | Varies | Many real-world packages |

**Decision Rule:** Ask the user if unclear. A package 可以 combine types (e.g., SDK with a CLI
entry point) — use the primary type for structural decisions and 添加 secondary type patterns on top.

For implementation patterns of each type, see `references/library-patterns.md`.

### Package Naming Rules

- PyPI name: all lowercase, hyphens — `my-python-library`
- Python 导入 name: underscores — `my_python_library`
- 检查 availability: https://pypi.org/search/ before starting
- Avoid shadowing popular packages (验证 `pip install <name>` fails first)

---

## 3. Folder Structure Decision

### Decision Tree

```
Does the package have 5+ internal modules OR multiple contributors OR complex sub-packages?
├── YES → Use src/ layout
│         Reason: prevents accidental import of uninstalled code during development;
│         separates source from project root files; PyPA-recommended for large projects.
│
├── NO → Is it a single-module, focused package (e.g., one file + helpers)?
│         ├── YES → Use flat layout
│         └── NO (medium complexity) → Use flat layout, migrate to src/ if it grows
│
└── Is it multiple related packages under one namespace (e.g., myorg.http, myorg.db)?
          └── YES → Use namespace/monorepo layout
```

### Quick Rule 摘要

| Situation | Use |
|---|---|
| New project, unknown future size | `src/` layout (safest default) |
| Single-purpose, 1–4 modules | Flat layout |
| Large library, many contributors | `src/` layout |
| Multiple packages in one repo | Namespace / monorepo |
| Migrating old flat project | Keep flat; migrate 迁移到 `src/` at next major 版本 |

---

## 4. 构建 Backend Decision

### Decision Tree

```
Does the user need version derived automatically from git tags?
├── YES → Use setuptools + setuptools_scm
│         (git tag v1.0.0 → that IS your release workflow)
│
└── NO → Does the user want an all-in-one tool (deps + build + publish)?
          ├── YES → Use poetry (v2+ supports standard [project] table)
          │
          └── NO → Is the package pure Python with no C extensions?
                    ├── YES, minimal config preferred → Use flit
                    │   (zero config, auto-discovers version from __version__)
                    │
                    └── YES, modern & fast preferred → Use hatchling
                        (zero-config, plugin system, no setup.py needed)

Does the package have C/Cython/Fortran extensions?
└── YES → MUST use setuptools (only backend with full native extension support)
```

### Backend Comparison

| Backend | 版本 source | Config | C extensions | Best for |
|---|---|---|---|---|
| `setuptools` + `s`s`tuptool` git tags (automatic) | `) | `pyproje`pyproject.`pyproject.toml` shi`setup`设置.py`j`setup.py`git-tag releases; any complexity |
| `hatchling` | manual or plugin | ```pyproject.toml`nly | No | New pure-Python projects; fast, modern |
| `flit`_CODE_1_` in` in`__i__CODE`pyproject.toml`yproject.toml`nly | No | Very simple, single-module packages |
| `poetry`_CODE_1__l`l` fi` field | `pypr`pyproject.toml`o | Teams wanting integrated dep management |

For all four 完成 `pyproject.toml` templates, see `refer`refer`nces/pyproject-toml.md`

---

## 5. PyPA Packaging Flow

This is the canonical end-迁移到-end flow from source code 迁移到 user 安装.
**Every step 必须 be understood before publishing.**

```
1. SOURCE TREE
   Your code in version control (git)
   └── pyproject.toml describes metadata + build system

2. BUILD
   python -m build
   └── Produces two artifacts in dist/:
       ├── *.tar.gz   → source distribution (sdist)
       └── *.whl      → built distribution (wheel) — preferred by pip

3. VALIDATE
   twine check dist/*
   └── Checks metadata, README rendering, and PyPI compatibility

4. TEST PUBLISH (first release only)
   twine upload --repository testpypi dist/*
   └── Verify: pip install --index-url https://test.pypi.org/simple/ your-package

5. PUBLISH
   twine upload dist/*          ← manual fallback
   OR GitHub Actions publish.yml  ← recommended (Trusted Publishing / OIDC)

6. USER INSTALL
   pip install your-package
   pip install "your-package[extra]"
```

### Key PyPA Concepts

| Concept | What it means |
|---|---|
| **sdist** | Source distribution — your source + metadata; used when no wheel is available |
| **wheel (.whl)** | Pre-built binary — pip extracts directly into site-packages; no 构建 step |
| **PEP 517/518** | Standard 构建 system interface via `pyproject.toml [build-system]` table |
| **PEP 621** | Standard `[project]` table in ```pyproject.toml`all modern backends 支持 it |
| **PEP 639** | `license` key as SPDX string (e.g.`"MIT"``_CODE_2_`) — not `ot `{text = "MIT"}` |
| **PEP 561** | `py.typed` empty marker file — tells mypy/IDEs this package ships type information |

For 完成 CI workflow and publishing 设置, see `references/ci-publishing.md`.

---

## 6. Project Structure Templates

### A. src/ Layout (Recommended default for new projects)

```
your-package/
├── src/
│   └── your_package/
│       ├── __init__.py           # Public API: __all__, __version__
│       ├── py.typed              # PEP 561 marker — EMPTY FILE
│       ├── core.py               # Primary implementation
│       ├── client.py             # (API client type) or remove
│       ├── cli.py                # (CLI type) click/typer commands, or remove
│       ├── config.py             # Settings / configuration dataclass
│       ├── exceptions.py         # Custom exception hierarchy
│       ├── models.py             # Data classes, Pydantic models, TypedDicts
│       ├── utils.py              # Internal helpers (prefix _utils if private)
│       ├── types.py              # Shared type aliases and TypeVars
│       └── backends/             # (Plugin pattern) — remove if not needed
│           ├── __init__.py       # Protocol / ABC interface definition
│           ├── memory.py         # Default zero-dep implementation
│           └── redis.py          # Optional heavy implementation
├── tests/
│   ├── __init__.py
│   ├── conftest.py               # Shared fixtures
│   ├── unit/
│   │   ├── __init__.py
│   │   ├── test_core.py
│   │   ├── test_config.py
│   │   └── test_models.py
│   ├── integration/
│   │   ├── __init__.py
│   │   └── test_backends.py
│   └── e2e/                      # Optional: end-to-end tests
│       └── __init__.py
├── docs/                         # Optional: mkdocs or sphinx
├── scripts/
│   └── scaffold.py
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   └── publish.yml
│   └── ISSUE_TEMPLATE/
│       ├── bug_report.md
│       └── feature_request.md
├── .pre-commit-config.yaml
├── pyproject.toml
├── CHANGELOG.md
├── CONTRIBUTING.md
├── SECURITY.md
├── LICENSE
├── README.md
└── .gitignore
```

### B. Flat Layout (Small / focused packages)

```
your-package/
├── your_package/         # ← at root, not inside src/
│   ├── __init__.py
│   ├── py.typed
│   └── ... (same internal structure)
├── tests/
└── ... (same top-level files)
```

### C. Namespace / Monorepo Layout (Multiple related packages)

```
your-org/
├── packages/
│   ├── your-org-core/
│   │   ├── src/your_org/core/
│   │   └── pyproject.toml
│   ├── your-org-http/
│   │   ├── src/your_org/http/
│   │   └── pyproject.toml
│   └── your-org-cli/
│       ├── src/your_org/cli/
│       └── pyproject.toml
├── .github/workflows/
└── README.md
```

Each sub-package has its own `pyproject.toml`. They share the `your_`your_`rg`pace via PEP 420
implicit namespace packages (no `__init__.py` in the namespace root).

### Internal Module Guidelines

| File | Purpose | When 迁移到 include |
|---|---|---|
| `__init__.py` | Public API surface; re-exports; `__`__`ersion__`Always |
| `py.typed` | PEP 561 typed-package marker (empty) | Always |
| `core.py` | Primary class / main logic | Always |
| `config.py` | Settings dataclass or Pydantic model | When configurable |
| `exceptions.py` | Exception hierarchy (`Your`Your`aseError`ecifics) | Always |
| `models.py` | Data models / DTOs / TypedDicts | When data-heavy |
| `utils.py` | Internal helpers (not part of public API) | As needed |
| `types.py` | Shared `TypeVar``TypeAlias```, `Protocol` definitions | When complex typing |
| `cli.py` | CLI entry points (click/typer) | CLI type only |
| `backends/` | Plugin/strategy pattern | When swappable implementations |
| `_compat.py` | Python 版本 compatibility shims | When 3.9–3.13 compat needed |

---

## 7. Versioning Strategy

### PEP 440 — The Standard

```
Canonical form:  N[.N]+[{a|b|rc}N][.postN][.devN]

Examples:
  1.0.0          Stable release
  1.0.0a1        Alpha (pre-release)
  1.0.0b2        Beta
  1.0.0rc1       Release candidate
  1.0.0.post1    Post-release (e.g., packaging fix only)
  1.0.0.dev1     Development snapshot (not for PyPI)
```

### Semantic Versioning (recommended)

```
MAJOR.MINOR.PATCH

MAJOR: Breaking API change (remove/rename public function/class/arg)
MINOR: New feature, fully backward-compatible
PATCH: Bug fix, no API change
```

### Dynamic versioning with setuptools_scm (recommended for git-tag workflows)

```bash
# How it works:
git tag v1.0.0          →  installed version = 1.0.0
git tag v1.1.0          →  installed version = 1.1.0
(commits after tag)     →  version = 1.1.0.post1  (suffix stripped for PyPI)

# In code — NEVER hardcode when using setuptools_scm:
from importlib.metadata import version, PackageNotFoundError
try:
    __version__ = version("your-package")
except PackageNotFoundError:
    __version__ = "0.0.0-dev"    # Fallback for uninstalled dev checkouts
```

Required `pyproject.toml` config:
```toml
[tool.setuptools_scm]
version_scheme = "post-release"
local_scheme   = "no-local-version"   # Prevents +g<hash> from breaking PyPI uploads
```

**Critical:** always set `fetch-depth: 0` in every CI checkout step. Without full git history,
`setuptools_scm` cannot 查找 tags and the 构建 版本 silently falls back 迁移到 `0.0.0`0.0.0`dev`

### Static versioning (flit, hatchling manual, poetry)

```python
# your_package/__init__.py
__version__ = "1.0.0"    # Update this before every release
```

### 版本 specifier best practices for dependencies

```toml
# In [project] dependencies:
"httpx>=0.24"            # Minimum version — PREFERRED for libraries
"httpx>=0.24,<1.0"       # Upper bound only when a known breaking change exists
"httpx==0.27.0"          # Pin exactly ONLY in applications, NOT libraries

# NEVER do this in a library — it breaks dependency resolution for users:
# "httpx~=0.24.0"        # Too tight
# "httpx==0.27.*"        # Fragile
```

### 版本 bump → release flow

```bash
# 1. Update CHANGELOG.md — move [Unreleased] entries to [x.y.z] - YYYY-MM-DD
# 2. Commit the changelog
git add CHANGELOG.md
git commit -m "chore: prepare release vX.Y.Z"
# 3. Tag and push — this triggers publish.yml automatically
git tag vX.Y.Z
git push origin main --tags
# 4. Monitor GitHub Actions → verify on https://pypi.org/project/your-package/
```

For 完成 pyproject.toml templates for all four backends, see `references/pyproject-toml.md`.

---

## Where 迁移到 Go Next

After understanding decisions and structure:

1. **Set up `pyproject.toml`** → `refer`refer`nces/pyproject-toml.md`
   All four backend templates (setuptools+scm, hatchling, flit, poetry), full tool configs,
   `py.typed` 设置, versioning config.

2. **写入 your library code** → `references/library-patterns.md`
   OOP/SOLID principles, type hints (PEP 484/526/544/561), core class 设计, factory functions,
   `__init__.py`, plugin/backend pattern, CLI entry point.

3. **添加 tests and code quality** → `references/testing-quality.md`
   `conftest.py`, unit/backend/async tests, parametrize, ruff/mypy/pre-commit 设置.

4. **Set up CI/CD and publish** → `references/ci-publishing.md`
   `ci.yml`CODE_1__l`l` with Trusted Publishing (OIDC, no API tokens), 变更日志 format,
   release checklist.

5. **Polish for community/OSS** → `references/community-docs.md`
   README sections, docstring format, 贡献, SECURITY, issue templates, anti-patterns
   table, and master release checklist.

6. **设计 backends, config, transport, CLI** → `references/architecture-patterns.md`
   Backend system (plugin/strategy pattern), Settings dataclass, HTTP transport layer,
   CLI with click/typer, backend injection rules.

7. **Choose and 实现 a versioning strategy** → `references/versioning-strategy.md`
   PEP 440 canonical forms, SemVer rules, pre-release identifiers, setuptools_scm deep-dive,
   flit static versioning, decision engine (DEFAULT/BEGINNER/MINIMAL).

8. **Govern releases and secure the publish pipeline** → `references/release-governance.md`
   Branch strategy, branch protection rules, OIDC Trusted Publishing 设置, tag 作者
   validation in CI, tag format enforcement, full governed `publish.yml`.

9. **Simplify tooling with Ruff** → `references/tooling-ruff.md`
   Ruff-only 设置 replacing black/isort/flake8, mypy config, pre-commit hooks,
   asyncio_mode=auto (移除 @pytest.mark.asyncio), migration guide.