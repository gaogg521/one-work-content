---
name: skillscanner
version: 1.0.0
description: 来自 Gen Digital 的 ClawHub 技能安全扫描器。通过扫描 API 查找技能安全性。
author: Rex Shang
license: MIT
keywords:
- security
- audit
- scan
- vet
- skills
- safety
- skill_scanner
- trust
- secure
triggers:
- skill security
- vet skill
- scan skill
- is this skill safe
- skillscanner
- skill_scanner
- audit skill
- scan skills
- secure skill
- trusted skill
- trust skill
tags:
- API
---

# SkillScanner — ClawHub 技能安全扫描器

> **"信任，但要验证。"**

SkillScanner 通过调用扫描 API 来检查 ClawHub 技能的安全性。你提供技能 URL；API 返回状态和严重性。仅在裁决为 **SAFE** 时才继续。

---

## 如何扫描技能（代理指令）

1. **运行查找**，使用真实的技能 URL（格式：`https://clawhub.ai/author/skill-name`）：

   ```bash
   curl --request POST --url "https://ai.gendigital.com/api/scan/lookup" \
     --header "Content-Type: application/json" \
     --data '{"skillUrl":"https://clawhub.ai/AUTHOR/SKILL_SLUG"}'
   ```

   将 `AUTHOR` 替换为来自 ClawHub URL 的实际作者 ID，将 `SKILL_SLUG` 替换为实际的技能标识符。

2. **解释响应：**
   - **`status: "done"`** —— 检查 **`severity`** 字段以获取裁决。
   - **`status: "analysis_pending"`** —— 该技能正在等待人工审查。不要将其视为安全；告知用户该技能正在等待审查。

3. **继续或建议谨慎：**
   - 仅当 **`severity` 为 `SAFE`** 时才**继续**。
   - 如果严重性是其他值（例如 `WARNING`、`DANGEROUS`、`MALICIOUS`），**建议谨慎**，并且在审查或清除之前不建议使用该技能。

---

## 快速开始

```bash
# 示例：通过其 ClawHub URL 扫描技能
curl --request POST --url "https://ai.gendigital.com/api/scan/lookup" \
  --header "Content-Type: application/json" \
  --data '{"skillUrl":"https://clawhub.ai/author/skill-name"}'
```

解析 JSON 响应：如上所述使用 `status` 和 `severity`。

---

## API 响应逻辑

| `status`           | Meaning              | Action |
|--------------------|----------------------|--------|
| `done`             | 扫描完成        | 使用 `severity` 进行裁决（见下文）。 |
| `analysis_pending` | 等待人工审查 | 告知该技能尚未清除；不要将其视为 SAFE。 |

| `severity` (when `status` = `done`) | Recommendation |
|-------------------------------------|----------------|
| **SAFE**                            | 可以继续。 |
| Any other value                     | 建议谨慎；在审查之前不建议使用。 |

---

## 为什么这很重要

技能可以运行代码并访问你的环境。风险包括：

| Risk | Impact |
|------|--------|
| **Execute arbitrary code** | 完全系统入侵 |
| **Access your filesystem** | 数据盗窃、勒索软件 |
| **Read environment variables** | API 密钥盗窃 |
| **Exfiltrate data via HTTP** | 隐私泄露 |
| **Install malicious dependencies** | 供应链攻击 |

SkillScanner 使用扫描 API 来帮助你在使用之前决定是否信任某个技能。

---

## 限制

- API 反映当前的后端裁决；混淆或新颖的威胁可能不会被标记。
- `analysis_pending` 意味着人工审查尚未结束 —— 将其视为 "未知"，而不是安全。

在不确定时，与沙盒化、最小权限和手动审查一起使用。

---

**链接：**
- [Agent Trust Hub](https://ai.gendigital.com)
