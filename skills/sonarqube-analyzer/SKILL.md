---
name: sonarqube-analyzer
description: 分析自托管 SonarQube 项目，获取代码问题(issues)并建议自动化修复方案。支持 Quality Gate 检查与 PR 级分析。触发词：SonarQube 分析、代码质量(code quality)、issue 修复、Quality Gate、静态分析(static analysis)
tags:
- Git
- 代码审查
---

# SonarQube Analyzer Skill

分析自托管 SonarQube 上的项目，获取 issues 并建议自动化解决方案。

## 注册的工具

### `sonar_get_issues`
从 SonarQube 获取项目/PR 的 issues 列表。

**参数：**
- `projectKey` (string, 必需): 项目 key
- `pullRequest` (string, 可选): 用于特定分析的 PR 编号
- `severities` (string[], 可选): 要过滤的 severities (BLOCKER, CRITICAL, MAJOR, MINOR, INFO)
- `status` (string, 可选): issues 的状态 (OPEN, CONFIRMED, FALSE_POSITIVE, 等)
- `limit` (number, 可选): issues 限制 (默认: 100)

**示例：**
```json
{
  "projectKey": "openclaw-panel",
  "pullRequest": "5",
  "severities": ["CRITICAL", "MAJOR"],
  "limit": 50
}
```

### `sonar_analyze_and_suggest`
基于 SonarQube rules 分析 issues 并建议解决方案。

**参数：**
- `projectKey` (string, 必需): 项目 key
- `pullRequest` (string, 可选): PR 编号
- `autoFix` (boolean, 可选): 尝试应用自动修复 (默认: false)

**示例：**
```json
{
  "projectKey": "openclaw-panel",
  "pullRequest": "5",
  "autoFix": false
}
```

### `sonar_quality_gate`
检查项目的 Quality Gate 状态。

**参数：**
- `projectKey` (string, 必需): 项目 key
- `pullRequest` (string, 可选): PR 编号

**示例：**
```json
{
  "projectKey": "openclaw-panel",
  "pullRequest": "5"
}
```

## 配置

此 skill 使用以下环境配置：

```bash
SONAR_HOST_URL=http://127.0.0.1:9000  # SonarQube 的 URL
SONAR_TOKEN=admin                      # Authentication token
```

## 用法

### 分析一个特定的 PR：
```bash
node scripts/analyze.js --project=my-project --pr=5
```

### 生成 issues 报告：
```bash
node scripts/report.js --project=my-project --format=markdown
```

### 检查 Quality Gate：
```bash
node scripts/quality-gate.js --project=my-project --pr=5
```

## 响应结构

### sonar_get_issues
```json
{
  "total": 12,
  "issues": [
    {
      "key": "...",
      "severity": "MAJOR",
      "component": "apps/web/src/ui/App.tsx",
      "line": 346,
      "message": "Extract this nested ternary...",
      "rule": "typescript:S3358",
      "status": "OPEN",
      "solution": "Extract nested ternary into a separate function..."
    }
  ],
  "summary": {
    "BLOCKER": 0,
    "CRITICAL": 0,
    "MAJOR": 2,
    "MINOR": 10,
    "INFO": 0
  }
}
```

### sonar_analyze_and_suggest
```json
{
  "projectKey": "openclaw-panel",
  "analysis": {
    "totalIssues": 12,
    "fixableAutomatically": 8,
    "requiresManualFix": 4
  },
  "suggestions": [
    {
      "file": "apps/web/src/ui/App.tsx",
      "line": 346,
      "issue": "Nested ternary operation",
      "suggestion": "Extract into independent component",
      "codeExample": "...",
      "autoFixable": false
    }
  ],
  "nextSteps": [
    "Run lint:fix for auto-fixable issues",
    "Refactor nested ternaries in App.tsx",
    "Replace || with ?? operators"
  ]
}
```

## 可用的自动解决方案

| Rule | 问题 | 自动解决方案 |
|-------|----------|-------------------|
| S6606 | 使用 `\|\|` 代替 `??` | ✅ 替换为 `??` |
| S3358 | Nested ternary | ❌ 需要手动重构 |
| S6749 | Redundant fragment | ✅ 移除 fragment |
| S6759 | Non-readonly props | ✅ 添加 `readonly` |
| S3776 | Cognitive complexity | ❌ 需要提取组件 |
| S6571 | `any` in union type | ✅ 移除冗余 |

## 要求

- Node.js 18+
- 访问 SonarQube (localhost:9000)
- 配置好的 authentication token

## 与 Workflows 集成

在 GitHub Actions 中使用示例：

```yaml
- name: Analyze with SonarQube Skill
  run: |
    npm install -g @felipeoff/sonarqube-analyzer
    sonarqube-analyzer \
      --project=my-project \
      --pr=${{ github.event.pull_request.number }} \
      --suggest-fixes
```