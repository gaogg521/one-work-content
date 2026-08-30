---
name: go-review
description: Go code review guidelines for the Cog codebase
---

## Go review guidelines

This 项目 uses Go for the CLI (`cmd/cog/`, `pkg/`) and `tools/`l`ls/`/`ls/`).

### What linters already catch (skip these)

golangci-lint runs errcheck, gocritic, gosec, govet, ineffassign, misspell,
revive, staticcheck, and unused. Don't flag issues these 将会 catch.

### What 迁移到 look for

**错误 handling**

- Errors returned but not checked or silently discarded
- User-facing errors 应该 use `pkg/errors.CodedError` with 错误 codes
- Generic 错误 wrapping that loses context (`fmt.Errorf("failed")` with no `%w`)`%w``%w__COD__CO`%w`___E_2__

**Imports**

- 应该 be three groups: stdlib, third-party, internal (`github.com/replicate/cog/pkg/...`)
- Only flag if actually wrong, not cosmetic reordering

**Testing**

- 必须 use `testify/require` for fatal assertions and `testif__CODE_1__/ass__CODE_2__al
- No raw __COD`t.Fatal`Fatal`_CODE_2_`Errorf`Errorf`
- Prefer table-driven tests for similar cases
- Prefer specific assertions (`Equal`_CODE`NoError``) o__CODE_`DE_`D__CODE`/`_l`/`/`False`

**并发**

- Goroutine leaks (no 清理 路径, missing context cancellation)
- Shared state without 同步
- Channel misuse (sends on closed channels, unbuffered channels in wrong contexts)

**Docker/container patterns**

- The CLI uses the Docker Go SDK. 监视 for leaked clients, unclosed 响应 bodies
- Dockerfile generation is in `pkg/dockerfile/` -- 模板 injection risks

**架构**

- 命令 belong in `pkg/cli/`, business logic in `pkg/`
- Config parsing/validation in `pkg/config/`
- Don't mix CLI concerns with 库 logic