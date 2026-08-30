---
name: memory-distill-extract-personal
description: 个人记忆提炼（细粒度、弱合并，AMC 内部专用）
---


## 输入

`initial_user_message` 为 JSON，`run_mode` 必须为 `memory_distill`。

关注字段：

- `memory_distill_phase`：`ingest` | `finalize`
- `memory_level`：应为 `personal`
- `extraction_policy`：`extraction_depth=fine`，`merge_policy=minimal`，`max_chunks_hint` 为上限
- `additional_prompt`：用户自定义补充要求（非空时必须遵守，且不得违背安全与隐私边界）

## distill_phase=ingest（memory_distill_phase=ingest）

读取 `trace_brief_batch`，提炼**个人可长期复用**的信息：偏好、习惯、个人事实、个人排障结论。

- **保留较细粒度**：相近但侧重点不同的条目可分开记
- **弱合并**：仅合并明显重复的同一句事实
- **排除**：部门 SOP、公司级规范、一次性闲聊、密钥/token
- **按本批 `trace_brief_batch` 的主题**提炼，勿混入其他批次主题
- 本阶段**必须输出 JSON**（勿写文件），供平台逐批累积：

```json
{"batch_chunks":[{"summary":"主题摘要","content":"完整记忆正文","tags":["personal"]}]}
```

- 条数 ≤ `max_batch_chunks`（通常每批 1～6 条）；无内容则 `{"batch_chunks":[]}`

## distill_phase=finalize（memory_distill_phase=finalize）

当 `finalize_mode=merge_staged_chunks` 时：阅读 `accumulated_batch_chunks`（各批已提炼条目），**按主题去重合并**后输出最终个人记忆；**禁止**只保留对话末尾用户问题相关主题（例如仅 Oracle），必须覆盖 Redis/Mongo 等较早主题。

当 `finalize_mode=from_trace_full` 时：结合 `trace_brief` 与 `trace_brief_batches` 输出个人记忆片段。

**禁止写文件**：不得使用 `read_file` / `write_file` 等工具，不得将 JSON 保存到 `/tmp/` 或任意路径（例如 `memory_distill_result.json`）。平台**只解析助手消息正文**中的 JSON；写到文件会导致 chunks 为空、提炼失败。

**只输出一个 JSON 对象**（勿 markdown 围栏、勿写「提取结果摘要」等说明，首字符必须是 `{`）：

```json
{"chunks":[{"summary":"一句话摘要","content":"完整记忆正文","tags":["personal"]}]}
```

约束：

- `chunks` 数量 ≤ `extraction_policy.max_chunks_hint`（默认约 16）
- `content` 必填，自洽可读；`summary` 建议 ≤120 字
- 遵守 `additional_prompt`；无额外要求时按个人记忆标准执行
- 合并时保留各主题的代表性条目，不要编造 trace 中未出现的事实
