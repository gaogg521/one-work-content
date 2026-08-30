---
name: memory-distill-extract-global
description: 全局记忆提炼（粗粒度、强合并，AMC 内部专用）
---


## 输入

`initial_user_message` 为 JSON，`run_mode` 必须为 `memory_distill`。

关注字段：

- `memory_distill_phase`：`ingest` | `finalize`
- `memory_level`：应为 `global`
- `extraction_policy`：`extraction_depth=coarse`，`merge_policy=aggressive`，`max_chunks_hint` 为上限
- `additional_prompt`：用户自定义补充要求（非空时必须遵守）

## distill_phase=ingest（memory_distill_phase=ingest）

读取 `trace_brief_batch`，识别**可跨团队复用**的原则、标准、通用模式。

- **粗粒度抽象**：上升为平台/组织级表述，避免项目偶发细节
- **强合并去重**：同类知识合并为少量高价值条目
- **排除**：个人习惯、部门内部黑话、未验证猜测、密钥
- 按本批主题输出 `{"batch_chunks":[...]}`（≤ `max_batch_chunks`，勿写文件）

## distill_phase=finalize（memory_distill_phase=finalize）

`finalize_mode=merge_staged_chunks`：合并各批 `accumulated_batch_chunks`，强去重、强抽象，覆盖全部主题。

`finalize_mode=from_trace_full`：输出**全局记忆片段**，供多团队检索。

**禁止写文件**：不得使用文件/Shell 工具，不得将结果写入 `/tmp/` 等路径。平台只解析助手消息正文中的 JSON。

**只输出一个 JSON 对象**（勿 markdown 围栏、勿额外说明，首字符必须是 `{`）：

```json
{"chunks":[{"summary":"全局原则摘要","content":"抽象后的组织级知识","tags":["global"]}]}
```

约束：

- `chunks` 数量 ≤ `extraction_policy.max_chunks_hint`（默认约 6），宁少勿滥
- 每条应独立成立、可长期有效；避免具体人名、工单号、临时环境信息
- 严格遵守 `additional_prompt`；与全局抽象冲突时以安全与可复用性优先
- 不要编造 trace 中未出现的规则
