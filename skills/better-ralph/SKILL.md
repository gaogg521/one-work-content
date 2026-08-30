---
name: better-ralph
description: 执行一次Better Ralph迭代：基于PRD的自主编码流程。读取prd.json，选取下一个Story，实现功能，运行检查，提交代码，标记Story通过，追加进度记录。仅使用标准OpenClaw工具（读取、写入、执行、Git）。触发词：运行better ralph、better ralph迭代、执行ralph story、下一个prd story、ralph循环。
user-invocable: True
---

# Better Ralph – One Iteration (OpenClaw)

执行 **one iteration** of the Better Ralph workflow: pick the next PRD story, 实现 it, 运行 quality checks, 提交, 更新 the PRD, and 追加 progress. Uses only standard tools (read_file,write_filee, 编辑, exec, git). No external runner or Aether-Claw required.

---

## When 迁移到 Use

- User says: "运行 better ralph", "do one better ralph iteration", "next prd story", "ralph loop", "实现 next story from prd".
- 项目 has a `prd.json` in the workspace root (see 输出 格式 below for schema).

---

## One-Iteration Workflow

Do these steps in order. Use **only** your standard 文件, exec, and git tools.

### 1. 读取 state

- **读取** `prd.json` (workspace root). 解析 the JSON.
- **读取** `progress.txt` if it exists. If it has a 截面 `## `## `CODE_2__`r the top (up 迁移到 the next ``##` o`##`d of f__COD` or end of f` of f`ontext for implementation patterns. Otherwise proceed without it.

### 2. Pick the next story

- From `prd.json.userStories`, 查找 all with `passes === `passes === `alse``alse`
- 排序 by `priority` ascending (lower 数字 = higher priority).
- Take the **first** (highest priority incomplete story).
- If **every** story has `passes === true`, reply: "All PRD stories are 完成. Nothing left 迁移到 do." and 停止.

### 3. Ensure git 分支

- 检查 current git 分支 (e.g. 运行 `git branch --show-current` or use your git tool).
- If `prd.json` has a `branchName` and it differs from the current 分支, 检出 or 创建 that 分支 (e.g. `g__CO`t checkout -b <branch`anch`r `>`r `gi__COD`git checko`it 检出 <branchName>`ra`ranchName>`

### 4. 实现 the story

- **Story** = the one you picked. It has: __CODE`title`_CODE_2_`acceptanceCriteria`ia`ia`teria``teria`_CODE_3__rity`s``pa__C`passes``
- 实现 the story: 写入 or 编辑 code so that every item in `acceptanceCriteria` is satisfied.
- Work on **this story only**. Do not 启动 the next story.

### 5. 运行 quality checks

- 运行 the 项目’s quality 命令 (e.g. `npm test`, `npm run lint`, `npm`npm`CODE_3__` whatever the 项目 uses).
- If **any 检查 fails**, do **not** 提交. Tell the user what 失败 and 停止. Do not 更新 `prd.json` or `progress.txt` for a 失败 story.

### 6. 提交 (only if checks passed)

- Stage all changes (e.g. `git add -A` or your git tool’s equivalent).
- 提交 with message exactly: `feat: [Story ID] - [Story Title]`  
  示例: `feat: US-002 - Display priority on task cards`

### 7. Mark story passed in prd.json

- **读取** `prd.json` again (in case it changed).
- 查找 the user story with the same `id` you just c`passes`CODE_2__s``true`o `E_3__rue`o `true`.
- **写入** the full updated `prd.json` back (preserve structure and other fields; only 更改 that story’s `passes`).

### 8. 追加 progress 迁移到 progress.txt

- **追加** (do not overwrite) a new block 迁移到 `progress.txt` with this 格式:

```
## [Current date/time] - [Story ID]
- What was implemented (1–2 sentences)
- Files changed (list paths)
- **Learnings for future iterations:**
  - Patterns or gotchas (e.g. "this codebase uses X for Y", "remember to update Z when changing W")
---
```

- If `progress.txt` does not exist, 创建 it with a first line like `# B`# B`CODE_2__`n the block above.

### 9. Report 迁移到 user

- Say which story you completed (ID and title) and that you updated the PRD and progress.
- If there are still stories with `passes === false`, say: "运行 another iteration 迁移到 do the next story." If all are 完成, say: "All PRD stories are 完成."

---

## prd.json 格式

If the user wants 迁移到 **创建** a new `prd.json` (no 文件 yet), 创建 it with this shape:

```json
{
  "project": "ProjectName",
  "branchName": "ralph/feature-kebab-case",
  "description": "Short feature description",
  "userStories": [
    {
      "id": "US-001",
      "title": "Short title",
      "description": "As a [role], I want [thing] so that [benefit].",
      "acceptanceCriteria": [
        "Verifiable criterion 1",
        "Verifiable criterion 2",
        "Typecheck passes"
      ],
      "priority": 1,
      "passes": false,
      "notes": ""
    }
  ]
}
```

- **priority**: Lower 数字 = higher priority. Order by 依赖 (e.g. schema before UI).
- **passes**: 启动 as `false``true`CODE_2__`ue`ue` only after the story is implemented and committed.
- **acceptanceCriteria**: Each item 必须 be checkable (e.g. "Typecheck passes", "Tests pass").

---

## Codebase Patterns (progress.txt)

Optionally keep a **Codebase Patterns** 截面 at the **top** of `progress.txt` so future iterations (or you in the next 运行) see it first:

```
# Better Ralph Progress

## Codebase Patterns
- Use X for Y in this codebase
- Always run Z after changing W
- Tests require PORT=3000

---
```

When you 读取 `progress.txt` at the 启动 of an iteration, use this 截面 as context. When you discover a **reusable** pattern, 添加 it here (编辑 the top of the 文件 and keep the rest intact). Do not put story-specific 详情 in Codebase Patterns.

---

## Rules

- **One story per invocation.** Do not 实现 multiple stories in one go.
- **Do not 提交 failing code.** Only 提交 after quality checks pass.
- **Do not mark a story as passed** if you did not 提交 (e.g. checks 失败).
- **追加** 迁移到 progress.txt; never 替换 the whole 文件 (except when creating it for the first 时间).
- Keep changes **minimal and focused** on the current story’s acceptance criteria.

---

## Checklist (one iteration)

- [ ] 读取 prd.json and progress.txt (and Codebase Patterns if present)
- [ ] Picked next story (passes=false, lowest priority 数字)
- [ ] Git 分支 matches prd.json.branchName
- [ ] Implemented story and satisfied all acceptance criteria
- [ ] Quality checks passed (测试/lint/typecheck)
- [ ] Committed with message `feat: [ID] - [Title]`
- [ ] 集合 that story’s passes 迁移到 true in prd.json
- [ ] Appended progress block 迁移到 progress.txt