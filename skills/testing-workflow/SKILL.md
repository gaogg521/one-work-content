---
name: testing-workflow
model: standard
category: testing
description: 元技能(meta-skill)，协调 testing-patterns、e2e-testing 与 testing agents 编排项目全面测试。适用于为新项目从零设置测试、为现有项目补全覆盖率、建立测试策略或发布前验证质量。
version: 1.0
tags:
- 测试
- AI
- 元数据
- 模式
---

# Testing Workflow

通过协调 **testing-patterns** skill、**e2e-testing** skill 和 testing agents 来编排项目中的全面测试。此 meta-skill 本身不定义测试模式——它在每个阶段路由到正确的 skill 或 agent，并确保没有遗漏。

---

## When to Use

- 从零开始为新项目设置测试
- 为存在缺口的现有项目提升覆盖率
- 建立或修订测试策略
- 在重大发布前验证 quality gates 是否满足
- 在大型重构后确认没有破坏任何东西
- 在代码审查时当测试充分性存疑
- 在团队 onboarding 测试工作流时

---

## Orchestration Flow

按顺序执行以下步骤。每一步都路由到一个特定的 skill 或 agent——在进入下一步之前，阅读并应用该资源。

### Phase 1: Discovery and Baseline

扫描项目以了解现有测试基础设施，测量当前覆盖率，并在进行更改前识别缺口。没有 baseline，你就无法证明改进。

1. **Identify test infrastructure** — 确定已在使用的 test runner、assertion library、coverage tool 和 CI configuration。如果不存在，标记需要 setup。
2. **Measure current coverage** — 运行现有测试套件并记录 statement、branch 和 function coverage。这是 baseline。
3. **Map untested code** — 识别没有测试覆盖的 modules、functions 和 code paths。按风险排序：business-critical logic 优先，utilities 最后。
4. **Catalog existing tests** — 将现有测试分类为 unit、integration 或 E2E。检查 skipped tests、flaky tests 以及不 assert 任何有意义内容的测试。

### Phase 2: Strategy Selection

基于 discovery 结果，为该项目选择合适的测试方法。

1. **Determine project type** — 使用下方的 Coverage Targets 表格为项目类型设置适当的 thresholds。
2. **Select test patterns** — 阅读 `ai/skills/testing/testing-patterns/SKILL.md` 并选择与项目架构、语言和框架匹配的 unit/integration test patterns。
3. **Identify critical user journeys** — 列出 3-10 个最重要的需要 E2E coverage 的用户工作流。这些是失败会直接影响 revenue、user trust 或 safety 的流程。
4. **Document the strategy** — 填写下方的 Testing Strategy Template 并将其 commit 到仓库。

### Phase 3: Implementation

按照 Phase 2 中选择的模式生成测试。

1. **Unit tests first** — 为未覆盖的 business logic 编写 unit tests，从最高风险的 modules 开始。遵循 testing pyramid：你的测试中约 70% 应为 unit tests。
2. **Integration tests next** — 为 module boundaries、API endpoints 和 database queries 编写 integration tests。重点关注 components 交互的 seams。
3. **E2E tests for critical journeys** — 阅读 `ai/skills/testing/e2e-testing/SKILL.md` 并为 Phase 2 中识别的每个 critical user journey 编写 E2E tests。
4. **Edge case coverage** — 在 happy paths 被覆盖后，为 error conditions、boundary values、null/empty inputs 和 concurrency scenarios 添加测试。

### Phase 4: Validation

验证新测试是否满足质量标准和覆盖率目标。

1. **Run the full test suite** — 每个测试都必须通过。在继续之前修复失败。
2. **Measure coverage against targets** — 将新覆盖率与项目类型的 thresholds 进行比较。如果未达到目标，返回 Phase 3。
3. **Check test quality** — 检查测试是否存在 testing-patterns 中列出的 anti-patterns（assert-free tests、overmocking、flaky tests、test pollution）。修复发现的任何问题。
4. **Verify CI integration** — 确认测试在每次 push/PR 时自动运行，并且 coverage thresholds 在 CI 中被强制执行。

### Phase 5: Maintenance

建立持续实践以保持测试套件健康。

1. **Set up coverage ratcheting** — 配置 CI，如果覆盖率低于当前水平则失败。覆盖率只应上升。
2. **Establish flaky test policy** — 任何间歇性失败的测试必须在一个 sprint 内修复，或附上理由移除。
3. **Define test review standards** — 每个添加或更改逻辑的 PR 都必须包含相应的测试更改。Reviewers 会检查这一点。
4. **Schedule test health audits** — 每季度审查 test execution time、flaky test rate、skipped test count 和 coverage trends。

---

## Skill Routing Table

使用此表将特定需求路由到正确的资源：

| Need | Route To | Path |
|------|----------|------|
| Unit/integration test patterns | testing-patterns | `ai/skills/testing/testing-patterns/SKILL.md` |
| E2E test patterns | e2e-testing | `ai/skills/testing/e2e-testing/SKILL.md` |
| Code quality standards | clean-code | `ai/skills/testing/clean-code/SKILL.md` |
| Review checklist | code-review | `ai/skills/testing/code-review/SKILL.md` |
| CI/CD quality gates | quality-gates | `ai/skills/testing/quality-gates/SKILL.md` |
| Debugging test failures | debugging | `ai/skills/testing/debugging/SKILL.md` |

当请求明确属于某一行时，直接前往该资源。仅当以全面覆盖为目标时，才使用完整的 orchestration flow。

---

## Coverage Targets

Targets 因项目类型而异。使用适当的行来设定期望：

| Project Type | Statement | Branch | Function | E2E Journeys | Notes |
|--------------|-----------|--------|----------|--------------|-------|
| Startup MVP | 60% | 50% | 60% | Top 3 flows | Focus on critical paths only |
| Production App | 80% | 70% | 80% | Top 10 flows | Balance speed with confidence |
| Library / Package | 90% | 85% | 95% | N/A | Public API must be fully covered |
| Critical Infrastructure | 95% | 90% | 95% | All flows | Zero tolerance for gaps |

这些是最低要求。在时间允许的情况下争取更高，但不要因 vanity metrics 而阻塞发布——优先关注有意义的 coverage 而非百分点。

---

## Testing Strategy Template

使用此模板为项目记录测试策略。在 orchestration flow 期间填写它，并将其保留在仓库中。

```markdown
# Testing Strategy

## Project Overview
- **Project**: [name]
- **Type**: [startup MVP | production app | library | critical infrastructure]
- **Primary Language**: [language]
- **Framework**: [framework]
- **Test Runner**: [runner]
- **Coverage Tool**: [tool]

## Coverage Baseline
- **Statement**: [X%]
- **Branch**: [X%]
- **Function**: [X%]
- **E2E Journeys Covered**: [N of M]
- **Date Measured**: [YYYY-MM-DD]

## Coverage Targets
- **Statement**: [target%]
- **Branch**: [target%]
- **Function**: [target%]
- **E2E Journeys**: [target count]

## Test Patterns Selected
- [ ] [Pattern 1 — reason for selection]
- [ ] [Pattern 2 — reason for selection]
- [ ] [Pattern 3 — reason for selection]

## Critical User Journeys (E2E)
1. [Journey 1 — e.g., signup -> onboarding -> first action]
2. [Journey 2 — e.g., login -> dashboard -> export]
3. [Journey 3 — e.g., checkout -> payment -> confirmation]

## Gaps and Risks
- [Untested area 1 — risk level, mitigation plan]
- [Untested area 2 — risk level, mitigation plan]

## Quality Gate Status
- [ ] All tests pass
- [ ] Coverage targets met
- [ ] Critical journeys covered with E2E
- [ ] No skipped tests without justification
- [ ] Test execution time within budget
- [ ] CI enforces coverage thresholds
```

---

## Quality Gates for Testing Completion

在标记测试完成之前，必须满足以下所有条件：

| Gate | Requirement | Why |
|------|------------|-----|
| **All tests pass** | Zero failures, zero errors | Flaky tests count as failures |
| **Coverage targets met** | Statement、branch 和 function coverage 达到项目类型的 thresholds | Untested code is unverified code |
| **Critical journeys covered** | 每个 critical user journey 都有一个通过的 E2E test | Revenue and trust depend on these flows |
| **No unjustified skips** | 每个 `skip`、`xit` 或 `xdescribe` 都有 comment 和 linked issue | Skipped tests rot into permanent gaps |
| **Execution time budget** | Unit < 60s, E2E < 10min | Slow suites get skipped by developers |
| **No test pollution** | 单独运行任何测试文件与运行完整套件产生相同结果 | Shared state masks failures |
| **Mocks are justified** | 每个 mock 都有 comment 解释为什么不能使用 real impl | Over-mocking hides real bugs |

---

## NEVER Do

1. **NEVER write tests that test implementation details instead of behavior** — 测试必须验证代码做什么，而不是怎么做
2. **NEVER skip the discovery phase** — 在编写新测试之前始终测量 baseline，否则你无法证明改进
3. **NEVER merge tests that depend on execution order** — 每个测试必须是独立的和 idempotent 的
4. **NEVER mock what you do not own** — 将第三方依赖包装在你自己的 adapters 中，并 mock adapters 而不是第三方本身
5. **NEVER treat coverage percentage as the sole quality metric** — 100% coverage 配合 weak assertions 比 70% coverage 配合 strong assertions 更糟
6. **NEVER leave the test suite in a failing state** — 如果测试失败，修复它或附上理由移除它，然后再继续
7. **NEVER skip E2E tests for critical user journeys** — 仅靠 unit tests 无法 catch 最重要的流程中的 integration failures
8. **NEVER deploy without running the full test suite** — 部分测试运行会产生 false confidence
