---
name: google-maps
description: 用于 OpenClaw 的 Google Maps 集成，带有 Routes API。用于：(1) 带交通预测的距离/行程时间计算，(2) 逐向导航，(3) 多点之间的距离矩阵，(4) 地址到坐标和反向的地理编码，(5) 地点搜索和详情，(6) 带到达时间的公交规划。支持未来出发时间、交通模型（pessimistic/optimistic）、避开选项（tolls/highways）以及多种出行模式（driving/walking/bicycling/transit）。
version: 3.0.0
author: Leo 🦁
tags:
- maps
- places
- location
- navigation
- google
metadata: None
clawdbot: None
emoji: 🗺️
install:
- id: pip
  kind: pip
  label: Install dependencies (pip)
  package: requests
primaryEnv: GOOGLE_API_KEY
requires: None
env:
- GOOGLE_API_KEY
allowed-tools:
- exec
---

# Google Maps 🗺️

由 Routes API 驱动的 Google Maps 集成。

## 要求

- `GOOGLE_API_KEY` 环境变量
- 在 Google Cloud Console 中启用：Routes API、Places API、Geocoding API

## 配置

| 环境变量 | 默认值 | 说明 |
|--------------|---------|-------------|
| `GOOGLE_API_KEY` | - | 必需。你的 Google Maps API key |
| `GOOGLE_MAPS_LANG` | `en` | 响应语言（en、he、ja 等） |

在 OpenClaw config 中设置：
```json
{
  "env": {
    "GOOGLE_API_KEY": "AIza...",
    "GOOGLE_MAPS_LANG": "en"
  }
}
```

## Script 位置

```bash
python3 skills/google-maps/lib/map_helper.py <action> [options]
```

---

## 操作

### distance - 计算行程时间

```bash
python3 lib/map_helper.py distance "origin" "destination" [options]
```

**选项：**
| 选项 | 可选值 | 说明 |
|--------|--------|-------------|
| `--mode` | driving、walking、bicycling、transit | 出行模式（默认：driving） |
| `--depart` | now、+30m、+1h、14:00、2026-02-07 08:00 | 出发时间 |
| `--arrive` | 14:00 | 到达时间（仅 transit） |
| `--traffic` | best_guess、pessimistic、optimistic | 交通模型 |
| `--avoid` | tolls、highways、ferries | 逗号分隔 |

**示例：**
```bash
python3 lib/map_helper.py distance "New York" "Boston"
python3 lib/map_helper.py distance "Los Angeles" "San Francisco" --depart="+1h"
python3 lib/map_helper.py distance "Chicago" "Detroit" --depart="08:00" --traffic=pessimistic
python3 lib/map_helper.py distance "London" "Manchester" --mode=transit --arrive="09:00"
python3 lib/map_helper.py distance "Paris" "Lyon" --avoid=tolls,highways
```

**响应：**
```json
{
  "distance": "215.2 mi",
  "distance_meters": 346300,
  "duration": "3 hrs 45 mins",
  "duration_seconds": 13500,
  "static_duration": "3 hrs 30 mins",
  "duration_in_traffic": "3 hrs 45 mins"
}
```

---

### directions - 逐向路线

```bash
python3 lib/map_helper.py directions "origin" "destination" [options]
```

**额外选项（超出 distance）：**
| 选项 | 说明 |
|--------|-------------|
| `--alternatives` | 返回多条路线 |
| `--waypoints` | 中途停靠点（pipe-separated） |
| `--optimize` | 优化 waypoint 顺序（TSP） |

**示例：**
```bash
python3 lib/map_helper.py directions "New York" "Washington DC"
python3 lib/map_helper.py directions "San Francisco" "Los Angeles" --alternatives
python3 lib/map_helper.py directions "Miami" "Orlando" --waypoints="Fort Lauderdale|West Palm Beach" --optimize
```

**响应包含：** summary、labels、duration、static_duration、warnings、steps[]、optimized_waypoint_order

---

### matrix - 距离矩阵

计算多个 origins 和 destinations 之间的距离：

```bash
python3 lib/map_helper.py matrix "orig1|orig2" "dest1|dest2"
```

**示例：**
```bash
python3 lib/map_helper.py matrix "New York|Boston" "Philadelphia|Washington DC"
```

**响应：**
```json
{
  "origins": ["New York", "Boston"],
  "destinations": ["Philadelphia", "Washington DC"],
  "results": [
    {"origin_index": 0, "destination_index": 0, "distance": "97 mi", "duration": "1 hr 45 mins"},
    {"origin_index": 0, "destination_index": 1, "distance": "225 mi", "duration": "4 hrs 10 mins"}
  ]
}
```

---

### geocode - 地址到坐标

```bash
python3 lib/map_helper.py geocode "1600 Amphitheatre Parkway, Mountain View, CA"
python3 lib/map_helper.py geocode "10 Downing Street, London"
```

### reverse - 坐标到地址

```bash
python3 lib/map_helper.py reverse 40.7128 -74.0060  # New York City
python3 lib/map_helper.py reverse 51.5074 -0.1278  # London
```

---

### search - 查找地点

```bash
python3 lib/map_helper.py search "coffee near Times Square"
python3 lib/map_helper.py search "pharmacy in San Francisco" --open
```

### details - 地点信息

```bash
python3 lib/map_helper.py details "<place_id>"
```

---

## 交通模型

| 模型 | 使用场景 |
|-------|----------|
| `best_guess` | 默认平衡估计 |
| `pessimistic` | 重要会议（最坏情况） |
| `optimistic` | 最佳情况 |

---

## 区域说明

某些功能可能在所有国家/地区不可用：

| 功能 | 可用性 |
|---------|--------------|
| `--fuel-efficient` | US、EU、select countries |
| `--shorter` | Limited availability |
| `--mode=two_wheeler` | Asia、select countries |

查看 [Google Maps coverage](https://developers.google.com/maps/coverage) 了解详情。

---

## 多语言支持

适用于任何语言的地址：

```bash
# Hebrew
python3 lib/map_helper.py distance "תל אביב" "ירושלים"
python3 lib/map_helper.py geocode "דיזנגוף 50, תל אביב"

# Japanese
python3 lib/map_helper.py distance "東京" "大阪"

# Arabic
python3 lib/map_helper.py distance "دبي" "أبو ظبي"
```

**语言配置：**

1. 通过 env 设置默认值：`GOOGLE_MAPS_LANG=he`（持久化）
2. 按请求覆盖：`--lang=ja`

```bash
# 在 OpenClaw config 中将 Hebrew 设为默认
GOOGLE_MAPS_LANG=he

# 为特定请求覆盖
python3 lib/map_helper.py distance "Tokyo" "Osaka" --lang=ja
```

---

## 帮助

```bash
python3 lib/map_helper.py help
```
