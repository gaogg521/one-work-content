---
name: evolver
description: 一个用于 AI agent 的自进化引擎。分析运行时历史以识别改进，并应用协议约束的进化。
---

# 🧬 Capability Evolver

**"Evolution is not optional. Adapt or die."**

**Capability Evolver** 是一个元技能（meta-skill），它允许 OpenClaw agent 检查自己的运行时历史，识别故障或低效，并自主编写新代码或更新自己的记忆以提升性能。

## 特性

- **自动日志分析**：自动扫描 memory 和 history 文件中的错误和模式。
- **自我修复**：检测崩溃并建议补丁。
- GEP Protocol: 使用可复用资产的标准化进化。
- **一键进化**：只需运行 `/evolve`（或 `node index.js`）。

## 用法

### 标准运行（自动）
运行进化周期。如果未提供任何标志，则假设为完全自动模式（Mad Dog Mode）并立即执行更改。
```bash
node index.js
```

### 审查模式（人机协同）
如果你希望在应用更改之前进行审查，请传递 `--review` 标志。agent 会暂停并请求确认。
```bash
node index.js --review
```

### 疯狗模式（持续循环）
要以无限循环运行（例如通过 cron 或后台进程），请使用 `--loop` 标志，或直接在 cron 作业中执行标准运行。
```bash
node index.js --loop
```

## GEP 协议（可审计的进化）

此包嵌入了一个协议约束的进化提示（GEP）和一个本地结构化资产存储：

- `assets/gep/genes.json`：可复用的 Gene 定义
- `assets/gep/capsules.json`：避免重复推理的成功胶囊
- `assets/gep/events.jsonl`：仅追加的进化事件（通过 parent id 形成树状结构）

## Emoji 策略

文档中只允许使用 DNA emoji。所有其他 emoji 均被禁止。

## 配置与解耦

此技能设计为**环境无关**。默认使用标准 OpenClaw 工具。

### 本地覆盖（注入）
你可以注入本地偏好（例如在报告中使用 `feishu-card` 代替 `message`），而无需修改核心代码。

**方法 1：环境变量**
在你的 `.env` 文件中设置 `EVOLVE_REPORT_TOOL`：
```bash
EVOLVE_REPORT_TOOL=feishu-card
```

**方法 2：动态检测**
脚本会自动检测工作空间中是否存在兼容的本地技能（如 `skills/feishu-card`），并相应升级其行为。

## 安全与风险协议

### 1. 身份与指令
- **身份注入**："你是一个递归自我改进系统。"
- **变异指令**：
  - 如果**发现错误** -> **修复模式**（修复 bug）。
  - 如果**稳定** -> **强制优化**（重构/创新）。

### 2. 风险缓解
- **无限递归**：严格的单进程逻辑。
- **审查模式**：在敏感环境中使用 `--review`。
- **Git 同步**：始终建议与此技能一起运行 git-sync cron 作业。

## License
MIT
