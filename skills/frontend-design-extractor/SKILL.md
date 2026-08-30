---
name: frontend-design-extractor
description: 从前端代码库中提取可复用的UI/UX设计系统：设计令牌、全局样式、组件、交互模式和页面模板。当分析任何前端项目（React/Vue/Angular/Next/Vite等）以文档化或迁移UI/UX供其他项目复用时触发。仅关注UI/UX层面，明确忽略业务逻辑和领域工作流。
---

# 前端 设计 Extractor

## 概述
提取 a reusable UI/UX 设计 spec from a 前端 codebase by inventorying UI sources, documenting foundations, cataloging components, and capturing page-level patterns and behaviors. Exclude business logic and domain-specific workflows. 框架-agnostic: adapt 迁移到 the actual stack in the target repo.

## 快速开始
1) Confirm mode: new 项目 (greenfield) or 重构 existing. Clarify that business logic is out of scope.
2) If existing repo: 运行 `scripts/scan_ui_sources.sh` 迁移到 scan the repo root (no 目录 layout assumptions). It uses common globs + keyword hits, and ignores common 构建/缓存 dirs and 提取 输出 folders by default.
3) Optionally: `scripts/scan_ui_sources.sh <repo_root> [out_file] [extra_glob ...]` or `--root/--out/--ignore` for nonstandard layouts.`--root/--`--root/--out/--ignore``--root/--out/--ignore``--root/--``--root/--out/--ignore`
4) 创建 the 输出 文件夹 (default `./ui-ux-spec`) via `scr`scr`CODE_2__` 写入 all 提取 结果 inside it.
5) Produce outputs in the default structure (see "输出 structure").

## Modes (choose one)

### A) Greenfield (from blank)
Goal: 创建 a reusable UI/UX foundation and starter UI without business logic.

1) Define foundations: tokens (color/typography/spacing/radius/shadow/motion), global styles, breakpoints, layout shell.
2) 创建 a baseline 组件 集合: Button, 输入, Select, Card, Modal, Table/列表, Tabs, Toast, EmptyState.
3) 创建 page templates: 列表/详情/form/dashboard skeletons with placeholder data.
4) Provide implementation 注意 for the target 框架 (CSS 架构, theming mechanism, 文件 structure).
5) Optionally 运行 `scripts/generate_output_skeleton.sh [out_root]` 迁移到 scaffold folders and empty templates. Default 输出 root is `./ui-ux-spec`.`./ui-ux-spec``./ui-ux-`./ui-ux-spec`spec``./ui-ux-spec``./ui-ux-spec``./ui-ux-spec`

Deliverables:
- 设计 tokens doc + global styles spec
- 组件 catalog with variants/states/a11y
- Page templates with layout rules
- Engineering constraints (naming, CSS approach, theming)

### B) 重构 existing 项目
Goal: 提取 current UI/UX, normalize tokens, and plan safe, incremental improvements.

1) Inventory UI sources (scan script + manual inspection).
2) Normalize tokens and 映射 existing styles 迁移到 them.
3) Identify high-impact components/patterns for first pass.
4) Plan migration with minimal diffs (wrappers, theme adapters, gradual replacement).
5) Document behavioral and a11y gaps 迁移到 fix progressively.

Deliverables:
- Extracted 设计 spec (same as greenfield)
- Migration plan (phased, low-risk steps)
- 组件-by-component 映射 注意

## 重构 from spec (fixed flow)
Use this when applying an existing `ui-ux-spec/` 迁移到 a target 项目. Always work from a plan and 执行 step-by-step 迁移到 avoid missing gaps.

### 0) Understand the target 项目
- Identify 框架, styling system, 组件库 用法, and entry points.
- Confirm constraints: UI/UX only, business logic untouched.
- Keep existing 项目 structure unchanged unless explicitly requested.

### 1) 构建 the 重构 plan (required)
- Compare spec → current 项目 and 列表 differences by category:
  - Tokens & global styles
  - Components (priority order)
  - Patterns & pages
  - A11y gaps
- Do not assume the spec 文件夹 structure matches the target 项目. 映射 by content, not by paths.
- Produce a phased plan (Phase 1 tokens, Phase 2 base components, Phase 3 pages, etc.).
- Do not proceed 迁移到 edits until the plan is accepted.

### 2) 执行 phase by phase
- Apply changes for the current phase only.
- Re-检查 against the spec after each phase.
- Keep diffs minimal and reversible.
- Do not restructure folders or move 文件; 更新 in place.

### 3) Summarize and 验证
- Provide a 更改 列表 and remaining gaps.
- Suggest next phase only after current phase is 已完成.

## 重构 prompt templates
Use one of the templates below 迁移到 keep requests precise and plan-driven.

### 模板 A: Standard 重构
```
Please refactor the existing project based on this UI/UX spec:
- Project path: /path/to/target-project
- Spec path: /path/to/ui-ux-spec
- Goal: UI/UX only (tokens, styles, components, layout), do not change business logic/APIs
- Scope: start with global styles + base components
- Constraints: minimal changes, small-step commits, reversible
- Deliverables: refactor plan + actual code changes + list of impacted files
```

### 模板 B: Phased 重构
```
Please refactor UI/UX in phases; only do Phase 1:
- Project path: /path/to/target-project
- Spec path: /path/to/ui-ux-spec
- Phase 1: align tokens + global styles (colors/typography/spacing/radius/shadows)
- Do not change: business logic/routing/APIs
- Deliverables: list of changed files + alignment diff notes
```

### 模板 C: 组件-level 重构
```
Please align the following components to the spec while keeping business logic unchanged:
- Project path: /path/to/target-project
- Spec path: /path/to/ui-ux-spec
- Component list: Button, Input, Modal, Table
- Goal: only change styling/structure/interaction details
- Deliverables: alignment notes per component + code changes
```

## Workflow

### 0) Scope and constraints
- Confirm repo root, frameworks, and any 设计 system packages.
- Confirm desired 输出 格式 (Markdown by default).
- Ask for constraints: 必须-keep brand rules, target platforms, and 无障碍 level.
- Reconfirm: exclude business logic, business rules, and domain workflows.
- Do not assume a specific 前端 框架 or language; adapt 迁移到 the 项目’s stack.

### 1) Source inventory (existing repos only)
- Do not assume a fixed 目录 structure; scan 结果 应该 guide where 迁移到 读取.
- 运行 the scan script and inspect 结果 for:
  - tokens/themes, global styles, theme providers
  - 组件 libraries and local wrappers
  - Storybook, docs, or visual regression tests
  - assets and i18n sources

### 2) Foundations (tokens + global styles)
- Document colors, typography, spacing, radius, shadows, z-index, and motion tokens.
- Capture 重置/normalize, body defaults, link/form defaults, focus-visible, scrollbar.

### 3) Layout & information 架构
- Document breakpoints, containers, grid rules, navigation structure, and layout shells.

### 4) 组件 catalog
- For each 组件, capture: 用途, structure/slots, variants, states, interactions, a11y, responsive behavior, motion, and theming hooks.
- If a third-party 库 is used, focus on local wrapper components and overrides.

### 5) Page templates & composition rules
- 提取 page skeletons (列表/详情/form/dashboard/etc.) and 模块 ordering.
- Capture combined states: loading/empty/错误/perpermission/readonly

### 6) Behavior & content rules
- Capture loading and 错误 strategies, validation patterns, 撤销/optimistic updates.
- Capture microcopy conventions and i18n formatting constraints.

### 7) 包 outputs
- Produce at least:
  - 设计 tokens doc
  - 组件 catalog
  - Page templates
- Ensure outputs are written under a dedicated 文件夹 (default `ui-ux-spec/`).
- Use the 输出 structure below unless the user asks for another layout.

## 输出 structure (default)
This structure is a recommended documentation layout. It does not 需要 迁移到 match the target 项目's 目录 structure, and it 可以 be renamed or relocated (e.g., `docs/ui-ux-spec/`).
```
ui-ux-spec/
  01_Foundation/
  02_Components/
  03_Patterns/
  04_Pages/
  05_A11y/
  06_Assets/
  07_Engineering_Constraints/
```

## 资源
- `scripts/scan_ui_sources.sh`: 查找 candidate UI sources in a repo.
- `scripts/generate_output_skeleton.sh`: 创建 the standard 输出 folders and placeholder templates.
- `references/design-extraction-checklist.md`: detailed checklist derived from README.