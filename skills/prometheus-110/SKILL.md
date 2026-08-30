---
name: prometheus
description: 查询 Prometheus 监控数据以检查服务器指标、资源使用情况和系统健康状态。当用户询问服务器状态、磁盘空间、CPU/内存使用情况、网络统计或 Prometheus 收集的任何指标时使用。支持通过配置文件或环境变量配置多个 Prometheus 实例以及 HTTP Basic Auth。
---

# Prometheus Skill

从一个或多个实例查询 Prometheus 监控数据。支持通过单个命令跨多个 Prometheus 服务器进行联邦查询。

## 快速开始

### 1. 初始设置

运行交互式配置向导：

```bash
cd ~/.openclaw/workspace/skills/prometheus
node scripts/cli.js init
```

这将在你的 OpenClaw 工作区（`~/.openclaw/workspace/prometheus.json`）中创建一个 `prometheus.json` 配置文件。

### 2. 开始查询

```bash
# 查询默认实例
node scripts/cli.js query 'up'

# 同时查询所有实例
node scripts/cli.js query 'up' --all

# 列出已配置的实例
node scripts/cli.js instances
```

## 配置

### 配置文件位置

默认情况下，skill 会在你的 OpenClaw 工作区中查找配置：

```
~/.openclaw/workspace/prometheus.json
```

**优先级顺序：**
1. 来自 `PROMETHEUS_CONFIG` 环境变量的路径
2. `~/.openclaw/workspace/prometheus.json`
3. `~/.openclaw/workspace/config/prometheus.json`
4. `./prometheus.json`（当前目录）
5. `~/.config/prometheus/config.json`

### 配置格式

在你的工作区中创建 `prometheus.json`（或使用 `node cli.js init`）：

```json
{
  "instances": [
    {
      "name": "production",
      "url": "https://prometheus.example.com",
      "user": "admin",
      "password": "secret"
    },
    {
      "name": "staging",
      "url": "http://prometheus-staging:9090"
    }
  ],
  "default": "production"
}
```

**字段：**
- `name` — 实例的唯一标识符
- `url` — Prometheus 服务器 URL
- `user` / `password` — 可选的 HTTP Basic Auth 凭据
- `default` — 未指定实例时使用的默认实例

### 环境变量（传统方式）

对于单实例设置，你可以使用环境变量：

```bash
export PROMETHEUS_URL=https://prometheus.example.com
export PROMETHEUS_USER=admin        # 可选
export PROMETHEUS_PASSWORD=secret   # 可选
```

## 用法

### 全局标志

| 标志 | 描述 |
|------|------|
| `-c, --config <path>` | 配置文件路径 |
| `-i, --instance <name>` | 指定目标实例 |
| `-a, --all` | 查询所有已配置的实例 |

### 命令

#### 设置

```bash
# 交互式配置向导
node scripts/cli.js init
```

#### 查询指标

```bash
cd ~/.openclaw/workspace/skills/prometheus

# 查询默认实例
node scripts/cli.js query 'up'

# 查询指定实例
node scripts/cli.js query 'up' -i staging

# 同时查询所有实例
node scripts/cli.js query 'up' --all

# 使用自定义配置文件
node scripts/cli.js query 'up' -c /path/to/config.json
```

#### 常用查询

**磁盘空间使用情况：**
```bash
node scripts/cli.js query '100 - (node_filesystem_avail_bytes / node_filesystem_size_bytes * 100)' --all
```

**CPU 使用情况：**
```bash
node scripts/cli.js query '100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)' --all
```

**内存使用情况：**
```bash
node scripts/cli.js query '(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100' --all
```

**平均负载：**
```bash
node scripts/cli.js query 'node_load1' --all
```

### 列出已配置的实例

```bash
node scripts/cli.js instances
```

输出：
```json
{
  "default": "production",
  "instances": [
    { "name": "production", "url": "https://prometheus.example.com", "hasAuth": true },
    { "name": "staging", "url": "http://prometheus-staging:9090", "hasAuth": false }
  ]
}
```

### 其他命令

```bash
# 列出匹配模式的所有指标
node scripts/cli.js metrics 'node_memory_*'

# 获取标签名称
node scripts/cli.js labels --all

# 获取标签的值
node scripts/cli.js label-values instance --all

# 查找时间序列
node scripts/cli.js series '{__name__=~"node_cpu_.*", instance=~".*:9100"}' --all

# 获取活动告警
node scripts/cli.js alerts --all

# 获取抓取目标
node scripts/cli.js targets --all
```

## 多实例输出格式

使用 `--all` 时，结果包含来自所有实例的数据：

```json
{
  "resultType": "vector",
  "results": [
    {
      "instance": "production",
      "status": "success",
      "resultType": "vector",
      "result": [...]
    },
    {
      "instance": "staging",
      "status": "success",
      "resultType": "vector",
      "result": [...]
    }
  ]
}
```

单个实例的错误不会导致整个查询失败 —— 它们会在结果数组中显示为 `"status": "error"`。

## 常用查询参考

| 指标 | PromQL 查询 |
|------|-------------|
| 磁盘可用百分比 | `node_filesystem_avail_bytes / node_filesystem_size_bytes * 100` |
| 磁盘已用百分比 | `100 - (node_filesystem_avail_bytes / node_filesystem_size_bytes * 100)` |
| CPU 空闲百分比 | `avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100` |
| 内存已用百分比 | `(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100` |
| 网络接收速率 | `rate(node_network_receive_bytes_total[5m])` |
| 网络发送速率 | `rate(node_network_transmit_bytes_total[5m])` |
| 运行时间 | `node_time_seconds - node_boot_time_seconds` |
| 服务运行状态 | `up` |

## 注意事项

- 即时查询的时间范围默认为最近 1 小时
- 使用范围查询 `[5m]` 进行速率计算
- 所有查询返回 JSON 格式，`data.result` 包含结果
- 实例标签通常显示 `host:port` 格式
- 使用 `--all` 时，查询并行运行以获得更快的结果
- 配置存储在 skill 目录之外，以便在 skill 更新时持久保留
