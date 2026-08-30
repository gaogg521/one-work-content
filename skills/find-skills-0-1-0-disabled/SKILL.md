---
name: find-skills-0-1-0-disabled
description: 帮助用户发现和安装Agent Skill。当用户询问\"如何做某事\"、\"查找某功能的Skill\"、\"是否有Skill可以...\"或表达扩展能力的意愿时触发。适用于用户寻找可能以可安装Skill形式存在的功能场景。
---

# 查找 Skills

This skill helps you discover and 安装 skills from the open agent skills ecosystem.

## When 迁移到 Use This Skill

何时使用此技能 the user:

- Asks "how do I do X" where X 也许 be a common 任务 with an existing skill
- Says "查找 a skill for X" or "is there a skill for X"
- Asks "可以 you do X" where X is a specialized capability
- Expresses interest in extending agent 能力
- Wants 迁移到 搜索 tools, templates, or workflows
- Mentions they wish they had help with a specific domain (设计, testing, deployment, etc.)

## What is the Skills CLI?

The Skills CLI (`npx skills`) is the 包 manager for the open agent skills ecosystem. Skills are modular packages that extend agent 能力 with specialized knowledge, workflows, and tools.

**键 命令:**

- `npx skills find [query]` - 搜索 skills interactively or by keyword
- `npx skills add <package>` - 安装 a skill from GitHub or other sources
- `npx skills check` - 检查 for skill updates
- `npx skills update` - 更新 all installed skills

**Browse skills at:** https://skills.sh/

## How 迁移到 Help Users 查找 Skills

### Step 1: Understand What They 需要

When a user asks for help with something, identify:

1. The domain (e.g., React, testing, 设计, deployment)
2. The specific 任务 (e.g., writing tests, creating animations, reviewing PRs)
3. Whether this is a common enough 任务 that a skill likely exists

### Step 2: 搜索 Skills

运行 the 查找 命令 with a relevant query:

```bash
npx skills find [query]
```

For 示例:

- User asks "how do I make my React app faster?" → `npx skills find react performance`
- User asks "可以 you help me with PR reviews?" → `npx skills find pr review`
- User asks "I 需要 迁移到 创建 a 变更日志" → `npx skills find changelog`

The 命令 将 返回 结果 like:

```
Install with npx skills add <owner/repo@skill>

vercel-labs/agent-skills@vercel-react-best-practices
└ https://skills.sh/vercel-labs/agent-skills/vercel-react-best-practices
```

### Step 3: Present 选项 迁移到 the User

When you 查找 relevant skills, present them 迁移到 the user with:

1. The skill name and 功能
2. The 安装 命令 they 可以 运行
3. A link 迁移到 learn more at skills.sh

示例 响应:

```
I found a skill that might help! The "vercel-react-best-practices" skill provides
React and Next.js performance optimization guidelines from Vercel Engineering.

To install it:
npx skills add vercel-labs/agent-skills@vercel-react-best-practices

Learn more: https://skills.sh/vercel-labs/agent-skills/vercel-react-best-practices
```

### Step 4: Offer 迁移到 安装

If the user wants 迁移到 proceed, you 可以 安装 the skill for them:

```bash
npx skills add <owner/repo@skill> -g -y
```

The `-g` flag installs globally (user__CODE__CODE`-y`DE`-y`el`-y`d `-y` skips confirmation prompts.

## Common Skill Categories

When searching, consider these common categories:

| Category        | 示例 Queries                          |
| --------------- | ---------------------------------------- |
| Web Development | react, nextjs, typescript, css, tailwind |
| Testing         | testing, jest, playwright, e2e           |
| DevOps          | 部署, docker, kubernetes, ci-cd        |
| Documentation   | docs, readme, 变更日志, api-docs        |
| Code Quality    | review, lint, 重构, best-practices   |
| 设计          | ui, ux, 设计-system, 无障碍     |
| Productivity    | workflow, automation, git                |

## 提示 for Effective Searches

1. **Use specific keywords**: "react testing" is better than just "testing"
2. **Try alternative terms**: If "部署" doesn't work, try "deployment" or "ci-cd"
3. **检查 popular sources**: Many skills come from `vercel-labs/agent-skills` or `ComposioHQ/awes__CODE_1__me-claude-skills``me-claude-skills`

## When No Skills Are Found

If no relevant skills exist:

1. Acknowledge that no existing skill was found
2. Offer 迁移到 help with the 任务 directly using your general 能力
3. Suggest the user 可能 创建 their own skill with `npx skills init`

示例:

```
I searched for skills related to "xyz" but didn't find any matches.
I can still help you with this task directly! Would you like me to proceed?

If this is something you do often, you could create your own skill:
npx skills init my-xyz-skill
```