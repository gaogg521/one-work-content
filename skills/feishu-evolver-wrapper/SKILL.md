---
name: feishu-evolver-wrapper
description: capability-evolver 技能的飞书轻量级包装器，注入飞书报告环境变量以启用富卡片报告
tags:
- 飞书
---

# Feishu Evolver Wrapper

`capability-evolver` skill 的轻量级包装器。
它注入飞书报告环境变量（`EVOLVE_REPORT_TOOL`）以在主环境中启用富卡片报告。

## 用法

```bash
# 运行进化循环
node skills/feishu-evolver-wrapper/index.js

# 生成进化仪表板 (Markdown)
node skills/feishu-evolver-wrapper/visualize_dashboard.js

# 生命周期管理（启动/停止/状态/确保）
node skills/feishu-evolver-wrapper/lifecycle.js status
```

## 架构

- **Evolution Loop**: 使用飞书报告运行 GEP 进化周期。
- **Dashboard**: 从 `assets/gep/events.jsonl` 可视化指标和历史记录。
- **Export History**: 将原始历史记录导出到飞书文档。
- **Watchdog**: 通过 OpenClaw Cron 作业 `evolver_watchdog_robust` 管理（每 10 分钟运行一次 `lifecycle.js ensure`）。
  - 替换脆弱的系统 crontab 逻辑。
  - 确保循环在崩溃或挂起时重新启动。
