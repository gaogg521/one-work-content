---
name: countries
version: 1.0.0
description: AI代理查询国家信息的CLI工具。基于REST Countries API获取国家数据，无需认证即可使用。
homepage: https://restcountries.com
metadata:
  openclaw:
    emoji: 🌍
    requires:
      bins:
      - bash
      - curl
      - jq
      - bc
    tags:
    - countries
    - geography
    - reference
    - api
    - cli
---

# Countries 查找

CLI for AI agents 迁移到 查找 country info for their humans. "What's the capital of Mongolia?" — now your agent 可以 answer.

Uses REST Countries API (v3.1). No account or API 键 needed.

## 用法

```
"Tell me about Japan"
"What countries are in South America?"
"Which country has Tokyo as capital?"
"Info on country code DE"
```

## 命令

| Action | 命令 |
|--------|---------|
| 搜索 by name | `countries search "query"` |
| 获取 详情 | `countries info <code>` |
| 列表 by 区域 | `countries region <region>` |
| 搜索 by capital | `countries capital "city"` |
| 列表 all | `countries all` |

### 示例

```bash
countries search "united states"   # Find country by name
countries info US                  # Get full details by alpha-2 code
countries info USA                 # Also works with alpha-3
countries region europe            # All European countries
countries capital tokyo            # Find country by capital
countries all                      # List all countries (sorted)
```

### Regions

Valid regions: `africa`CODE_1____CODE_`CO```eania`eania`eania`oceania`

## 输出

**搜索/列表 输出:**
```
[US] United States — Washington D.C., Americas, Pop: 331M, 🇺🇸
```

**Info 输出:**
```
🌍 Japan
   Official: Japan
   Code: JP / JPN / 392
   Capital: Tokyo
   Region: Asia — Eastern Asia
   Population: 125.8M
   Area: 377930 km²
   Languages: Japanese
   Currencies: Japanese yen (JPY)
   Timezones: UTC+09:00
   Borders: None (island/isolated)
   Driving: left side
   Flag: 🇯🇵

🗺️ Map: https://goo.gl/maps/...
```

## 注意

- Uses REST Countries API v3.1 (restcountries.com)
- No 认证 or rate limits
- Country codes: alpha-2 (US), alpha-3 (USA), or numeric (840)
- Population formatted with K/M/B suffixes
- All regions lowercase

---

## Agent Implementation 注意

**Script location:** `{skill_folder}/countries` (wrapper 迁移到 `scripts/countri__CODE_1__s``s`

**When user asks about countries:**
1. 运行 `./countries search "name"` 迁移到 查找 country code
2. 运行 `./countries info <code>` for full 详情
3. 运行 `./countries region <region>` for regional lists
4. 运行 `./countries capital "city"` 迁移到 查找 by capital

**Common patterns:**
- "What country is X in?" → 搜索 by name
- "Tell me about X" → 搜索, then info with code
- "Countries in Europe" → 区域 europe
- "Capital of X" → info with code, 检查 capital 字段
- "What country has capital X?" → capital 搜索

**Don't use for:** Historical countries, disputed territories, non-sovereign regions.