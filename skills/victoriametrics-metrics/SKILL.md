---
name: victoriametrics-metrics
description: VictoriaMetrics metrics analysis using MetricsQL. Use when querying time-series metrics stored in VictoriaMetrics. Supports PromQL and MetricsQL extensions.
allowed-tools: Bash(python *)
---

# VictoriaMetrics Metrics Analysis

Query and 分析 time-series metrics from VictoriaMetrics using MetricsQL (PromQL-compatible with extensions).

## Authentication

**重要**: Credentials are injected automatically by a proxy layer. Do NOT 检查 for `VICTORIAMETRICS_TOKEN` in environment variables - they won't be visible 迁移到 you. Just 运行 the scripts directly; authentication is handled transparently.

---

## MANDATORY: Context-Efficient Investigation

**NEVER dump all series or 运行 unbounded range queries.** Always follow this pattern:

```
GET STATISTICS → INSTANT QUERY → RANGE QUERY (only if needed)
```

1. **Statistics First** — Know how many series exist, what metrics/jobs are active
2. **Instant Query** — 获取 current values (single point, compact 输出)
3. **Range Query** — Only when you 需要 trend over time, and ALWAYS with `topk()` 迁移到 cap 输出

## Available Scripts

All scripts are in `.claude/skills/metrics-victoriametrics/scripts/`

### get_statistics.py — ALWAYS 启动 HERE
Discover what metrics exist and their cardinality.
```bash
python .claude/skills/metrics-victoriametrics/scripts/get_statistics.py --query '{job="api"}'
python .claude/skills/metrics-victoriametrics/scripts/get_statistics.py --query '{namespace="production"}' --time-range 120
python .claude/skills/metrics-victoriametrics/scripts/get_statistics.py --query '{}' --json
```

输出 includes:
- Active series count
- Top 10 metric names by series count
- Top 5 jobs
- Compact 摘要 (~20 lines)

### query_metrics.py — Targeted Queries
执行 MetricsQL queries with 输出 limits.
```bash
# Instant query (default - single value per series, compact)
python .claude/skills/metrics-victoriametrics/scripts/query_metrics.py --query 'up{job="api"}'
python .claude/skills/metrics-victoriametrics/scripts/query_metrics.py --query 'rate(http_requests_total{service="payment"}[5m])'

# Range query (use sparingly - shows latest value per series, not all datapoints)
python .claude/skills/metrics-victoriametrics/scripts/query_metrics.py --query 'rate(http_requests_total[5m])' --type range --time-range 60

# Limit output
python .claude/skills/metrics-victoriametrics/scripts/query_metrics.py --query 'topk(5, rate(http_requests_total[5m]))' --limit 10 --json
```

### list_labels.py — Metadata Discovery
Discover available labels and values.
```bash
python .claude/skills/metrics-victoriametrics/scripts/list_labels.py
python .claude/skills/metrics-victoriametrics/scripts/list_labels.py --label job
python .claude/skills/metrics-victoriametrics/scripts/list_labels.py --label namespace --match '{job="api"}' --json
```

---

## MetricsQL Quick 参考

MetricsQL is fully PromQL-compatible with additional extensions.

### Basic Queries
```metricsql
# Instant vector
http_requests_total{service="api"}

# Rate of increase per second
rate(http_requests_total{service="api"}[5m])

# Histogram quantile
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

### Aggregations
```metricsql
sum by (service) (rate(http_requests_total[5m]))
avg by (instance) (cpu_usage)
topk(5, sum by (service) (rate(http_requests_total[5m])))
bottomk(3, avg_over_time(cpu_usage[1h]))
```

### MetricsQL Extensions (beyond PromQL)

#### WITH Templates — Reusable Filters
```metricsql
WITH (
  commonFilters = {job="api", env="prod"},
  errorRate(m) = rate(m{status=~"5.."}[5m]) / rate(m[5m])
)
errorRate(http_requests_total{commonFilters})
```

#### Rollup Functions
```metricsql
rollup(metric[5m])              # Returns min, max, avg in one query
rollup_rate(counter[5m])        # Rate with proper counter reset handling
rollup_increase(counter[5m])    # Increase with counter reset handling
```

#### Label Manipulation
```metricsql
label_set(metric, "env", "prod")       # Set label value
label_del(metric, "instance")          # Remove label
label_copy(metric, "pod", "instance")  # Copy label
label_move(metric, "old", "new")       # Rename label
label_join(metric, "dst", ",", "a", "b")  # Join labels
```

#### Range & Time Functions
```metricsql
range_median(metric[1h])         # Median over range
range_first(metric[1h])         # First value in range
range_last(metric[1h])          # Last value in range
running_avg(metric[1h])         # Running average
```

### Label Matching
```metricsql
{status="500"}           # Exact match
{status=~"5.."}          # Regex match
{status!="200"}          # Not equal
{service="api", status=~"5.."}   # Multiple labels
```

---

## Investigation Workflows

### 错误 Rate Investigation
```bash
# Step 1: Get statistics
python get_statistics.py --query '{job="api"}'

# Step 2: Current error rate
python query_metrics.py --query 'sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))'

# Step 3: Error rate by service (top 5 only)
python query_metrics.py --query 'topk(5, sum by (service) (rate(http_requests_total{status=~"5.."}[5m])))'
```

### Latency Investigation
```bash
python query_metrics.py --query 'histogram_quantile(0.95, sum by (le) (rate(http_request_duration_seconds_bucket{service="api"}[5m])))'
```

### 资源 Investigation
```bash
python query_metrics.py --query 'topk(5, rate(container_cpu_usage_seconds_total{namespace="prod"}[5m]))'
python query_metrics.py --query 'topk(5, container_memory_usage_bytes{namespace="prod"} / container_spec_memory_limit_bytes{namespace="prod"})'
```

---

## Anti-Patterns 迁移到 Avoid

1. **NEVER 运行 unbounded `query_range`** — Always use `to`to`k()` 筛选 by specific labels
2. **NEVER skip `get_statistics.py`** — Know your cardinality before querying
3. **NEVER use `rate()` without range vector** — Always incl`[5m]`]`]` or similar
4. **NEVER compare counters directly** — Use `rate()``increase()`)`)` first
5. **Avoid short steps in range queries** — Use `step` >= 2x scrape interval