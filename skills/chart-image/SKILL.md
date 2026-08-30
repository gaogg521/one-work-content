---
name: chart-image
version: 2.2.0
description: 从数据生成出版物质量的图表图像。支持折线图、柱状图、面积图、点图、蜡烛图、饼图/环形图、热力图、多系列图和堆叠图。在可视化数据、创建图形、绘制时间序列或生成用于报告/警报的图表图像时使用。专为 Fly.io/VPS 部署设计 —— 无需原生编译、无需 Puppeteer、无需浏览器。纯 Node.js 配合预构建二进制文件。
provides:
- capability: chart-generation
  methods:
  - lineChart
  - barChart
  - areaChart
  - pieChart
  - candlestickChart
  - heatmap
tags:
- Helm
---

# Chart Image Generator

使用 Vega-Lite 从数据生成 PNG 图表图像。非常适合无头服务器环境。

## 为什么选择此技能？

**为 Fly.io / VPS / Docker 部署而构建：**
- ✅ **无需原生编译** - 使用 Sharp 配合预构建二进制文件（不像 `canvas` 需要构建工具）
- ✅ **无需 Puppeteer/浏览器** - 纯 Node.js，无需下载 Chrome，无无头浏览器开销
- ✅ **轻量** - 总依赖约 15MB，而基于 Puppeteer 的解决方案需要 400MB+
- ✅ **快速冷启动** - 无需浏览器启动延迟，生成图表 <500ms
- ✅ **离线工作** - 无外部 API 调用（不像 QuickChart.io）

## 设置（一次性）

```bash
cd /data/clawd/skills/chart-image/scripts && npm install
```

## 快速使用

```bash
node /data/clawd/skills/chart-image/scripts/chart.mjs \
  --type line \
  --data '[{"x":"10:00","y":25},{"x":"10:30","y":27},{"x":"11:00","y":31}]' \
  --title "Price Over Time" \
  --output chart.png
```

## 图表类型

### Line Chart（默认）
```bash
node chart.mjs --type line --data '[{"x":"A","y":10},{"x":"B","y":15}]' --output line.png
```

### Bar Chart
```bash
node chart.mjs --type bar --data '[{"x":"A","y":10},{"x":"B","y":15}]' --output bar.png
```

### Area Chart
```bash
node chart.mjs --type area --data '[{"x":"A","y":10},{"x":"B","y":15}]' --output area.png
```

### Pie / Donut Chart
```bash
# Pie
node chart.mjs --type pie --data '[{"category":"A","value":30},{"category":"B","value":70}]' \
  --category-field category --y-field value --output pie.png

# Donut（带孔）
node chart.mjs --type donut --data '[{"category":"A","value":30},{"category":"B","value":70}]' \
  --category-field category --y-field value --output donut.png
```

### Candlestick Chart（OHLC）
```bash
node chart.mjs --type candlestick \
  --data '[{"x":"Mon","open":100,"high":110,"low":95,"close":105}]' \
  --open-field open --high-field high --low-field low --close-field close \
  --title "Stock Price" --output candle.png
```

### Heatmap
```bash
node chart.mjs --type heatmap \
  --data '[{"x":"Mon","y":"Week1","value":5},{"x":"Tue","y":"Week1","value":8}]' \
  --color-value-field value --color-scheme viridis \
  --title "Activity Heatmap" --output heatmap.png
```

### Multi-Series Line Chart
在同一图表上比较多个趋势：
```bash
node chart.mjs --type line --series-field "market" \
  --data '[{"x":"Jan","y":10,"market":"A"},{"x":"Jan","y":15,"market":"B"}]' \
  --title "Comparison" --output multi.png
```

### Stacked Bar Chart
```bash
node chart.mjs --type bar --stacked --color-field "category" \
  --data '[{"x":"Mon","y":10,"category":"Work"},{"x":"Mon","y":5,"category":"Personal"}]' \
  --title "Hours by Category" --output stacked.png
```

### Volume Overlay（双 Y 轴）
价格折线配合成交量柱状图：
```bash
node chart.mjs --type line --volume-field volume \
  --data '[{"x":"10:00","y":100,"volume":5000},{"x":"11:00","y":105,"volume":3000}]' \
  --title "Price + Volume" --output volume.png
```

### Sparkline（迷你内联图表）
```bash
node chart.mjs --sparkline --data '[{"x":"1","y":10},{"x":"2","y":15}]' --output spark.png
```
Sparklines 默认 80x20，透明，无坐标轴。

## 选项参考

### 基础选项
| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `--type` | 图表类型：line、bar、area、point、pie、donut、candlestick、heatmap | line |
| `--data` | 数据点的 JSON 数组 | - |
| `--output` | 输出文件路径 | chart.png |
| `--title` | 图表标题 | - |
| `--width` | 宽度（像素） | 600 |
| `--height` | 高度（像素） | 300 |

### 坐标轴选项
| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `--x-field` | X 轴字段名 | x |
| `--y-field` | Y 轴字段名 | y |
| `--x-title` | X 轴标签 | 字段名 |
| `--y-title` | Y 轴标签 | 字段名 |
| `--x-type` | X 轴类型：ordinal、temporal、quantitative | ordinal |
| `--y-domain` | Y 比例尺，格式为 "min,max" | auto |

### 视觉选项
| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `--color` | 折线/柱状图颜色 | #e63946 |
| `--dark` | 深色模式主题 | false |
| `--svg` | 输出 SVG 而不是 PNG | false |
| `--color-scheme` | Vega 颜色方案（category10、viridis 等） | - |

### 警报/监控选项
| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `--show-change` | 在最后一点显示 +/-% 变化注释 | false |
| `--focus-change` | 将 Y 轴缩放到 2 倍数据范围 | false |
| `--focus-recent N` | 仅显示最后 N 个数据点 | all |
| `--show-values` | 标注最小/最大峰值点 | false |

### 多系列/堆叠选项
| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `--series-field` | 多系列折线图的字段 | - |
| `--stacked` | 启用堆叠柱状图模式 | false |
| `--color-field` | 堆叠/颜色分类的字段 | - |

### Candlestick 选项
| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `--open-field` | OHLC open 字段 | open |
| `--high-field` | OHLC high 字段 | high |
| `--low-field` | OHLC low 字段 | low |
| `--close-field` | OHLC close 字段 | close |

### Pie/Donut 选项
| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `--category-field` | 饼图切片分类的字段 | x |
| `--donut` | 渲染为 donut（带中心孔） | false |

### Heatmap 选项
| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `--color-value-field` | 热力图强度字段 | value |
| `--y-category-field` | Y 轴分类字段 | y |

### Volume Overlay 选项
| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `--volume-field` | 成交量柱状图字段（启用双轴） | - |
| `--volume-color` | 成交量柱状图颜色 | #4a5568 |

### 格式化选项
| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `--y-format` | Y 轴格式：percent、dollar、compact、decimal4、integer、scientific 或 d3-format 字符串 | auto |
| `--subtitle` | 图表标题下方的副标题文本 | - |
| `--hline` | 水平参考线："value" 或 "value,color" 或 "value,color,label"（可重复） | - |

### 注释选项
| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `--annotation` | 静态文本注释 | - |
| `--annotations` | 事件标记的 JSON 数组 | - |

## Alert-Style Chart（监控推荐）

```bash
node chart.mjs --type line --data '[...]' \
  --title "Iran Strike Odds (48h)" \
  --show-change --focus-change --show-values --dark \
  --output alert.png
```

仅显示近期动作：
```bash
node chart.mjs --type line --data '[hourly data...]' \
  --focus-recent 4 --show-change --focus-change --dark \
  --output recent.png
```

## Timeline Annotations

在图表上标记事件：
```bash
node chart.mjs --type line --data '[...]' \
  --annotations '[{"x":"14:00","label":"News broke"},{"x":"16:30","label":"Press conf"}]' \
  --output annotated.png
```

## Temporal X-Axis

对于带有日期间隔的适当时间序列：
```bash
node chart.mjs --type line --x-type temporal \
  --data '[{"x":"2026-01-01","y":10},{"x":"2026-01-15","y":20}]' \
  --output temporal.png
```

当 X 值是 ISO 日期且你希望间距反映实际时间间隔（而非均匀分布）时，使用 `--x-type temporal`。

## Y-Axis Formatting

格式化坐标轴值以提高可读性：
```bash
# 美元金额
node chart.mjs --data '[...]' --y-format dollar --output revenue.png
# → $1,234.56

# 百分比（值为小数 0-1）
node chart.mjs --data '[...]' --y-format percent --output rates.png
# → 45.2%

# 紧凑大数字
node chart.mjs --data '[...]' --y-format compact --output users.png
# → 1.2K, 3.4M

# 加密货币价格（4 位小数）
node chart.mjs --data '[...]' --y-format decimal4 --output molt.png
# → 0.0004

# 自定义 d3-format 字符串
node chart.mjs --data '[...]' --y-format ',.3f' --output custom.png
```

可用快捷方式：`percent`、`dollar`/`usd`、`compact`、`integer`、`decimal2`、`decimal4`、`scientific`

## Chart Subtitle

在标题下方添加上下文：
```bash
node chart.mjs --title "MOLT Price" --subtitle "20,668 MOLT held" --data '[...]' --output molt.png
```

## Theme Selection

使用 `--dark` 开启深色模式。基于时间自动选择：
- **夜间（20:00-07:00 本地时间）**：`--dark`
- **白天（07:00-20:00 本地时间）**：浅色模式（默认）

## Piping Data

```bash
echo '[{"x":"A","y":1},{"x":"B","y":2}]' | node chart.mjs --output out.png
```

## Custom Vega-Lite Spec

对于高级图表：
```bash
node chart.mjs --spec my-spec.json --output custom.png
```

## ⚠️ 重要：始终发送图像！

生成图表后，**始终将其发送回用户的频道**。
不要仅仅保存到文件并描述它 —— 重点在于视觉。

```bash
# 1. 生成图表
node chart.mjs --type line --data '...' --output /data/clawd/tmp/my-chart.png

# 2. 发送！使用 message 工具并带 filePath：
#    action=send, target=<channel_id>, filePath=/data/clawd/tmp/my-chart.png
```

**提示：**
- 保存到 `/data/clawd/tmp/`（持久化）而不是 `/tmp/`（可能被清理）
- 使用 `action=send` 并带 `filePath` —— `thread-reply` 不支持文件附件
- 在消息文本中包含简短的标题
- 在 20:00-07:00 以色列时间自动使用 `--dark`

---
*更新于：2026-02-04 —— 添加了 --y-format（percent/dollar/compact/decimal4）和 --subtitle*
