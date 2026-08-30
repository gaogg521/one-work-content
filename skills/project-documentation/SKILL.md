---
name: project-documentation
model: standard
description: 项目文档完整工作流，包括架构决策记录（ADR）、产品需求文档（PRD）、用户画像和文档组织。当为新项目设置文档或改进现有文档时触发。触发词：项目文档、ADR、PRD、用户画像、文档结构、文档配置。
---

# Project Documentation (Meta-Skill)

完成 workflow for setting up and maintaining project documentation.


## 安装

### OpenClaw / Moltbot / Clawbot

```bash
npx clawhub@latest install project-documentation
```


---

## When 迁移到 Use

- Starting a new project and 需要 docs structure
- Improving documentation on existing project
- Setting up ADRs, PRDs, or persona docs
- Want consistent documentation across projects

---

## Docs-First Philosophy

启动 every project with documentation, not code:

```
1. Define the idea      → What is this? What problem does it solve?
2. Define the personas  → Who uses this? What are their journeys?
3. Define the features  → What does it do for each persona?
4. Define the stack     → What technologies? Why?
5. Then build           → With full context established
```

---

## Directory Structure

```
docs/
├── architecture/        # CURRENT STATE - Living docs of actual code
│   ├── overview.md
│   └── data-flow.md
├── guides/              # CURRENT STATE - How to use/operate
│   ├── getting-started.md
│   └── configuration.md
├── runbooks/            # CURRENT STATE - Short, actionable guides
│   ├── local-dev.md
│   ├── deploy.md
│   └── database.md
├── planning/            # FUTURE - Not for docs site
│   ├── roadmap.md
│   └── specs/
├── decisions/           # ADRs - Decision records
│   ├── 001-tech-stack.md
│   └── 002-auth-approach.md
└── product/             # PRD, personas
    ├── overview.md
    ├── personas/
    └── features.md
```

---

## Critical Separation: Current vs Future

| Category | Purpose | Goes on Docs Site? |
|----------|---------|-------------------|
| **Current State** | How things work now | Yes |
| **Planning** | Future specs, designs | No |
| **Architecture** | Living docs of code | Yes |
| **Roadmap/Todos** | What we're working on | No |
| **Runbooks** | How 迁移到 operate | Yes |
| **Proposed Runbooks** | Future plans | No |

---

## Documentation Types

### Architecture Decision Records (ADRs)

Template:
```markdown
# ADR-001: [Title]

## Status
[Proposed | Accepted | Deprecated | Superseded]

## Context
[What is the issue we're solving?]

## Decision
[What did we decide?]

## Consequences
[What are the results - positive and negative?]

## Alternatives Considered
[What other options did we evaluate?]
```

### Product 环境要求 Document (PRD)

Template:
```markdown
# PRD: [Feature Name]

## Problem
[What problem are we solving?]

## Users
[Which personas does this serve?]

## Requirements
- [ ] Requirement 1
- [ ] Requirement 2

## Non-Goals
[What are we explicitly NOT doing?]

## Success Metrics
[How do we know this worked?]
```

### Persona Documentation

Template:
```markdown
# Persona: [Name]

## Who They Are
- Background
- Technical level
- Goals

## Pain Points
- [Pain 1]
- [Pain 2]

## Journey
1. Discovery
2. Onboarding
3. Daily use
4. Advanced usage

## Content Needs
- Doc types they need
- Format preferences
```

### Runbooks

Template:
```markdown
# Runbook: [Task Name]

## Prerequisites
- [Requirement 1]
- [Requirement 2]

## Steps
1. [Step 1]
2. [Step 2]

## Verify
[How to confirm success]

## Troubleshooting
| Problem | Solution |
|---------|----------|
| [Issue] | [Fix] |
```

---

## Roadmap Format

```markdown
## Roadmap

### Current Sprint
- [ ] Add user authentication endpoint
- [ ] Create login form component
- [ ] Wire form to auth endpoint

### Backlog
- [ ] Password reset flow
- [ ] OAuth integration
- [ ] Two-factor auth
```

---

## Quality Gates

Before shipping docs:

- [ ] Separates current state from planning
- [ ] Uses appropriate template for doc type
- [ ] Written for the right audience
- [ ] Actionable (runbooks) or explanatory (guides)
- [ ] No stale/outdated information

---

## Anti-Patterns

- **Mixing future plans with current state** — Confuses what's real
- **Planning docs on docs site** — Users expect reality
- **One-size-fits-all docs** — Different audiences 需要 different depth
- **Building 功能特性 before personas** — No context for decisions
- **Documentation written once and forgotten** — Keep it current

---

## Checklist for New Projects

- [ ] 创建 docs/ directory structure
- [ ] 写入 initial PRD/概述
- [ ] Document 2-3 personas
- [ ] 创建 ADR-001 for tech stack
- [ ] Set up roadmap format
- [ ] 创建 essential runbooks (local-dev, 部署)
- [ ] Separate planning/ from current-state docs

---

## Related Skills

- **命令:** [/bootstrap-docs](../../../命令/bootstrap/bootstrap-docs.md), [/new-feature](../../../命令/development/new-feature.md)
- **Agent:** [development](../../../agents/development/)