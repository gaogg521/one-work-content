---
name: asc-release-flow
description: 使用 asc publish、builds、versions 与 submit 命令的 TestFlight 与 App Store 端到端发布工作流。支持上传构建(build upload)、TestFlight 分发(TestFlight distribution) 与 App Store 提交(App Store submission)。触发词：TestFlight 发布(TestFlight release)、App Store 提交(App Store submit)、构建上传(build upload)、asc 发布(asc publish)。
tags:
- 发布工程
---

# 发布流程 (TestFlight 和 App Store)

当你需要将新构建上传到 TestFlight 或提交到 App Store 时使用此技能。

## 前置条件
- 确保凭据已设置 (`asc auth login` 或 `ASC_*` 环境变量)。
- 每次上传使用新的构建号。
- 优先使用 `ASC_APP_ID` 或显式传递 `--app`。

## 首选端到端命令
- TestFlight:
  - `asc publish testflight --app <APP_ID> --ipa <PATH> --group <GROUP_ID>[,<GROUP_ID>]`
  - 可选：`--wait`, `--notify`, `--platform`, `--poll-interval`, `--timeout`
- App Store:
  - `asc publish appstore --app <APP_ID> --ipa <PATH> --version <VERSION>`
  - 可选：`--wait`, `--submit --confirm`, `--platform`, `--poll-interval`, `--timeout`

## 手动序列（当你需要更多控制时）
1. 上传构建：
   - `asc builds upload --app <APP_ID> --ipa <PATH>`
2. 如果需要，查找构建 ID：
   - `asc builds latest --app <APP_ID> [--version <VERSION>] [--platform <PLATFORM>]`
3. TestFlight 分发：
   - `asc builds add-groups --build <BUILD_ID> --group <GROUP_ID>[,<GROUP_ID>]`
4. App Store 附加 + 提交：
   - `asc versions attach-build --version-id <VERSION_ID> --build <BUILD_ID>`
   - `asc submit create --app <APP_ID> --version <VERSION> --build <BUILD_ID> --confirm`
5. 检查或取消提交：
   - `asc submit status --id <SUBMISSION_ID>` 或 `--version-id <VERSION_ID>`
   - `asc submit cancel --id <SUBMISSION_ID> --confirm`

## 注意事项
- 始终使用 `--help` 验证确切命令的标志。
- 使用 `--output table` / `--output markdown` 获取人类可读的输出；默认是 JSON。
