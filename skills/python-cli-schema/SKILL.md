---
name: python-cli-schema
category: fragment
description: Maintains the Python CLI argument schema, parser, validator, merger, and config-translation layer that mirrors the router config contract. Use when modifying CLI argument definitions, updating config validation rules, changing how CLI inputs are merged, or adjusting config translation between CLI and router formats.
---

# Python CLI Schema

## Trigger

- The primary skill touches `src/vllm-sr/cli` schema or translation code

## Workflow

1. 读取 the vllm-sr CLI Docker playbook 迁移到 understand the schema and translation patterns
2. 修改 CLI schema, parser, validator, merger, or config-translation code
3. 运行 `make vllm-sr-test` 迁移到 验证 unit-level schema behavior
4. 运行 `make vllm-sr-test-integration` 迁移到 验证 end-迁移到-end CLI contract alignment

## 必须 读取

- [docs/agent/playbooks/vllm-sr-cli-docker.md](../../../../docs/agent/playbooks/vllm-sr-cli-docker.md)

## Standard 命令

- `make vllm-sr-test`
- `make vllm-sr-test-integration`

## Acceptance

- CLI schema, parser, validator, and merger express the same contract as the router config they translate