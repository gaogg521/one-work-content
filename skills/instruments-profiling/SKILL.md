---
name: instruments-profiling
description: 使用 Instruments/xctrace 对原生 macOS/iOS app 进行性能分析(profiling)。涵盖 binary 选择、CLI 参数、导出(export)及常见陷阱。触发词：性能分析(profiling)、Instruments、xctrace、macOS/iOS
metadata: None
short-description: 针对 macOS/iOS app 的 Instruments 性能分析
tags:
- 性能优化
---

# Instruments 性能分析 (macOS/iOS)

当用户希望对原生 app 进行性能分析或堆栈分析时使用此 skill。
重点：Time Profiler、`xctrace` CLI 以及选择正确的 binary/app 实例。

## 快速开始 (CLI)

- 列出模板：`xcrun xctrace list templates`
- 记录 Time Profiler（启动）：
  - `xcrun xctrace record --template 'Time Profiler' --time-limit 60s --output /tmp/App.trace --launch -- /path/To/App.app`
- 记录 Time Profiler（附加）：
  - 自行启动 app，获取 PID，然后：
  - `xcrun xctrace record --template 'Time Profiler' --time-limit 60s --output /tmp/App.trace --attach <pid>`
- 在 Instruments 中打开 trace：
  - `open -a Instruments /tmp/App.trace`

注意：`xcrun xctrace --help` 不是有效的子命令。使用 `xcrun xctrace help record`。

## 选择正确的 Binary（关键）

**陷阱：Instruments 可能会分析错误的 app**（例如，LaunchServices 解析到了 `/Applications` 中的另一个 bundle）。
使用以下规则：

- 优先使用直接的 binary 路径以实现确定性启动：
  - `xcrun xctrace record ... --launch -- /path/App.app/Contents/MacOS/App`
- 如果启动 `.app`，确保它是预期的 bundle：
  - `open -n /path/App.app`
  - 使用 `ps -p <pid> -o comm= -o command=` 进行验证
- 如果 `/Applications/App.app` 和本地构建同时存在，显式指定本地构建路径。
- 启动后，在信任 trace 之前确认进程路径。

## 命令参数 (xctrace)

- `--template 'Time Profiler'`：来自 `xctrace list templates` 的模板名称。
- `--launch -- <cmd>`：`--` 之后的所有内容都是目标命令（binary 或 app bundle）。
- `--attach <pid|name>`：附加到正在运行的进程。
- `--output <path>`：`.trace` 输出路径。如果省略，文件将保存在当前工作目录。
- `--time-limit 60s|5m`：设置捕获时长。
- `--device <name|UDID>`：针对 iOS 设备运行时需要。
- `--target-stdout -`：将启动的进程 stdout 流式传输到终端（对 CLI 工具很有用）。

## 导出堆栈 (CLI)

- 检查 trace 表格：
  - `xcrun xctrace export --input /tmp/App.trace --toc`
- 导出原始 time-profile 样本：
  - `xcrun xctrace export --input /tmp/App.trace --xpath '/trace-toc/run[@number="1"]/data/table[@schema="time-profile"]' --output /tmp/time-profile.xml`
- 在脚本（Python/Rust）中进行后处理以聚合堆栈。

## Instruments UI 工作流

- 模板：Time Profiler
- 使用“Record”并捕获慢路径（启动 vs 稳态）
- Call Tree 提示：
  - Hide System Libraries
  - Invert Call Tree
  - Separate by Thread
  - 关注热点帧和调用次数

## 陷阱与修复

- **分析了错误的 app**：LaunchServices 解析到了已安装的 app 而非本地构建。
  - 修复：使用直接的 binary 路径或 `--attach` 配合已知 PID。
- **没有样本 / 空 trace**：App 快速退出或从未执行工作。
  - 修复：延长捕获时间，在录制期间触发工作负载。
- **隐私提示**：`xctrace` 可能需要 Developer Tools 权限。
  - 修复：System Settings → Privacy & Security → Developer Tools → 允许 Terminal/Xcode。
- **大型 XML 导出**：`time-profile` 导出体积巨大。
  - 修复：使用 XPath 过滤并离线聚合；不要打印到终端。

## iOS 特定说明

- 设备：使用 `xcrun xctrace list devices` 和 `--device <UDID>`。
- 如果需要，通过 Xcode 启动；使用 `xctrace --attach` 附加。
- 确保有 debug symbols 以获得有意义的堆栈。

## 验证清单

- 确认 trace 进程路径与目标构建匹配。
- 确认堆栈显示预期的 app 帧。
- 捕获覆盖慢操作（启动/刷新）。
- 如果进行优化，导出堆栈用于自动化对比。
