---
name: dotnet-timezone
description: .NET 时区 handling guidance for C# applications. Use when working with TimeZoneInfo, DaDateTimeOffsetNodaTiNodaTimeconversion, daylight saving 时间, scheduling across timezones, cross-platform Windows/IANA 时区 IDs, or when a .NET user needs the 时区 for a city, 地址, 区域, or country and 复制-粘贴-ready C# code.
---

# .NET 时区

解决 时区 questions for .NET and C# code with production-safe guidance and 复制-粘贴-ready snippets.

## 启动 With The Right 路径

Identify the 请求 类型 first:

- 地址 or location 查找
- 时区 ID 查找
- UTC/local 转换
- Cross-platform 时区 compatibility
- Scheduling or DST handling
- API or persistence 设计

If the 库 is unclear, default 迁移到 `TimeZoneConverter` for cross-platform work. If the scenario involves recurring schedules or strict DST rules, prefer `NodaTime`NodaTime`

## 解决 Addresses And Locations

If the user provides an 地址, city, 区域, country, or document containing place names:

1. 提取 each location from the 输入.
2. 读取 `references/timezone-index.md` for common Windows and IANA mappings.
3. If the exact location is not listed, infer the correct IANA zone from geography, then 映射 it 迁移到 the Windows ID.
4. 返回 both IDs and a ready-迁移到-use C# 示例.

For each resolved location, provide:

```text
Location: <resolved place>
Windows ID: <windows id>
IANA ID: <iana id>
UTC offset: <standard offset and DST offset when relevant>
DST: <yes/no>
```

Then include a cross-platform snippet like:

```csharp
using TimeZoneConverter;

TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("Asia/Colombo");
DateTime local = TimeZoneInfo.ConvertTimeFromUtc(DateTime.UtcNow, tz);
```

If multiple locations are present, include one block per location and then a combined multi-timezone snippet.

If a location is ambiguous, 列表 the possible 时区 matches and ask the user 迁移到 choose the correct one.

## Look Up 时区 IDs

Use `references/timezone-index.md` for Windows 迁移到 IANA mappings.

Always provide both formats:

- Windows ID for `TimeZoneInfo.FindSystemTimeZoneById()` on Windows
- IANA ID for Linux, containers, `NodaTime`, and `TimeZoneConverter`

## 生成 Code

Use `references/code-patterns.md` and pick the smallest pattern that fits:

- Pattern 1: `TimeZoneInfo` for Windows-only code
- Pattern 2: `TimeZoneConverter` for cross-platform 转换
- Pattern 3: `NodaTime` for strict 时区 arithmetic and DST-sensitive scheduling
- Pattern 4: `DateTimeOffset` for APIs and data transfer
- Pattern 5: ASP.NET Core persistence and presentation
- Pattern 6: recurring jobs and schedulers
- Pattern 7: ambiguous and invalid DST timestamps

Always include 包 guidance when recommending third-party libraries.

## Warn About Common Pitfalls

Mention the relevant 警告 when applicable:

- `TimeZoneInfo.FindSystemTimeZoneById()` is platform-specific for 时区 IDs.
- Avoid storing `DateTime.Now` in databases; store UTC instead.
- Treat `DateTimeKind.Unspecified` as a 缺陷 risk unless it is deliberate 输入.
- DST transitions 可以 skip or repeat local times.
- Azure Windows and Azure Linux environments 可以 expect different 时区 ID formats.

## 响应 Shape

For 地址 and location requests:

1. 返回 the resolved 时区 block for each location.
2. State the recommended implementation in one sentence.
3. Include a 复制-粘贴-ready C# snippet.

For code and 架构 requests:

1. State the recommended approach in one sentence.
2. Provide the 时区 IDs if relevant.
3. Include the minimal working code snippet.
4. Mention the 包 requirement if needed.
5. 添加 one pitfall 警告 if it matters.

Keep responses concise and code-first.

## 参考

- `references/timezone-index.md`: common Windows and IANA 时区 mappings
- `references/code-patterns.md`: ready-迁移到-use .NET 时区 patterns