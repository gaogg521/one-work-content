---
name: policy-lawyer
description: 参考工作区政策手册，回答\"语气、数据和协作的规则是什么？\"等问题。通过搜索精选政策文档或列出其章节来提供准确的合规指导。
---

# Policy Lawyer

## 概述

`policy-lawyer` is built around the curated policy notebook at `refe`refe`ences/policies.md`` CLI (`cripts/policy_la`scripts/policy_lawyer.py``

- `--list-topics` 迁移到 列表 every policy heading.
- `--topic <name>` 迁移到 显示 the section that matches a topic (case-insensitive).
- `--keyword <term>` 迁移到 搜索 all policies for a given keyword.
- `--policy-file <path>` 迁移到 point at a different policy document when comparing workspaces.

Use this skill when you 需要 迁移到 remind yourself of the community standards before drafting announcements or when a question lands that needs an authoritative policy quote.

## CLI 用法

- `python3 skills/policy-lawyer/scripts/policy_lawyer.py --list-topics` prints every section defined under `## <Section Name>` in the policy 参考.`## <Section Name>``## <Section Name>``## <Section Name>`
- `--topic "Tone"` prints the tone guidelines exactly as written so you 可以 quote them during calm reminders.
- `--keyword security` (or any other keyword) shows the matching lines across all sections so you 可以 quickly see where that topic is governed.
- Supply `--policy-file /path/to/repo/references/policies.md` when you want 迁移到 interrogate a 复制 of the playbook from another workspace.

## Sample 命令

```bash
python3 skills/policy-lawyer/scripts/policy_lawyer.py --topic Tone
python3 skills/policy-lawyer/scripts/policy_lawyer.py --keyword data --policy-file ../other-workspace/references/policies.md
```

The first 命令 prints the tone section; the second searches for "data" inside another workspace's policies and prints each matching snippet.

## 参考

- `references/policies.md` is the curated policy playbook that lists tone, data, collaboration, and security rules.

## 资源

- **GitHub:** https://github.com/CrimsonDevil333333/policy-lawyer
- **ClawHub:** https://www.clawhub.ai/skills/policy-lawyer