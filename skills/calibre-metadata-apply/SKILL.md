---
name: calibre-metadata-apply
description: 通过 Content server 使用 calibredb 为现有 Calibre 书籍应用元数据更新。在通过只读查询确认目标 ID 后，用于受控的元数据编辑。
metadata: None
openclaw: None
dependsOnSkills:
- subagent-spawn-command-builder
localWrites:
- skills/calibre-metadata-apply/state/runs.json
- ~/.config/calibre-metadata-apply/auth.json
- ~/.config/calibre-metadata-apply/config.json
modifiesRemoteData:
- calibre:metadata
optionalBins:
- pdffonts
optionalEnv:
- CALIBRE_USERNAME
primaryEnv: CALIBRE_PASSWORD
requires: None
bins:
- node
- calibredb
env:
- CALIBRE_PASSWORD
tags:
- 元数据
---

# calibre-metadata-apply

一个用于更新现有 Calibre 书籍元数据的技能。

## 要求

- `calibredb` 必须在运行环境的 PATH 中可用
- 已安装 `subagent-spawn-command-builder`（用于生成 spawn payload）
- `pdffonts` 为可选/推荐，用于 PDF 证据检查
- 可访问的 Calibre Content server URL
  - `http://HOST:PORT/#LIBRARY_ID`
- 如果启用了认证，优先使用 `/home/altair/.openclaw/.env`：
  - `CALIBRE_USERNAME=<user>`
  - `CALIBRE_PASSWORD=<password>`
- 传递 `--password-env CALIBRE_PASSWORD`（用户名自动从环境变量加载）
- 你仍然可以用 `--username <user>` 显式覆盖。
- 可选的认证缓存：`--save-auth`（默认文件：`~/.config/calibre-metadata-apply/auth.json`）

## 支持的字段

### 直接字段（`set_metadata --field`）
- `title`
- `title_sort`
- `authors`（带 `&` 的字符串或数组）
- `author_sort`
- `series`
- `series_index`
- `tags`（字符串或数组）
- `publisher`
- `pubdate`（`YYYY-MM-DD`）
- `languages`
- `comments`

### 辅助字段
- `comments_html`（OC 标记块 upsert）
- `analysis`（为 comments 自动生成 analysis HTML）
- `analysis_tags`（添加标签）
- `tags_merge`（默认 `true`）
- `tags_remove`（合并后移除特定标签）

## 必需执行流程

### A. 目标确认（强制）
1. 运行只读查询以缩小候选范围
2. 显示 `id,title,authors,series,series_index`
3. 获取用户对最终目标 ID 的确认
4. 仅使用已确认 ID 构建 JSONL

### B. 提案综合（当元数据缺失时）
1. 从文件提取 + 网络来源收集证据
2. 显示一个合并的提案表格，包含：
   - `candidate`, `source`, `confidence (high|medium|low)`
   - `title_sort_candidate`, `author_sort_candidate`
3. 获取用户决策：
   - `approve all`
   - `approve only: <fields>`
   - `reject: <fields>`
   - `edit: <field>=<value>`
4. 仅应用已批准/最终确定的字段
5. 如果置信度低或来源冲突，保持字段为空

### C. 应用
1. 先运行 dry-run（强制）
2. 仅在用户明确批准后运行 `--apply`
3. 重新读取并报告最终值

## Analysis worker 策略

- 使用 `subagent-spawn-command-builder` 生成 `sessions_spawn` payload 用于繁重的候选生成
  - `task` 是必需的。
  - Profile 应包含此工作流的 model/thinking/timeout/cleanup。
- 使用轻量级 subagent 模型进行分析（避免主重模型）
- 在主会话中保留最终决策 + dry-run/apply

## 数据流披露

- 本地执行：
  - 从 JSONL 构建 `calibredb set_metadata` 命令。
  - 读/写本地状态文件（`state/runs.json`）和 `~/.config/calibre-metadata-apply/` 下的可选认证/配置文件。
- Subagent 执行（可选，用于繁重的候选生成）：
  - 通过 `subagent-spawn-command-builder` 使用 `sessions_spawn`。
  - 发送给 subagent 的文本/元数据可以到达运行时 profile 配置的模型端点。
- 远程写入：
  - `calibredb set_metadata` 更新目标 Calibre Content server 上的元数据。

安全规则：
- 除非用户明确指示，否则不要使用 `--save-plain-password`。
- 优先使用基于环境变量的密码（`--password-env CALIBRE_PASSWORD`）而非内联 `--password`。
- 如果用户不希望外部模型/subagent 处理，保持流程本地并跳过 subagent 编排。

## 长运行分轮次策略（全库范围）

对于全库范围的繁重处理，始终使用分轮次执行。

## 未知文档恢复流程（M3）

批次大小规则：
- 保持每个未知文档批次足够小，以在聊天中显示完整的逐行结果（不要代表性抽样）。
- 如果仍有未解决项目，停止并等待用户明确指示再开始下一批次。

### 用户干预检查点（固定）

1. **轻量通过（仅元数据）**
   - 默认始终运行此阶段（无需额外用户指令）
   - 仅分析现有元数据（不读取文件内容）
   - 向用户展示表格：
     - 当前文件/标题
     - 推荐的标题/元数据
     - 置信度/证据摘要
   - 在任何更深入阶段之前停止并等待用户指令

2. **应用户请求：第1页通过**
   - 仅读取第1页并优化提案
   - 报告与轻量通过的差异

3. **如果仍不确定：深度通过**
   - 读取前5页 + 后5页
   - 添加网络证据搜索
   - 生成带置信度和依据的最终化提案

4. **审批关口**
   - 显示详细发现并请求明确批准后再应用

### 待处理和不受支持的处理

- 对未解决/搁置项目使用 `pending-review` 标签。
- 如果文档在当前流程中未解决，不要强制猜测元数据。
  - 标记为 `pending-review` 并保留给后续调查。

### 差异报告格式（用于未知批次运行）

返回完整结果（不要抽样）：
- 执行摘要（目标/已更改/待处理/已跳过/错误）
- 完整已更改列表，包含 `id` + 关键前后字段
- 完整待处理列表，包含 `id` + 原因
- 完整错误列表，包含 `id` + 错误摘要
- 置信度必须表达为 `high|medium|low`

### 运行时产物策略

- 仅在运行活跃时保留运行状态和临时产物。
- 成功完成后，移除每次运行的状态/产物。
- 失败时，仅保留用于重试/调试的最小产物，解决后清理。

### 内部编排（推荐）

- 使用轻量级 subagent 进行所有分析阶段
- 在主会话中保留应用决策
- 在每个阶段将运行状态持久化到 `state/runs.json`

### 第1轮（开始）
1. 主会话定义范围
2. 主会话通过 `subagent-spawn-command-builder` 生成 spawn payload（profile 示例：`calibre-meta`），然后调用 `sessions_spawn`
3. 通过 `scripts/run_state.mjs upsert` 保存 `run_id/session_key/task`
4. 立即告诉用户这是一个 subagent 作业，并说明用于分析的执行模型
5. 回复 "analysis started" 并保持正常聊天响应

### 第2轮（完成）
1. 接收 subagent 完成通知
2. 保存结果 JSON
3. 通过 `scripts/handle_completion.mjs --run-id ... --result-json ...` 完成状态处理
4. 返回汇总提案（仅在需要时应用）

运行状态文件：
- `state/runs.json`

## PDF 提取策略

1. 先尝试 `ebook-convert`
2. 如果为空/失败，回退到 `pdftotext`
3. 如果两者都失败，切换到 web-evidence-first 模式

## 读音策略

- 使用用户配置的 `reading_script` 处理日语/非拉丁排序字段
  - `katakana` / `hiragana` / `latin`
- 首次使用时询问一次，然后持久化并复用
- 默认策略为完整读音（不截断）
- 配置路径：`~/.config/calibre-metadata-apply/config.json`
  - 键：`reading_script`

## 用法

Dry-run：

```bash
cat changes.jsonl | node skills/calibre-metadata-apply/scripts/calibredb_apply.mjs \
  --with-library "http://127.0.0.1:8080/#MyLibrary" \
  --password-env CALIBRE_PASSWORD \
  --lang ja
```

Apply：

```bash
cat changes.jsonl | node skills/calibre-metadata-apply/scripts/calibredb_apply.mjs \
  --with-library "http://127.0.0.1:8080/#MyLibrary" \
  --password-env CALIBRE_PASSWORD \
  --apply
```

## 禁止事项

- 不要仅使用模糊标题匹配直接运行 `--apply`
- 不要在 apply payload 中包含未确认的 ID
- 不要在没有明确确认的情况下自动填充低置信度候选
