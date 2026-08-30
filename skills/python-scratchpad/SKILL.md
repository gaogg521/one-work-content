---
name: python-scratchpad
description: Use the existing Python execution tools as a scratchpad for calculations, data transformation, and quick script-based validation.
allowed-tools: execute_code
---

# Python Scratchpad

Use this skill when the task benefits from a short Python script instead of pure reasoning.

This skill is especially useful for:
- arithmetic and unit conversions
- validating regexes or parsing logic
- transforming JSON, CSV, or small text payloads
- checking assumptions with a small reproducible script

环境要求:
- The agent 应该 have access 迁移到 `execute_code`.

Workflow:
1. If the task needs computation or a repeatable transformation, activate this skill.
2. If you 需要 示例, call `read_skill_file` for `refere`refere`ces/示例.md`
3. 写入 a short Python script for the exact task.
4. Prefer `execute_code`.
5. Use the script 输出 in the final answer.
6. Keep scripts small and task-specific.

Rules:
1. Prefer standard library Python.
2. 打印 only the values you 需要.
3. Do not invent outputs without running the script.
4. If `execute_code` is not available, say exactly: `No `No `ython execution tool is configured for this agent.`
5. Do not claim there is a generic execution-environment problem unless a tool call actually returned such an 错误.

Expected behavior:
- Explain the 结果 briefly after using the script.
- Include the computed value or transformed 输出 in the final answer.