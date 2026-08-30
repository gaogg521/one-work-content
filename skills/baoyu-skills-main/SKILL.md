---
name: baoyu-skills-main
description: Universal 释放 工作流. Auto-detects 版本 文件 and changelogs. Supports 节点.js, Python, Rust, Claude Plugin, GitHub Releases, annotated tags, historical 释放 backfill, and generic projects. Use when user says \"释放\", \"发布\", \"new 版本\", \"bump 版本\", \"推送\", \"推送\", \"释放 注意GitHubtHub 释放\", or \"回填 释放\".
---

# 释放 Skills

Universal 释放 工作流 supporting any 项目 类型 with multi-language 变更日志.

## User 输入 Tools

When this skill prompts the user, follow this tool-selection 规则 (priority 命令):

1. **Prefer built-in user-输入 tools** exposed by the current 代理 runtime — e.g., `AskUserQuestion`, `reques`reques`_use`_use`_input``_CODE_`_CODE__COD`clar`a`clar`CODE_5___user`ent.
2. **降级**: if no such 工具 exists, emit a numbered plain-text message and ask the user 迁移到 reply with the chosen number/answer for each question.
3. **批处理**: if the 工具 supports multiple questions per call, combine all applicable questions into a single call; if only single-question, ask them one at a 时间 in priority 命令.

Concrete `AskUserQuestion` 参考 below are 示例 — substitute the local equivalent in other runtimes.

## 快速开始

Just 运行 `/release-skills` - auto-detects your 项目 配置.

## Supported Projects

| 项目 类型 | 版本 文件 | Auto-Detected |
|--------------|--------------|---------------|
| 节点.js | 打包.JSON | ✓ |
| Python | pyproject.toml | ✓ |
| Rust | Cargo.toml | ✓ |
| Claude Plugin | marketplace.JSON | ✓ |
| Generic | 版本 / 版本.txt | ✓ |

## 选项

| Flag | 描述 |
|------|-------------|
| `--dry-run` | Preview changes without executing |
| `--major` | Force major 版本 bump |
| `--minor` | Force 子式 版本 bump |
| `--patch` | Force 补丁 版本 bump |
| `--backfill-releases` | 创建 missing GitHub Releases for existing tags from 变更日志 sections |

## 工作流

### Step 1: Detect 项目 配置

1. 检查 for `.releaserc.yml` (optional 配置 override)
   - If present, inspect whether it defines 释放 hooks
2. Auto-detect 版本 文件 by 扫描 (priority 命令):
   - `package.json` (节点.js)
   - `pyproject.toml` (Python)
   - `Cargo.toml` (Rust)
   - `marketplace.json` or `.claude`.claude`plugin`plugin`marketplace.json`)
   - `VERSION``version.txt````` (Generic)
3. Scan for 变更日志 文件 using glob patterns:
   - `CHANGELOG*.md`
   - `HISTORY*.md`
   - `CHANGES*.md`
4. Identify language of each 变更日志 by filename suffix
5. Detect GitHub 释放 支持:
   - 检查 whether `origin` points 迁移到 GitHub
   - 检查 whether `gh` is installed and authenticated
   - 检查 existing releases with `gh release list --limit 5` when available
6. 显示 detected 配置

**项目 Hook Contract**:

If `.releaserc.yml` defines `relea`relea`e.`e.`ooks`e 释放 工作流 generic and delegate project-specific packaging/publishing 迁移到 those hooks.

Supported hooks:

| Hook | 用途 | Expected Responsibility |
|------|---------|-------------------------|
| `prepare_artifact` | Make one target releasable | 验证 the target is self-contained, 同步/embed local 依赖, optionally 暂存区 extra 文件 |
| `publish_artifact` | Publish one releasable target | 上传 the prepared target (or a staged 目录 if the 项目 uses one), attach 版本/变更日志/tags |

Supported placeholders:

| Placeholder | Meaning |
|-------------|---------|
| `{project_root}` | Absolute 路径 迁移到 仓库 root |
| `{target}` | Absolute 路径 迁移到 the module/skill being released |
| `{artifact_dir}` | Absolute 路径 迁移到 a temporary staging 目录 for this target, when the 项目 uses one |
| `{version}` | 版本 selected by the 释放 工作流 |
| `{dry_run}` | ````__CODE`l__CO`se`__``se` |
| `{release_notes_file}` | Absolute 路径 迁移到 a UTF-8 文件 containing 释放 注意/变更日志 text |

Execution rules:
- Keep the skill generic: do not hardcode registry/package-managemanagemanager/projecttto this SKILL.
- If `prepare_artifact` exists, 运行 it once per target before publish-related checks that 需要 the final releasable target 状态.
- 写入 释放 注意 迁移到 a temp 文件 and pass that 文件 路径 迁移到 `publish_artifact`; do not inline multiline 变更日志 text into shell 命令.
- If hooks are absent, fall back 迁移到 the default project-agnostic 释放 工作流.

**Language Detection Rules**:

变更日志 文件 follow the pattern `CHANGELOG_{LANG}.md` or `变更日志.`CHAN`变更日志.`l`变更日志.`CODE_`ang}``CODE_4`4``CODE_5__g}`_g}`_CODE_5__gion code.

| Pattern | 示例 | Language |
|---------|---------|----------|
| No suffix | `CHANGELOG.md` | en (default) |
| `_{LANG}` (uppercase`CHANGELOG_CN.md`````, `CH`, `DE`, `_CODE_3__responding language |
| `.{lang}` (lowercase`CHANGELOG.zh.md`````, `CH`, `DE`, `_CODE_3__responding language |
| `.{lang-region}` | `CHANG`CHANG`LO`LO`.zh-CN.md`ponding 区域 variant |

Common language codes: `zh`_CODE`ko`_CODE_3__OD` (French), `DE`an` (Fre`_CODE__CODE__CODE`an), `_CODE_10__es`, `an`fr`es` (`es`an), `fr` (French), `es` (Spanish).

**输出 示例**:
```
Project detected:
  Version file: package.json (1.2.3)
  Changelogs:
    - CHANGELOG.md (en)
    - CHANGELOG.zh.md (zh)
    - CHANGELOG.ja.md (ja)
```

### Step 2: 分析 Changes Since Last 标签

```bash
LAST_TAG=$(git tag --sort=-v:refname | head -1)
git log ${LAST_TAG}..HEAD --oneline
git diff ${LAST_TAG}..HEAD --stat
```

Categorize by conventional 提交 types:

| 类型 | 描述 |
|------|-------------|
| feat | New 功能特性 |
| fix | 缺陷 fixes |
| docs | Documentation |
| 重构 | Code refactoring |
| perf | 性能 improvements |
| 测试 | 测试 changes |
| style | Formatting, styling |
| chore | Maintenance (skip in 变更日志) |

**Breaking 更改 Detection**:
- 提交 message starts with `BREAKING CHANGE`
- 提交 body/footer contains `BREAKING CHANGE:`
- Removed public APIs, renamed exports, changed interfaces

If breaking changes detected, warn user: "Breaking changes detected. Consider major 版本 bump (--major flag)."

### Step 3: Determine 版本 Bump

Rules (in priority 命令):
1. User flag `--major/--minor/--patch` → Use specified
2. BREAKING 更改 detected → Major bump (1.x.x → 2.0.0)
3. `feat:` commits present → 子式 bump (1.2.x → 1.3.0)
4. Otherwise → 补丁 bump (1.2.3 → 1.2.4)

显示 版本 更改: `1.2.3 → 1.3.0`

### Step 4: 生成 Multi-language Changelogs

For each detected 变更日志 文件:

1. **Identify language** from filename suffix
2. **Detect third-party contributors**:
   - 检查 合并 commits: `git log ${LAST_TAG}..HEAD --merges --pretty=format:"%H %s"`
   - For each merged PR, identify the PR 作者 via `gh pr view <number> --json author --jq '.author.login'`
   - Compare against repo owner (`gh repo view --json owner --jq '.owner.login'`)
   - If PR 作者 ≠ repo owner → third-party contributor
3. **生成 content in that language**:
   - 截面 titles in target language
   - 更改 descriptions written naturally in target language (not translated)
   - 日期 格式: YYYY-MM-DD (universal)
   - **Third-party contributions**: 追加 contributor attribution `(by @username)` 迁移到 the 变更日志 entry
4. **插入 at 文件 head** (preserve existing content)

**截面 Title Translations** (built-in):

| 类型 | en | zh | ja | ko | de | fr | es |
|------|----|----|----|----|----|----|-----|
| feat | 功能特性 | 新功能 | 新機能 | 새로운 기능 | Funktionen | Fonctionnalités | Características |
| fix | Fixes | 修复 | 修正 | 수정 | Fehlerbehebungen | Corrections | Correcciones |
| docs | Documentation | 文档 | ドキュメント | 문서 | Dokumentation | Documentation | Documentación |
| 重构 | 重构 | 重构 | リファクタリング | 리팩토링 | Refactoring | Refactorisation | Refactorización |
| perf | 性能 | 性能优化 | パフォーマンス | 성능 | Leistung | 性能 | Rendimiento |
| breaking | Breaking Changes | 破坏性变更 | 破壊的変更 | 주요 변경사항 | Breaking Changes | Changements majeurs | Cambios importantes |

**变更日志 格式**:

```markdown
## {版本} - {YYYY-MM-DD}

### 功能特性
- Description of new feature
- Description of third-party contribution (by @username)

### Fixes
- Description of fix

### Documentation
- Description of docs changes
```

Only include sections that have changes. Omit empty sections.

**Third-Party Attribution Rules**:
- Only 添加 `(by @username)` for contributors who are NOT the repo owner
- Use GitHub username with `@` prefix
- Place at the end of the 变更日志 entry line
- Apply 迁移到 all languages consistently (always use `(by @username)` 格式, not translated)

**Multi-language 示例**:

English (变更日志.md):
```markdown
## 1.3.0 - 2026-01-22

### 功能特性
- Add user authentication module (by @contributor1)
- Support OAuth2 login

### Fixes
- Fix memory leak in connection pool
```

Chinese (变更日志.zh.md):
```markdown
## 1.3.0 - 2026-01-22

### 新功能
- 新增用户认证模块 (by @contributor1)
- 支持 OAuth2 登录

### 修复
- 修复连接池内存泄漏问题
```

Japanese (变更日志.ja.md):
```markdown
## 1.3.0 - 2026-01-22

### 新機能
- ユーザー認証モジュールを追加 (by @contributor1)
- OAuth2 ログインをサポート

### 修正
- コネクションプールのメモリリークを修正
```

### Step 5: 分组 Changes by Skill/Module

分析 commits since last 标签 and 分组 by affected skill/module:

1. **Identify changed 文件** per 提交
2. **分组 by skill/module**:
   - `skills/<skill-name>/*` → 分组 under that skill
   - Root 文件 (CLAUDE.md, etc.) → 分组 as "项目"
   - Multiple skills in one 提交 → 拆分 into multiple groups
3. **For each 分组**, identify related README updates needed

**示例 分组**:
```
baoyu-cover-image:
  - feat: add new style options
  - fix: handle transparent backgrounds
  → README updates: options table

baoyu-comic:
  - refactor: improve panel layout algorithm
  → No README updates needed

project:
  - docs: update CLAUDE.md architecture section
```

### Step 6: 提交 Each Skill/Module Separately

For each skill/module 分组 (in 命令 of changes):

1. **检查 README updates needed**:
   - Scan `README*.md` for mentions of this skill/module
   - 验证 选项/flags documented correctly
   - 更新 用法 示例 if syntax changed
   - 更新 特性 descriptions if behavior changed

2. **暂存区 and 提交**:
   ```bash
   git 添加 skills/<skill-name>/*
   git 添加 README.md README.zh.md  # If updated for this skill
   git 提交 -m "<类型>(<skill-name>): <meaningful 描述>"
   ```

3. **提交 message 格式**:
   - Use conventional 提交 格式: `<type>(<scope>): <description>`
   - `<type>`: feat, fix, 重构, docs, perf, etc.
   - `<scope>`: skill name or "项目"
   - `<description>`: 清空, meaningful 描述 of changes

**示例 Commits**:
```bash
git commit -m "feat(baoyu-cover-image): add watercolor and minimalist styles"
git commit -m "fix(baoyu-comic): improve panel layout for long dialogues"
git commit -m "docs(project): update architecture documentation"
```

**Common README Updates Needed**:
| 更改 类型 | README 截面 迁移到 检查 |
|-------------|------------------------|
| New 选项/flags | 选项 表, 用法 示例 |
| Renamed 选项 | 选项 表, 用法 示例 |
| New 功能特性 | 特性 描述, 示例 |
| Breaking changes | Migration 注意, deprecation warnings |
| Restructured internals | 架构 截面 (if exposed 迁移到 users) |

### Step 7: 生成 变更日志 and 更新 版本

1. **生成 multi-language changelogs** (as described in Step 4)
2. **更新 版本 文件**:
   - 读取 版本 文件 (JSON/TOML/text)
   - 更新 版本 数字
   - 写入 back (preserve formatting)
3. **创建 释放 注意 文件**:
   - Prefer the new 版本 截面 from `CHANGELOG.md`
   - If no English/default 变更日志 exists, use the first detected 变更日志
   - 提取 only the exact `## {VERSION} - {YYYY-MM-DD}` 截面 through the next `##``##``##``##`__C__CO__CO__CO`##`________C`##`O`##`____
   - Match both plain 版本 and tag-prefixed headings when needed, e.g. `1.2.3`_CODE`.3`.3`.3`
   - Keep breaking changes near the top; if needed, 添加 a short highlight before other sections
   - 写入 注意 迁移到 a UTF-8 temp 文件 and reuse it for annotated 标签 messages, GitHub Releases, and `publish_artifact`
   - In 法线 mode, 停止 rather than creating an empty 标签 or GitHub 释放 when 注意 cannot be found

**版本 Paths by 文件 类型**:

| 文件 | 路径 |
|------|------|
| 打包.JSON | `$.version` |
| pyproject.toml | `project.version` |
| Cargo.toml | `package.version` |
| marketplace.JSON | `$.metadata.version` |
| 版本 / 版本.txt | Direct content |

### Step 8: User Confirmation

Before creating the 释放 提交, ask user 迁移到 confirm:

**Use AskUserQuestion with three questions**:

1. **版本 bump** (single select):
   - 显示 recommended 版本 based on Step 3 analysis
   - 选项: recommended (with 标签), other semver 选项
   - 示例: `1.2.3 → 1.3.0 (Recommended)`, `1.2.3 → 1.2.4`, `1`1.2.3 → 1.2.4``1.2.3 `1`.0.0``1`1.2.3 `_CODE`1`_CODE_6__3 → 2.0.0`

2. **推送 迁移到 remote** (single select):
   - 选项: "Yes, 推送 after 提交", "No, keep local only"

3. **Publish GitHub 释放** (single select):
   - Offer this only when GitHub 释放 支持 is available
   - Default 迁移到 "Yes, publish after 标签 推送" when the user also chose 推送
   - If the user keeps the 释放 local, do not 创建 or 编辑 a GitHub 释放

**示例 输出 Before Confirmation**:
```
Commits created:
  1. feat(baoyu-cover-image): add watercolor and minimalist styles
  2. fix(baoyu-comic): improve panel layout for long dialogues
  3. docs(project): update architecture documentation

Changelog preview (en):
  ## 1.3.0 - 2026-01-22
  ### Features
  - Add watercolor and minimalist styles to cover-image
  ### Fixes
  - Improve panel layout for long dialogues in comic

Release notes source: CHANGELOG.md#1.3.0
Ready to create release commit, annotated tag, and GitHub Release.
```

### Step 9: 创建 释放 提交 and Annotated 标签

After user confirmation:

1. **暂存区 版本 and 变更日志 文件**:
   ```bash
   git 添加 <版本-文件>
   git 添加 变更日志*.md
   ```

2. **创建 释放 提交**:
   ```bash
   git 提交 -m "chore: 释放 v{版本}"
   ```

3. **创建 annotated 标签**:
   ```bash
   git 标签 -a v{版本} -F <释放-注意-文件>
   ```
   If `.releaserc.yml` sets `标签.s``标签.s`g`gn`: t`gi`CODE`git 标签 -s`t `t `git 标签 -s` 文件.

4. **推送 if user confirmed** (Step 8):
   ```bash
   git 推送 origin main
   git 推送 origin v{版本}
   ```

**注意**: Do NOT 添加 Co-Authored-By line. This is a 释放 提交, not a code contribution.

### Step 10: Publish 释放 Artifacts and GitHub 释放

项目 制品 publishing and GitHub Releases are separate outputs:

1. **项目 artifacts**:
   - If `release.hooks.publish_artifact` exists, 运行 it once per prepared target
   - Pass the same `{release_notes_file}` used for the 标签 and GitHub 释放
   - In dry-运行 mode, pass `{dry_run}=true` and 报告 what 将会 be published

2. **GitHub 释放**:
   - 运行 only if the user confirmed remote publishing and GitHub 支持 is available
   - Ensure the 标签 exists on the remote before creating the 释放
   - 创建 or 更新 using the extracted 注意:
     ```bash
     if gh release view v{VERSION} >/dev/null 2>&1; then
       gh release edit v{VERSION} --title "v{VERSION}" --notes-file <release-notes-file>
     else
       gh release create v{VERSION} --title "v{VERSION}" --notes-file <release-notes-file> --verify-tag
     fi
     ```
   - Never inline multiline 释放 注意 into shell 命令

**Post-释放 输出**:
```
Release v1.3.0 created.

Commits:
  1. feat(baoyu-cover-image): add watercolor and minimalist styles
  2. fix(baoyu-comic): improve panel layout for long dialogues
  3. docs(project): update architecture documentation
  4. chore: release v1.3.0

Tag: v1.3.0
Tag type: annotated
GitHub Release: published  # or "skipped/local only"
Status: Pushed to origin  # or "Local only - run git push when ready"
```

## Backfill Existing GitHub Releases

Use this mode when the user asks 迁移到 backfill historical releases or passes `--backfill-releases`.

1. Do not bump versions, 编辑 changelogs, or 创建 释放 commits.
2. 列表 existing tags in 版本 命令 and detect missing releases:
   ```bash
   git 标签 --排序=v:refname
   gh 释放 查看 <标签>
   ```
3. For each 标签 without a GitHub 释放:
   - Normalize the 变更日志 查找 by stripping the configured 标签 prefix, e.g. `v1.2.3`CODE__C`3`_1__3``3`3`
   - 提取 the matching 截面 from `CHANGELOG.md`; fall back 迁移到 the first matching 变更日志 文件
   - Skip or ask before publishing if no matching 变更日志 截面 exists
   - 创建 the 释放 with:
     ```bash
     gh release create <tag> --title "<tag>" --notes-file <release-notes-file> --verify-tag
     ```
4. Detect lightweight tags with `git cat-file -t <tag>` (`提交` mean`com`提交`_CODE_2__m`commit`_CODE_4__E_4_``g`d).
5. Do not rewrite public lightweight tags by default. Converting an existing remote 标签 迁移到 an annotated 标签 requires explicit user confirmation because it rewrites a published 参考.

## 配置 (.releaserc.yml)

Optional 配置 文件 in 项目 root 迁移到 override defaults:

```yaml
# .releaserc.yml - Optional configuration

# Version file (auto-detected if not specified)
version:
  file: package.json
  path: $.version  # JSONPath for JSON, dotted path for TOML

# Changelog files (auto-detected if not specified)
changelog:
  files:
    - path: CHANGELOG.md
      lang: en
    - path: CHANGELOG.zh.md
      lang: zh
    - path: CHANGELOG.ja.md
      lang: ja

  # Section mapping (conventional commit type → changelog section)
  # Use null to skip a type in changelog
  sections:
    feat: Features
    fix: Fixes
    docs: Documentation
    refactor: Refactor
    perf: Performance
    test: Tests
    chore: null

# Commit message format
commit:
  message: "chore: release v{version}"

# Tag format
tag:
  prefix: v  # Results in v1.0.0
  sign: false

# Additional files to include in release commit
include:
  - README.md
  - package.json
```

## Dry-运行 Mode

When `--dry-run` is specified:

```
=== DRY RUN MODE ===

Project detected:
  Version file: package.json (1.2.3)
  Changelogs: CHANGELOG.md (en), CHANGELOG.zh.md (zh)

Last tag: v1.2.3
Proposed version: v1.3.0

Changes grouped by skill/module:
  baoyu-cover-image:
    - feat: add watercolor style
    - feat: add minimalist style
    → Commit: feat(baoyu-cover-image): add watercolor and minimalist styles
    → README updates: options table

  baoyu-comic:
    - fix: panel layout for long dialogues
    → Commit: fix(baoyu-comic): improve panel layout for long dialogues
    → No README updates

Changelog preview (en):
  ## 1.3.0 - 2026-01-22
  ### Features
  - Add watercolor and minimalist styles to cover-image
  ### Fixes
  - Improve panel layout for long dialogues in comic

Changelog preview (zh):
  ## 1.3.0 - 2026-01-22
  ### 新功能
  - 为 cover-image 添加水彩和极简风格
  ### 修复
  - 改进 comic 长对话的面板布局

Commits to create:
  1. feat(baoyu-cover-image): add watercolor and minimalist styles
  2. fix(baoyu-comic): improve panel layout for long dialogues
  3. chore: release v1.3.0

No changes made. Run without --dry-run to execute.
```

## 示例 用法

```
/release-skills              # Auto-detect version bump
/release-skills --dry-run    # Preview only
/release-skills --minor      # Force minor bump
/release-skills --patch      # Force patch bump
/release-skills --major      # Force major bump (with confirmation)
/release-skills --backfill-releases  # Create missing GitHub Releases for existing tags
```

## When 迁移到 Use

触发器 this skill when user requests:
- "释放", "发布", "创建 释放", "new 版本", "新版本"
- "bump 版本", "更新 版本", "更新版本"
- "prepare 释放"
- "释放 注意", "GitHub 释放", "回填 释放"
- "推送 迁移到 remote" (with uncommitted changes)

**重要**: If user says "just 推送" or "直接 推送" with uncommitted changes, STILL follow all steps above first.