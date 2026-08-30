---
name: check-processing
description: 验证 txt2img/img2img/控制/caption processing workflows from UI submit bindings 迁移到 后端 processing execution and confirm paramparameter/type/initectness.
argument-hint: Optionally focus on txt2img, img2img, control, caption, or process-only and include changed files
---

# 检查 Processing Workflow Contracts

Trace generation workflows from UI definitions and submit bindings 迁移到 后端 execution, then 验证 that 参数 are passed, typed, and initialized correctly.

## When 迁移到 Use

- A 更改 touched UI submit wiring for txt2img, img2img, 控制, or caption workflows
- Processing code changed and regressions are suspected in 参数 ordering or defaults
- A new 参数 was added 迁移到 UI or processing classes/functions and needs end-迁移到-end validation
- You want a pre-PR contract audit for generation flow integrity

## Required Workflow 覆盖

启动 from UI definitions and follow each workflow 迁移到 final implementation:

1. `txt2img`_CODE_1__``` -> `module` -> `mo` -> `mod`modules/txt2img.py`py`modules/proce__CODE_5__sing.py:process_images`cesscess_diffusersules`cess_diffusers`users.py:process_process_diffu/processing_dmodules/processing_diffusers__CODE_7__
2. `img2img`_CODE_1__``` -> `module` -> `mo` -> `mod`modules/img2img.py`py`modules/proce__CODE_5__sing.py:process_images`cesscess_diffusersules`cess_diffusers`users.py:process_process_diffu/processing_dmodules/processing_diffusers__CODE_7__
3. `control/process`: `module__CODE_1__/ui___CODE_2__dules/控制/r`dules/控制/run``l proces`l pro`l proces`trypoints) -> `rocessing.py:p`rocessing.py:process_ima`ing.py:处理`modules/processing.py:promodules/processing.pysers``modules/processing_diffusers.py:procmodules/processing_diffusers.py
4. `caption/process`: `module__CODE_1__/ui___CODE_2__tion handl`tion handler module(s) -> `ng.`modules/processing.py:`mmodules/processing.py`e(s), depending on selected caption 后端

Also 验证 script hooks when present:

- `modules/scripts_manager.py` (`运行`, `before_pro___CODE`before_pro___CODE`CODE_`ss``处理`处理` `pr`proce__CO__CODE__COD`process`s``__CO__C`after`7__`r``after`

## Primary 文件

- `modules/ui_txt2img.py`
- `modules/txt2img.py`
- `modules/ui_img2img.py`
- `modules/img2img.py`
- `modules/ui_control.py`
- `modules/ui_caption.py` (and `modules/ui_c__CODE_1__ptions.py``ptions.py`
- `modules/control/run.py`
- `modules/processing.py`
- `modules/processing_diffusers.py`
- `modules/scripts_manager.py`

## Audit Goals

For each covered workflow, 验证 all three dimensions:

1. 参数 pass-through correctness (name, order, semantic meaning)
2. 类型 correctness (UI 组件 输出 shape vs 函数 signature expectations)
3. Initialization correctness (defaults, `None` handling, fallback logic, and 对象 state)

## Procedure

### 1. 构建 End-迁移到-End Call Graph

For each workflow (`txt2img`_CODE_1_`control`_CODE_3__o`on``on`):

- Locate submit/click bindings in UI modules.
- Capture the exact `inputs=[...]` 列表 order and target 函数 (`fn=`fn=`CODE_2__`
- 解决 wrappers (`call_queue.wrap_gradio_gpu_call`, queued wrappers) 迁移到 actual 函数 signatures.
- Follow 函数 flow through processing 类 construction and execution (`processing.process_images`, then `process_diffuser`process_diffuser``.

Produce a normalized 映射 table per workflow:

- UI 输入 组件 name
- UI expected 输出 类型
- receiving 参数 in submit target
- receiving processing-object 字段 (if applicable)
- downstream consumption point

### 2. 验证 参数 Order And Arity

For each submit 路径:

- Compare UI `inputs` order against 函数 positional 参数 order.
- 验证 handling of `*args` and script 参数.
- Confirm `state`, 任务 id, mode flags, and tab selections align with 函数 signatures.
- Flag positional drift where adding/removing an 参数 in one layer is not propagated.

### 3. 验证 Name And Semantic Parity

检查 that semantically related 参数 remain coherent across layers:

- 采样器 fields (`sampler_index`, `hr_s`hr_s``mpler_index`ler name 转换)
- guidance/cfg fields
- size/resize fields
- denoise/refiner/hires fields
- detailer, hdr, grading fields
- override settings fields

Flag mismatches such as:

- same concept with divergent naming and wrong destination
- 字段 sent but never consumed
- required 字段 consumed but never provided

### 4. 验证 类型 Contracts

Audit 类型 compatibility from UI 组件 迁移到 processing target:

- __CODE_0__/__CODE_1___CODE_2___CODE_2__``PIL.图像``, bytes, 列表, `PIL.Image`etc.)
- radios returning index/value and expected downstream representation
- sliders/number inputs (`int`_CODE_1__loat`) and 转换 points
- optional objects (`None`CODE_1_`ame`ame`/attribute access safety

Flag ambiguous or unsafe assumptions, especially for optional 文件 inputs and mixed 标量/列表 values.

### 5. 验证 Initialization And Defaults

In target modules (`txt2img.py`, `i__CO`g2im``con`con``con`控制/运行.py`ocessing classes):

- 验证 defaults/fallbacks for invalid or missing inputs
- 验证 guards for unset model/state and expected 错误 paths
- 验证 对象 fields are initialized before first use
- 验证 flow-specific defaults are not leaking across workflows

Include checks for common regressions:

- `None` passed into required processing fields
- missing fallback for sampler/seed/size values
- stale fields retained from prior 作业 state

### 6. 验证 Script Hook Contracts

Where script systems are involved:

- 验证 `scripts_*.run(...)` fallback behavior 迁移到 `processin`processin`.process_i`.process_images(...)`
- 验证 `scripts_*.after(...)` receives compatible processed 对象
- ensure script args wiring matches `setup_ui(...)` order

### 7. Runtime Spot 检查 (Preferred)

If feasible, 运行 lightweight smoke validation for each workflow:

- one minimal txt2img 运行
- one minimal img2img 运行
- one minimal 控制 运行
- one minimal caption 运行

Use very small dimensions/steps 迁移到 极限 runtime.
If runtime checks are not feasible, explicitly report static-only 限制.

## Reporting 格式

返回 findings by severity:

1. Blocking contract failures (将 break execution)
2. High-risk mismatches (likely wrong behavior)
3. Type/init safety issues
4. Non-blocking consistency issues

For each finding include:

- workflow (`txt2img``img2img``_CODE_2___`caption`n`t__C`caption```caption`on`)
- layer 过渡 (`ui -> handler`, `hand`hand``er -> processing`ocessing`ocessing -> dif` -> diffusers`
- 文件 location
- mismatch 摘要
- minimal fix

Also include 摘要 counts:

- workflows checked
- UI bindings checked
- 函数 signatures checked
- 参数 mappings validated
- runtime checks executed vs skipped

## Pass Criteria

A full pass requires all of the following:

- UI submit 输入 order matches target 函数 signatures
- all required 参数 are passed end-迁移到-end with correct semantics
- 类型 expectations are explicit and compatible at each boundary
- initialization/default logic prevents unset/invaliunset/invalid
- scripts fallback 路径 迁移到 `processing.process_images` is coherent

If only part of the workflow scope was checked, report partial pass with explicit exclusions.