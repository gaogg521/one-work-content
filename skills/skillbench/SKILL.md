---
name: skillbench
description: 追踪技能版本、基准测试性能、对比改进并获取自我改进信号。与 tasktime 和 ClawVault 集成。
metadata: None
openclaw: None
install:
- bins:
  - skillbench
  id: node
  kind: node
  label: 安装 SkillBench CLI (npm)
  package: '@versatly/skillbench'
requires: None
bins:
- skillbench
tags:
- Vault
---

# skillbench Skill

**面向 AI 智能体的自我改进技能生态系统。**

追踪技能版本、基准测试性能、对比改进，并获取下一步修复的信号。

**[ClawVault](https://clawvault.dev) 生态系统的一部分** | [tasktime](https://clawhub.com/skills/tasktime) | [ClawHub](https://clawhub.com)

## 安装

```bash
npm install -g @versatly/skillbench
```

## 循环

```
1. 使用技能    → skillbench use github@1.0.0
2. 执行任务    → tt start "Create PR" && ... && tt stop
3. 记录结果    → skillbench record "Create PR" --success
4. 查看分数   → skillbench score github
5. 改进技能    → 更新技能，提升版本
6. 重复         → 对比 v1.0.0 与 v1.1.0
```

## 命令

### 追踪技能
```bash
skillbench use github@1.2.0            # 设置活跃技能版本
skillbench skills                       # 列出已追踪的技能 + 信号
```

### 记录基准测试
```bash
# 自动从 tasktime 拉取持续时间
skillbench record "Create PR" --success

# 手动指定持续时间
skillbench record "Create PR" --duration 45s --success

# 记录失败
skillbench record "Create PR" --fail --error-type "auth-error"
```

### 评分与对比
```bash
skillbench score                        # 所有技能及其等级
skillbench score github                 # 单个技能
skillbench compare github@1.0.0 github@1.1.0
```

### 导出与仪表盘
```bash
skillbench export --format markdown
skillbench export --format json
skillbench dashboard                    # 生成 HTML 仪表盘
skillbench dashboard --open             # 生成并在浏览器中打开
```

### 自动化测试
```bash
skillbench test tasktime@1.1.0          # 运行冒烟测试
skillbench test tasktime@1.1.0 --suite full  # 运行指定测试套件
skillbench test tasktime@1.1.0 --dry-run     # 试运行，不记录
```

### 同步
```bash
skillbench sync --clawhub               # 导入已安装的技能
skillbench sync --vault                 # 同步到 ClawVault
skillbench sync --all                   # 全部同步
```

### 健康与监控
```bash
skillbench health                       # 整体健康报告及告警
skillbench watch --once                 # 一次性运行所有测试套件
skillbench watch --interval 300         # 每 5 分钟持续监控
```

### 分析与改进
```bash
skillbench improve                      # 获取最弱技能的改进建议
skillbench improve github               # 针对特定技能的改进计划
skillbench trend tasktime --days 30     # 随时间的性能趋势
skillbench leaderboard                  # 对比智能体（多智能体环境）
skillbench schedule --interval 60       # 生成自动测试的 cron 配置
```

### 基线与回归检测
```bash
skillbench baseline tasktime --set      # 根据当前性能设置基线
skillbench baseline --list              # 列出所有基线
skillbench baseline --check             # 检查所有基线（CI 友好，失败时退出码 1）
skillbench baseline tasktime --remove   # 移除基线
```

### CI/CD 集成
```bash
skillbench ci                           # 运行所有测试 + 基线检查
skillbench ci --json                    # JSON 输出，便于自动化
skillbench badge                        # 为 README 生成 shields.io 徽章
```

复制 `examples/github-action.yml` 以获取现成的 GitHub Actions 工作流。

## 评分系统

| 等级 | 分数 | 含义 |
|-------|-------|---------|
| 🏆 A+ | 95-100 | 精英表现 |
| ✅ A | 85-94 | 优秀 |
| 👍 B | 70-84 | 良好 |
| ⚠️ C | 50-69 | 需要改进 |
| ❌ D | <50 | 损坏 |

基于：成功率 (40%)、平均耗时 (30%)、一致性 (20%)、趋势 (10%)

## tasktime 集成

当你省略 `--duration` 时，skillbench 会从 [tasktime](https://clawhub.com/skills/tasktime) 拉取：

```bash
tt start "Create PR" -c git
# ... 执行工作 ...
tt stop
skillbench record --success   # 持续时间自动拉取
```

## ClawVault 集成

基准测试会自动同步到 [ClawVault](https://clawvault.dev)。

## 改进信号

`skillbench skills` 显示：
- ⚠️ **needs work** — 成功率低于 70%
- 🕐 **stale** — 超过 7 天没有基准测试
- ↘️ **declining** — 随时间推移表现变差

## 相关

- [ClawVault](https://clawvault.dev) — AI 智能体的记忆系统
- [tasktime](https://clawhub.com/skills/tasktime) — 任务计时 CLI
- [ClawHub](https://clawhub.com) — 技能市场
