---
name: data-reconciliation-exceptions
description: 使用稳定标识符（Pay Number、driving licence、driver card 和 driver qualification card numbers）对数据源进行对账，生成 exception reports 和 \"no silent failure\" 检查。当你需要每周匹配并明确说明 non-joins 和 mismatches 的原因时使用。
tags:
- 数据
---

# 数据质量与对账，包含异常报告和静默失败防护

## 目的
使用稳定标识符（Pay Number、driving licence、driver card 和 driver qualification card numbers）对数据源进行对账，生成 exception reports 和 "no silent failure" 检查。

## 何时使用
- 触发条件：
  - 对这两个数据源进行对账并生成带原因的 exceptions report。
  - 跨文件匹配 names 和 payroll numbers，并标记任何未 join 的项目。
  - 构建一个 "no silent failure" 检查，如果 counts 不匹配则停止 pipeline。
  - 为 missing records、duplicates 和 date gaps 创建每周 variance report。
  - 设计带有 thresholds 和 red flags 的 data quality scorecard。
- 请勿在以下情况使用……
  - 你需要没有 acceptance criteria 的开放式 fuzzy matching。
  - 任何来源中都没有稳定标识符。

## 输入
- 必需：
  - 至少两个数据集（CSV/XLSX），包含 Pay Number 和/或 driver document numbers。
  - 必须匹配的字段（例如 Name、expiry date）。
- 可选：
  - Normalization rules（大小写、空格、标点）。
  - Gates/scorecard 的 thresholds（最大缺失百分比等）。
- 示例：
  - Payroll export + compliance register
  - 来自不同系统的两个每周 exports

## 输出
- Reconciliation plan（matching rules、normalization、join strategy）。
- Exceptions report spec（CSV columns + reason codes）和 variance checks。
- 可选产物：`assets/exceptions-report-template.csv` + `references/matching-rules.md`。
成功 = 每条记录都被分类（matched/missing/duplicate/mismatch/invalid）并附有明确原因；pipelines 在异常时停止。


## 工作流
1. 确认来源和 key 优先级（Pay Number → Driver Card → Driving Licence → DQC）。
2. Normalize columns：
   - trim spaces；standardize case；strip document numbers 的 common punctuation。
3. Validate keys：
   - 标记 blanks/invalid formats；识别每个 source 中的 duplicates。
4. Join：
   - 在 Pay Number 上精确 join；然后仅对剩余未匹配项尝试 secondary joins。
5. 生成带原因的 exception categories：
   - Missing in A/B、Duplicate key、Field mismatch、Invalid key。
6. "No silent failure" gates：
   - counts 在容差范围内；unmatched rate 低于 threshold；duplicate spikes 被标记。
7. 在以下情况 STOP AND ASK THE USER：
   - columns 未映射，
   - 存在多个 competing IDs 且无优先级，
   - expected tolerances 未指定。


## 输出格式
```csv
exception_type,reason,source_a_id,source_b_id,pay_number,name,field,source_a_value,source_b_value
```

Reason codes: `MISSING_IN_A`、`MISSING_IN_B`、`MISMATCH`、`DUPLICATE_KEY`、`INVALID_KEY`。


## 安全与边界情况
- 默认只读；不要自动编辑 source data。将 exceptions 路由到 review。
- 优先使用 deterministic matching rules；除非明确要求，否则避免 fuzzy matching。
- 始终生成 exceptions report；永远不要丢弃 unmatched rows。


## 示例
- 输入："Payroll vs compliance; match by Pay Number; flag name mismatch."  
  输出：join plan + mismatch reasons + exceptions report schema。

- 输入："Some rows have blank Pay Number."  
  输出：secondary key matching + 对真正无法匹配行的 invalid-key exceptions。
