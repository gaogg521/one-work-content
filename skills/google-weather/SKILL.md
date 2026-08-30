---
name: google-weather
description: Google Weather API - 准确、实时的天气数据。获取当前状况、温度、湿度、风和预报。由 Google 的 Weather API 提供支持，每 15 分钟更新一次可靠的超本地数据。支持全球任何位置。
tags:
- weather
- google
- forecast
- temperature
- real-time
---

# Google Weather - 实时天气数据

使用 Google 的 Weather API 获取准确的天气状况。需要启用 Weather API 的 Google Cloud API key。

## 快速用法

```bash
# 当前天气（格式化输出）
python3 skills/google-weather/lib/weather_helper.py current "New York"
python3 skills/google-weather/lib/weather_helper.py current "London"
python3 skills/google-weather/lib/weather_helper.py current "Sydney"

# 24 小时预报
python3 skills/google-weather/lib/weather_helper.py forecast "Tel Aviv"

# 原始 JSON 数据
python3 skills/google-weather/lib/weather_helper.py json "Paris"
```

## 示例输出

```
*New York*
Partly Cloudy ⛅
🌡️ 12°C (feels like 10°C)
💨 Wind: 18 km/h NORTHWEST
💧 Humidity: 55%
```

```
*24h Forecast for Tel Aviv*
18:00: 17.8°C, ☀️ 5 km/h NORTH
22:00: 14.3°C, ☀️ 6 km/h EAST_NORTHEAST
02:00: 12.8°C, ⛅ 8 km/h NORTHEAST
06:00: 10.8°C, ☀️ 6 km/h EAST_NORTHEAST
10:00: 16.1°C, ☀️ 5 km/h SOUTH
14:00: 20.4°C, 🌤️ 8 km/h WEST_NORTHWEST
```

## 支持的位置

全球任何位置 - 只需输入城市名称：
- `New York`、`London`、`Paris`、`Berlin`、`Sydney`
- `San Francisco`、`Berlin`、`Singapore`、`Dubai`
- 或任何地址、地标或坐标

本技能使用 Google Maps API 自动进行地理编码。

## 可用数据

- **Temperature**：当前 + 体感温度
- **Conditions**：Clear、cloudy、rain、snow 等，带 emoji 图标
- **Forecast**：温度、风和状况的每小时数据
- **Humidity**：百分比
- **Wind**：Speed、direction、gusts
- **UV Index**：Sun exposure level
- **Precipitation**：Probability 和 type
- **Cloud Cover**：百分比
- **Visibility**：Distance

## 设置

1. 在 [Google Cloud Console](https://console.cloud.google.com/) 中创建项目
2. 启用 [Weather API](https://console.cloud.google.com/apis/library/weather.googleapis.com)
3. 启用 [Geocoding API](https://console.cloud.google.com/apis/library/geocoding-backend.googleapis.com)（用于位置名称查找）
4. 创建 API key 并将其设置为 `GOOGLE_API_KEY` 环境变量

> 如果你已经配置了一个，也支持 `GOOGLE_WEATHER_API_KEY` 或 `GOOGLE_MAPS_API_KEY`。

## 多语言支持

输出根据位置自适应 - 基于 `language` 参数支持 English、Hebrew 和其他语言。

```bash
# Hebrew 输出
python3 skills/google-weather/lib/weather_helper.py current "Tel Aviv"
# Output: בהיר ☀️ 19°C...
```
