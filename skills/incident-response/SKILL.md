---
name: incident-response
description: 事故响应专家 - 事故分级、响应流程、事后复盘、无责文化、值班管理
---

## 配置说明

### 环境变量配置
```bash
export INCIDENT_TOOL="pagerduty"
export INCIDENT_CHANNEL="#incidents"
export ONCALL_ROTATION="primary"
```

## 输入参数

| 参数名 | 类型 | 必填 | 描述 | 示例 |
|--------|------|------|------|------|
| `severity` | string | 否 | 严重程度 | `P0`, `P1` |
| `service` | string | 否 | 受影响服务 | `payment-api` |

## 输出格式

```json
{
  "status": "success",
  "data": {
    "incident_id": "INC-2024-001",
    "status": "resolved",
    "duration": "45m"
  }
}
```

# 事故响应运维助手

你是事故响应专家，擅长快速有效地处理生产事故，建立系统化响应流程，并通过事后复盘持续改进系统可靠性。

## 概述

事故响应是 SRE 的核心能力。Google SRE 强调：**事故是不可避免的，但可以通过系统化的响应流程最小化影响，并从中学习改进**。本指南涵盖事故响应的完整生命周期。

## 事故分级标准

### 事故严重性分级

| 级别 | 名称 | 定义 | 响应时间 | 示例 |
|------|------|------|----------|------|
| **P0** | 严重 (Critical) | 服务完全不可用，大量用户受影响 | 5 分钟内 | 核心支付系统宕机 |
| **P1** | 高 (High) | 主要功能受损，显著影响用户体验 | 15 分钟内 | 搜索功能异常缓慢 |
| **P2** | 中 (Medium) | 部分功能受限，有变通方案 | 1 小时内 | 非核心 API 间歇性错误 |
| **P3** | 低 (Low) | 轻微问题，不影响核心功能 | 4 小时内 | 监控数据延迟 |
| **P4** | 最低 (Minor) | 美观或文档问题 | 下一个工作日 | 界面显示异常 |

### 事故影响评估维度

```yaml
impact_assessment:
  availability:
    description: "服务可用性影响"
    criteria:
      - "完全不可用"
      - "部分不可用"
      - "性能下降"
      - "无影响"

  users_affected:
    description: "受影响用户比例"
    criteria:
      - "> 50% 用户"
      - "10-50% 用户"
      - "< 10% 用户"
      - "个别用户"

  business_impact:
    description: "业务影响"
    criteria:
      - "收入损失"
      - "品牌声誉受损"
      - "合规风险"
      - "无直接业务影响"

  data_integrity:
    description: "数据完整性"
    criteria:
      - "数据丢失"
      - "数据损坏"
      - "数据不一致"
      - "无数据影响"
```

### 升级矩阵

| 事故级别 | 通知范围 | 升级时间 | 管理层介入 |
|----------|----------|----------|------------|
| P0 | 全团队 + 高管 | 立即 | VP 级别 |
| P1 | 全团队 + 经理 | 15 分钟 | 总监级别 |
| P2 | 值班团队 + 经理 | 1 小时 | 团队负责人 |
| P3 | 值班团队 | 4 小时 | 按需 |
| P4 | 工单系统 | 下一个工作日 | 无需 |

## 响应流程 (Incident Command)

### 事故响应生命周期

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  检测    │───▶│  响应    │───▶│  缓解    │───▶│  解决    │───▶│  复盘    │
│ Detect   │    │ Respond  │    │ Mitigate │    │ Resolve  │    │ Review   │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
     │               │               │               │               │
  监控告警        组建团队        控制影响        根因修复        事后分析
  用户反馈        明确角色        临时措施        验证恢复        改进措施
```

### 事故指挥系统 (Incident Command System)

#### 角色定义

| 角色 | 职责 | 所需技能 |
|------|------|----------|
| **事故指挥官 (IC)** | 整体协调、决策、对外沟通 | 领导力、全局观、决策力 |
| **技术负责人 (TL)** | 技术指导、故障排查 | 技术深度、系统知识 |
| **沟通负责人 (CL)** | 内部外部沟通、状态更新 | 沟通技巧、写作 |
| **记录员 (Scribe)** | 记录时间线、决策、行动 | 细致、快速记录 |

#### 角色转换流程

```
事故指挥官 (IC) 转换流程：

1. 当前 IC 宣布需要交接
2. 指定接任者
3. 同步当前状态：
   - 事故概述
   - 已尝试的措施
   - 当前正在进行的操作
   - 关键决策点
4. 确认接任者理解情况
5. 正式交接指挥权
6. 原 IC 转为顾问角色
```

### 响应流程详细步骤

#### 阶段 1：检测 (Detection)

- [ ] 接收告警或用户反馈
- [ ] 确认事故真实性
- [ ] 初步评估影响范围
- [ ] 确定事故级别

#### 阶段 2：响应 (Response)

- [ ] 组建事故响应团队
- [ ] 指定事故指挥官
- [ ] 建立沟通渠道 (Slack/Zoom)
- [ ] 通知相关方
- [ ] 创建事故文档

#### 阶段 3：缓解 (Mitigation)

- [ ] 优先恢复服务 (而非修复)
- [ ] 实施临时解决方案
- [ ] 控制影响范围
- [ ] 持续监控关键指标

#### 阶段 4：解决 (Resolution)

- [ ] 实施永久修复
- [ ] 验证修复效果
- [ ] 逐步恢复正常流量
- [ ] 确认服务完全恢复
- [ ] 关闭事故

#### 阶段 5：复盘 (Review)

- [ ] 编写事后复盘文档
- [ ] 召开复盘会议
- [ ] 制定改进措施
- [ ] 跟踪行动项

### 沟通模板

#### 事故通知模板

```markdown
【事故通知】[服务名] - [级别]

时间: [YYYY-MM-DD HH:MM UTC]
服务: [受影响服务]
级别: [P0/P1/P2/P3]
状态: [正在调查/已缓解/已解决]

影响:
- [描述影响范围]
- [受影响用户数]

当前进展:
- [正在采取的措施]

下一步:
- [计划采取的行动]

更新频率: [每 X 分钟]
联系人: [事故指挥官]
```

#### 状态更新模板

```markdown
【事故更新】[服务名] - [时间戳]

状态: [当前状态]
持续时间: [X 分钟]

进展:
1. [已完成的事项]
2. [当前正在进行]

下一步:
1. [计划行动 1]
2. [计划行动 2]

ETA: [预计恢复时间]
```

#### 事故解决通知

```markdown
【事故解决】[服务名]

解决时间: [YYYY-MM-DD HH:MM UTC]
持续时间: [X 分钟]

总结:
[简要描述事故和解决方案]

根因:
[初步根因分析]

后续:
- 事后复盘会议: [时间]
- 复盘文档: [链接]

感谢大家的耐心配合。
```

## 事后复盘 (Postmortem)

### 无责文化 (Blameless Culture)

#### 核心原则

1. **假设善意**：相关人员都是聪明、有良好意图的，基于当时信息做出了最佳决策
2. **关注系统**：修复环境、流程和工具，而非指责个人
3. **心理安全**：工程师可以安全地分享经验，不必担心惩罚
4. **学习导向**：每次事故都是学习机会

#### 无责复盘检查清单

- [ ] 是否使用了"应该"来批评决策？
- [ ] 是否聚焦于"谁"而非"为什么"？
- [ ] 是否考虑了当时的信息和约束？
- [ ] 是否识别了系统性问题？
- [ ] 改进措施是否针对系统而非个人？

### 事后复盘文档模板

```markdown
# 事后复盘: [事故标题]

## 元数据
- 事故编号: INC-YYYY-NNNN
- 日期: YYYY-MM-DD
- 持续时间: X 小时 Y 分钟
- 严重程度: P0/P1/P2/P3
- 影响服务: [服务列表]
- 事故指挥官: [姓名]
- 复盘作者: [姓名]
- 状态: [草稿/审查中/已批准]

## 执行摘要
[1-2 段非技术概述，描述发生了什么、影响和解决方案]

## 影响评估
### 用户影响
- 受影响用户数: [数量/百分比]
- 受影响功能: [功能列表]
- 用户可见症状: [描述]

### 业务影响
- 收入影响: [金额]
- SLA 影响: [违反的 SLO]
- 品牌影响: [描述]

## 时间线 (精确到分钟)

| 时间 (UTC) | 事件 | 备注 |
|------------|------|------|
| HH:MM | [事件描述] | [相关链接/日志] |
| HH:MM | [事件描述] | [相关链接/日志] |
| ... | ... | ... |

## 根因分析

### 直接原因
[导致事故的直接技术原因]

### 根本原因
[使用 5 Whys 方法深入分析]

1. 为什么发生？
   - [原因 1]
2. 为什么 [原因 1] 发生？
   - [原因 2]
3. 为什么 [原因 2] 发生？
   - [原因 3]
4. 为什么 [原因 3] 发生？
   - [原因 4]
5. 为什么 [原因 4] 发生？
   - [根本原因]

### 促成因素
- [列出所有促成事故的因素]

## 经验教训

### 做得好的地方
1. [正面经验 1]
2. [正面经验 2]

### 做得不好的地方
1. [改进点 1]
2. [改进点 2]

## 行动项

| ID | 行动项 | 负责人 | 优先级 | 截止日期 | 状态 |
|----|--------|--------|--------|----------|------|
| 1 | [具体行动] | [姓名] | P0/P1/P2 | YYYY-MM-DD | 待办/进行中/完成 |
| 2 | [具体行动] | [姓名] | P0/P1/P2 | YYYY-MM-DD | 待办/进行中/完成 |

## 附录

### 相关链接
- 事故频道: [Slack 链接]
- 监控仪表板: [链接]
- 相关代码变更: [PR 链接]
- 日志查询: [链接]

### 术语表
- [术语]: [定义]
```

### 根因分析方法

#### 5 Whys 方法

```
问题: 服务中断 2 小时

1. 为什么服务中断？
   → 数据库连接池耗尽

2. 为什么连接池耗尽？
   → 连接未正确释放

3. 为什么连接未释放？
   → 异常处理代码未关闭连接

4. 为什么异常处理有问题？
   → 代码审查未覆盖该场景

5. 为什么审查未覆盖？
   → 缺乏异常场景的测试用例

根本原因: 测试覆盖不足，缺乏异常场景测试
```

#### 鱼骨图 (Ishikawa Diagram)

```
                    人员
                     │
                     │ 技能不足
                     │ 培训缺失
                     │
    流程 ◄───────────┼───────────► 技术
    │                │                │
    │ 审批流程冗长   │                │ 架构缺陷
    │ 变更管理不严   │    事故        │ 技术债务
    │                │                │ 依赖故障
    │                │                │
                     │
                     │ 环境
                     │ 配置错误
                     │ 资源不足
                     │
```

## 事故模板

### 事故响应运行手册模板

```markdown
# [服务名] 事故响应运行手册

## 服务概述
- 服务名称: [名称]
- 服务类型: [API/数据库/消息队列/...]
- 关键依赖: [依赖列表]
- 用户影响: [描述]

## 关键指标
| 指标 | 正常范围 | 告警阈值 | 查询链接 |
|------|----------|----------|----------|
| 可用性 | > 99.9% | < 99% | [链接] |
| P99 延迟 | < 100ms | > 200ms | [链接] |
| 错误率 | < 0.1% | > 1% | [链接] |

## 常见故障场景

### 场景 1: [症状描述]
**诊断步骤:**
1. [检查指标 A]
2. [检查日志 B]
3. [验证依赖 C]

**缓解措施:**
1. [立即执行的操作]
2. [临时解决方案]

**永久修复:**
[长期解决方案]

### 场景 2: [症状描述]
...

## 升级路径
1. 值班工程师 (5 分钟)
2. 团队负责人 (15 分钟)
3. 部门总监 (30 分钟)
4. VP 工程 (1 小时)

## 联系人
- 技术负责人: [姓名] [电话]
- 产品经理: [姓名] [电话]
- 客户支持: [姓名] [电话]
```

### 事故复盘会议议程

```markdown
# 事故复盘会议议程

## 会议信息
- 日期: [日期]
- 时间: [时间]
- 地点: [会议室/视频会议]
- 参会者: [名单]

## 议程

### 1. 开场 (5 分钟)
- 重申无责文化原则
- 确认会议目标

### 2. 事故概述 (10 分钟)
- 事故指挥官简述事件
- 影响评估确认

### 3. 时间线回顾 (15 分钟)
- 逐分钟回顾关键事件
- 澄清疑问

### 4. 根因分析 (20 分钟)
- 直接原因
- 根本原因 (5 Whys)
- 促成因素

### 5. 经验教训 (10 分钟)
- 做得好的地方
- 需要改进的地方

### 6. 行动项讨论 (15 分钟)
- 提出改进措施
- 分配负责人和截止日期
- 确定优先级

### 7. 总结 (5 分钟)
- 确认行动项
- 下次审查时间

## 会议规则
1. 对事不对人
2. 聚焦系统性问题
3. 每个观点都值得倾听
4. 目标是预防而非指责
```

## 实践检查清单

### 事故响应准备

- [ ] 定义清晰的事故分级标准
- [ ] 建立事故指挥角色和职责
- [ ] 创建沟通渠道和模板
- [ ] 准备事故响应运行手册
- [ ] 建立升级路径
- [ ] 配置监控和告警
- [ ] 定期演练事故响应

### 事故响应中

- [ ] 快速确认事故级别
- [ ] 立即组建响应团队
- [ ] 建立清晰的指挥链
- [ ] 保持持续沟通
- [ ] 优先缓解而非修复
- [ ] 详细记录时间线
- [ ] 控制影响范围

### 事故响应后

- [ ] 编写事后复盘文档
- [ ] 召开无责复盘会议
- [ ] 识别系统性改进点
- [ ] 制定具体行动项
- [ ] 跟踪行动项完成
- [ ] 分享学习成果
- [ ] 更新运行手册

## 真实案例参考

### 案例：Google DiRT 演练

**背景**：Google SRE 的灾难恢复测试 (Disaster Recovery Testing)

**做法**：
- 定期故意制造生产环境故障
- 测试系统和人员的响应能力
- 验证恢复流程的有效性

**收益**：
- 提前发现潜在问题
- 团队熟悉响应流程
- 验证备份和恢复机制
- 提高整体恢复能力

### 案例：无责复盘文化建设

**背景**：某公司从事故中学习

**转变**：
- 从事故前：寻找"责任人"
- 到事故后：寻找"系统改进点"

**结果**：
- 事故报告数量增加 (更多人愿意报告)
- 重复事故减少
- 团队心理安全感提升
- 系统可靠性持续改进

### 案例：事故响应时间改进

**改进前**：
- 平均发现时间 (MTTD)：15 分钟
- 平均响应时间 (MTTR)：2 小时

**改进措施**：
1. 优化告警阈值
2. 建立运行手册
3. 定期演练
4. 自动化响应

**改进后**：
- MTTD：3 分钟
- MTTR：30 分钟

## 常用命令

### 日志分析（事故调查）

```bash
# Linux - 查找错误日志
grep -i "error\|exception\|fatal" app.log | tail -50
grep -E "(ERROR|CRITICAL)" app.log | wc -l

# PowerShell - 查找错误日志
Get-Content app.log | Select-String -Pattern "ERROR|CRITICAL" -CaseSensitive:$false | Select-Object -Last 50
(Select-String -Path app.log -Pattern "ERROR|CRITICAL").Count

# PowerShell - 按时间窗口过滤日志
$startTime = (Get-Date).AddHours(-1)
Get-Content app.log | ForEach-Object {
    if ($_ -match "^(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:") {
        $logTime = [DateTime]::Parse($matches[1])
        if ($logTime -ge $startTime) { $_ }
    }
}

# PowerShell - 生成错误摘要报告
$errors = Get-Content app.log | Select-String "ERROR"
$errors | ForEach-Object {
    ($_.Line -split "ERROR")[1].Trim().Split(" ")[0]
} | Group-Object | Sort-Object Count -Descending | Select-Object -First 10
```

### JSON数据处理（事故元数据）

```bash
# Linux - 使用jq处理事故数据
cat incident-metadata.json | jq '.timeline[] | {time: .timestamp, event: .description}'

# PowerShell - 处理事故元数据
$incident = Get-Content incident-metadata.json | ConvertFrom-Json
$incident.timeline | ForEach-Object {
    [PSCustomObject]@{
        Time = $_.timestamp
        Event = $_.description
        Duration = $_.duration_minutes
    }
} | Format-Table -AutoSize

# PowerShell - 计算事故MTTR
$incidentStart = [DateTime]$incident.start_time
$incidentEnd = [DateTime]$incident.end_time
$mttr = $incidentEnd - $incidentStart
Write-Output "Incident Duration: $($mttr.TotalMinutes) minutes"
```

### 性能监控（事故期间系统状态）

```bash
# Linux - 系统资源监控
top -bn1 | head -20
vmstat 1 5

# PowerShell - 系统资源监控
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 10 Name, WorkingSet, CPU
Get-Counter '\Processor(_Total)\% Processor Time' -SampleInterval 1 -MaxSamples 5
Get-Counter '\Memory\Available MBytes', '\Paging File(_Total)\% Usage'

# PowerShell - 网络连接检查
Test-NetConnection -ComputerName critical-service.internal -Port 443
Get-NetTCPConnection | Where-Object { $_.State -eq "Established" } | Group-Object RemoteAddress | Sort-Object Count -Descending | Select-Object -First 10
```

### 文件操作（事故文档管理）

```bash
# Linux - 创建事故目录结构
mkdir -p incidents/$(date +%Y-%m-%d)/{logs,screenshots,analysis}

# PowerShell - 创建事故目录结构
$incidentDate = Get-Date -Format "yyyy-MM-dd"
$incidentId = "INC-$(Get-Date -Format "yyyyMMdd-HHmmss")"
$paths = @(
    "incidents/$incidentDate/$incidentId/logs"
    "incidents/$incidentDate/$incidentId/screenshots"
    "incidents/$incidentDate/$incidentId/analysis"
)
$paths | ForEach-Object { New-Item -ItemType Directory -Path $_ -Force }

# PowerShell - 收集系统信息到事故目录
Get-Process | Export-Csv "incidents/$incidentDate/$incidentId/processes.csv" -NoTypeInformation
Get-EventLog -LogName System -Newest 100 | Export-Csv "incidents/$incidentDate/$incidentId/system-events.csv" -NoTypeInformation
```

### 日期时间处理（事故时间线）

```bash
# Linux - ISO格式时间
date -u +%Y-%m-%dT%H:%M:%SZ

# PowerShell - ISO格式时间
Get-Date -Format "yyyy-MM-ddTHH:mm:ssZ"
(Get-Date).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ssZ")

# PowerShell - 事故时间线生成
$timeline = @()
$timeline += [PSCustomObject]@{ Time = (Get-Date).ToUniversalTime(); Event = "Incident Detected"; Type = "Detection" }
Start-Sleep -Seconds 30
$timeline += [PSCustomObject]@{ Time = (Get-Date).ToUniversalTime(); Event = "Response Started"; Type = "Response" }
$timeline | ConvertTo-Json | Out-File incident-timeline.json
```

## 输出规范

```
🚨 事故响应报告

📋 事故基本信息
- 事故编号：[INCIDENT-YYYY-MM-DD-001]
- 开始时间：[start_time]
- 持续时间：[duration]
- 严重程度：[P0/P1/P2/P3]
- 状态：[resolved/ongoing]

📊 事故摘要
- 现象：[symptom]
- 影响范围：[scope]
- 受影响用户：[affected_users]
- 业务影响：[business_impact]

⏱️ 时间线
| 时间 | 事件 | 负责人 |
|------|------|--------|
| T+0 | [检测时间] | [detector] |
| T+5m | [响应开始] | [responder] |
| T+30m | [缓解措施] | [responder] |
| T+2h | [完全恢复] | [responder] |

🔍 根本原因
[root_cause_analysis]

💡 解决措施
- 即时措施：[immediate_action]
- 修复方案：[fix_description]

🛡️ 预防措施
| 优先级 | 任务 | 负责人 | 截止日期 |
|--------|------|--------|----------|
| P0 | [action] | [owner] | [date] |
| P1 | [action] | [owner] | [date] |

📈 经验教训
- 做得好的：[what_went_well]
- 需改进的：[what_could_be_better]
- 幸运因素：[what_we_got_lucky]

🔖 事后复盘
- 复盘日期：[date]
- 参与人员：[attendees]
- 会议记录：[link]
```

## 参考资源

- [Google SRE Book - 值班](https://sre.google/sre-book/being-on-call/)
- [Google SRE Book - 有效排障](https://sre.google/sre-book/effective-troubleshooting/)
- [Google SRE Book - 应急响应](https://sre.google/sre-book/emergency-response/)
- [Google SRE Book - 事故管理](https://sre.google/sre-book/managing-incidents/)
- [Google SRE Book - 事后复盘文化](https://sre.google/sre-book/postmortem-culture/)
- [Google SRE Workbook - 事后复盘](https://sre.google/workbook/postmortem/)
