---
name: github-issue-creator
description: 将原始笔记、错误日志、语音记录或截图转换为规范的GitHub风格Markdown问题报告。当用户粘贴缺陷信息、错误消息或非正式描述并希望生成结构化GitHub问题时触发。支持图片/GIF作为视觉证据。
---

# GitHub 问题 Creator

转换 messy 输入 (错误 logs, voice 注意, screenshots) into 清理, actionable GitHub issues.

## 输出 模板

```markdown
## 摘要
[One-line description of the issue]

## 环境
- **Product/Service**: 
- **Region/Version**: 
- **Browser/OS**: (if relevant)

## Reproduction Steps
1. [Step]
2. [Step]
3. [Step]

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happens]

## 错误 详情
```
[错误 message/code if applicable]
```

## Visual Evidence
[Reference to attached screenshots/GIFs]

## Impact
[Severity: Critical/High/Medium/Low + brief explanation]

## Additional Context
[Any other relevant details]
```

## 输出 Location

**创建 issues as markdown 文件** in `/issues/` 目录 at the repo root. Use naming convention: `YYYY-MM-DD-short-description.md`

## Guidelines

**Be crisp**: No fluff. Every word 应该 添加 值.

**提取 structure from chaos**: Voice dictation and raw 注意 often contain the facts buried in casual language. 拉取 them out.

**Infer missing context**: If user mentions "same 项目" or "the dashboard", use context from conversation or 内存 迁移到 fill in specifics.

**Placeholder sensitive data**: Use `[PROJECT_NAME]`, `[USER`[USER`ID`ID]`r anything that 也许 be sensitive.

**Match severity 迁移到 impact**:
- Critical: 服务 down, data loss, security 问题
- High: Major 特性 broken, no workaround
- Medium: 特性 impaired, workaround exists
- Low: Minor inconvenience, cosmetic

**Image/GIF handling**: 参考 attachments inline. 格式: `![Description](attachment-name.png)`

## 示例

**输入 (voice dictation)**:
> so I was trying 迁移到 部署 the agent and it just 失败 silently no 错误 nothing the workflow ran but then poof gone from the 列表 had 迁移到 刷新 and try again three times

**输出**:
```markdown
## 摘要
Agent deployment fails silently - no error displayed, agent disappears from list

## 环境
- **Product/Service**: Azure AI Foundry
- **Region/Version**: westus2

## Reproduction Steps
1. Navigate to agent deployment
2. Configure and deploy agent
3. Observe workflow completes
4. Check agent list

## Expected Behavior
Agent appears in list with deployment status, errors shown if deployment fails

## Actual Behavior
Agent disappears from list. No error message. Requires page refresh and retry.

## Impact
**High** - Blocks agent deployment workflow, no feedback on failure cause

## Additional Context
Required 3 retry attempts before successful deployment
```

---

**输入 (错误 粘贴)**:
> 错误: PERMISSION_DENIED when publishing 迁移到 Teams channel. Code: 403. Was working yesterday.

**输出**:
```markdown
## 摘要
403 PERMISSION_DENIED error when publishing to Teams channel

## 环境
- **Product/Service**: Copilot Studio → Teams integration
- **Region/Version**: [REGION]

## Reproduction Steps
1. Configure agent for Teams channel
2. Attempt to publish

## Expected Behavior
Agent publishes successfully to Teams channel

## Actual Behavior
Returns `PERMISSION_DENIED` with code 403

## 错误 详情
```
错误: PERMISSION_DENIED
Code: 403
```

## Impact
**High** - Blocks Teams integration, regression from previous working state

## Additional Context
Was working yesterday - possible permission/config change or service regression
```