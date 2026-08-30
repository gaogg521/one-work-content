---
name: skill-distill-author
description: 从 TraceBrief 分批提炼可复用 Skill（AMC 内部专用）
---


## 输入

`initial_user_message` 为 JSON，`run_mode` 必须为 `skill_distill`。

**重要**：每批 / finalize 均为**独立空会话**。请**只阅读本条 JSON** 中的字段；不要假设存在更早对话，也不要复述 `trace_brief_batch` 原文。

关注字段：

- `distill_phase`：`ingest` | `finalize`
- `prior_digests`（ingest）：此前各批已落库的 BatchDigest 摘要
- `batch_digests`（finalize）：全部批次 Digest
- `session_summary`（finalize）：标题与用户目标
- `finalize_mode`：通常为 `merge_batch_digests`；仅异常回退时为 `from_trace_full`

## distill_phase=ingest

读取 `trace_brief_batch`（可结合 `prior_digests` 去重），理解本批对话中的**分析思路与处理思路**（不是 tool 调用清单）。

关注：分析/处理模式、能力描述、思路顺序、常见误区、待确认信息。

**禁止**具体 tool / shell 名（如 ls、execute、read_file、sshpass、vim-cmd）。

**必须只输出一个 JSON 对象**（BatchDigest；勿 markdown、勿长报告、勿「最终汇总」、勿 SKILL.md）：

```json
{
  "batch_index": 1,
  "content": "本批分析/处理思路的简短摘要（数句即可）",
  "patterns": ["可复用模式…"],
  "capabilities": ["analysis.network"],
  "workflow_steps": ["先确认…", "再比对…"],
  "pitfalls": ["常见误区…"],
  "open_questions": []
}
```

无实质内容时仍输出 JSON，字段可为空数组 / 空字符串。首字符必须是 `{`。

## distill_phase=finalize

读取 `batch_digests` 与 `session_summary`（`finalize_mode=merge_batch_digests` 时**不要**依赖完整 trace）。

**只输出一个 JSON 对象**（SkillDraft，供 AMC 程序解析；不要在 JSON 外写说明、不要 markdown 代码围栏）。

`skill_md` 必须严格按同包内 `skill-template.md` 填写完整 markdown（作为 JSON 字符串，换行写 `\n`），保留全部章节标题：

- `## 何时使用`、`## 用户目标`、`## 先了解什么`
- `## 分析思路`、`## 处理思路`、`## 行动要点`
- `## 输出要求`、`## 注意事项`、`## 能力依赖`

JSON 字段：

- `skill_name`（string，小写连字符，与 skill_md front matter 的 name 一致；**禁止** `distilled-skill`）
- `skill_md`（string，完整 SKILL.md 正文；description 须为有意义概括，禁止占位文案 `distilled skill`）
- `generic_capabilities`（string[]，与「能力依赖」一致）
- `scripts`（可选，通常 `[]`）

示例（单行 JSON，勿照搬内容）：

```json
{"skill_name":"apm-service-triage","skill_md":"---\nname: apm-service-triage\ndescription: …\n---\n\n# …\n\n## 何时使用\n\n- …","generic_capabilities":["analysis.apm"],"scripts":[]}
```

## 输出约束

- **ingest**：只输出 BatchDigest JSON
- **finalize**：只输出 SkillDraft JSON；`skill_md` 在 JSON 字符串内
- 不要引用密钥、token、密码
- 不要编造 digests / trace 中未出现的思路
- 不要把 TraceBrief 字段（actionLabel、parametersPattern、resultSummary）原样写入 skill_md
