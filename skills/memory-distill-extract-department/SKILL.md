---
name: memory-distill-extract-department
description: 部门记忆提炼（标准粒度、中等合并，AMC 内部专用）
---


## 输入

`initial_user_message` 为 JSON，`run_mode` 必须为 `memory_distill`。

关注字段：

- `memory_distill_phase`：`ingest` | `finalize`
- `memory_level`：应为 `department`
- `extraction_policy`：`extraction_depth=standard`，`merge_policy=moderate`，`max_chunks_hint` 为上限
- `additional_prompt`：用户自定义补充要求（非空时必须遵守）

## distill_phase=ingest（memory_distill_phase=ingest）

读取 `trace_brief_batch`，提炼**团队/部门级**可复用知识：共识、排障套路、部门规范、协作约定。

- **标准粒度**：一条记忆对应一个清晰主题
- **中等合并**：合并相近条目，去掉重复与琐碎步骤
- **排除**：纯个人偏好、一次性闲聊、密钥、仅适用于单人的偶发细节
- 按本批 `trace_brief_batch` 主题输出 JSON（勿写文件）：

```json
{"batch_chunks":[{"summary":"…","content":"…","tags":["department"]}]}
```

- 条数 ≤ `max_batch_chunks`；无内容则 `{"batch_chunks":[]}`

## distill_phase=finalize（memory_distill_phase=finalize）

`finalize_mode=merge_staged_chunks`：合并 `accumulated_batch_chunks`，按主题去重，勿偏向对话末尾单一主题。

`finalize_mode=from_trace_full`：结合全量 trace 与 ingest 上下文，输出**部门记忆片段**。

**禁止写文件**：不得使用文件/Shell 工具，不得将结果写入 `/tmp/` 等路径。平台只解析助手消息正文中的 JSON。

**只输出一个 JSON 对象**（勿 markdown 围栏、勿额外说明，首字符必须是 `{`）：

```json
{"chunks":[{"summary":"部门级摘要","content":"可跨成员复用的部门知识","tags":["department"]}]}
```

约束：

- `chunks` 数量 ≤ `extraction_policy.max_chunks_hint`（默认约 10）
- 内容应可供部门成员共享，避免「我/某同事」式个人指代（除非必要上下文）
- 遵守 `additional_prompt` 与 `level_guidance`（在 `extraction_policy` 中）
- 不要编造 trace 中未出现的结论
