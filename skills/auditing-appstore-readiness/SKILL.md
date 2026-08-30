---
name: auditing-appstore-readiness
description: 审计 iOS 应用仓库（Swift/Xcode 或 React Native/Expo）的 App Store 合规性(compliance)与发布就绪性(readiness)。输出通过/警告/失败报告与发布检查清单。触发词：App Store 审计(App Store audit)、发布就绪(release readiness)、合规检查(compliance check)、iOS 审计(iOS audit)。
metadata: None
openclaw: None
emoji: 🧾
requires: None
bins:
- git
tags:
- 安全
- 发布工程
---

# App Store 就绪性审计

此技能审查应用仓库并为 iOS **App Store** / **TestFlight** 提交生成发布就绪性报告。

它支持：
- 原生 iOS (Swift/Obj‑C, Xcode project/workspace)
- React Native (bare)
- Expo (managed 或 prebuild)

## 快速开始 (推荐)

从仓库根目录运行只读审计脚本：

{ "tool": "exec", "command": "node {baseDir}/scripts/audit.mjs --repo . --format md" }

如果你也想要 JSON 输出：

{ "tool": "exec", "command": "node {baseDir}/scripts/audit.mjs --repo . --format md --json audit.json" }

如果仓库是 monorepo，指向应用目录：

{ "tool": "exec", "command": "node {baseDir}/scripts/audit.mjs --repo apps/mobile --format md" }

## 输出约定

始终返回：
- 总体裁决：**PASS** / **WARN** / **FAIL**
- 检测到的项目风格和关键标识符 (bundle id, version, build)
- 带有证据和修复步骤的检查列表
- 开发者可以勾选的**发布检查清单**

使用：[references/report-template.md](references/report-template.md)

## 安全规则（不要破坏仓库）

默认使用**只读**命令。不要运行修改工作区的命令，除非：
- 用户明确要求，**或者**
- 修复是微不足道的且明显被期望的（然后先解释将会改变什么）

变更命令示例：
- 依赖安装 (`npm i`, `yarn`, `pnpm i`, `pod install`)
- 配置生成 (`expo prebuild`)
- 签名自动化 (`fastlane match`)
- 归档 (`xcodebuild archive`, `eas build`) — 创建产物并可能需要签名

如果你必须运行变更命令，在运行前清楚地标记为 **MUTATING**。

## 主要工作流程

### 1) 识别仓库和项目风格

优先使用脚本检测 (`audit.mjs`)。如果手动进行：

- Expo 可能：`package.json` 包含 `expo` 且存在 `app.json` / `app.config.*`
- React Native (bare)：`package.json` 包含 `react-native` 且存在 `ios/`
- 原生 iOS：存在 `*.xcodeproj` 或 `*.xcworkspace`

如果存在多个应用，选择匹配用户意图的一个；否则选择具有以下条件的目录：
- 单个 `ios/<AppName>/Info.plist`，且
- 根目录附近恰好有一个 `.xcodeproj` 或 `.xcworkspace`。

### 2) 运行静态合规检查（到处都有效）

即使没有 Xcode 也运行这些检查：

- 仓库卫生：干净的 git 状态；没有提交明显的 secrets
- iOS 标识符：bundle id, version, build number
- 应用图标：包含 App Store (1024×1024) 图标
- 启动屏存在
- 隐私与权限：
  - 隐私清单存在 (`PrivacyInfo.xcprivacy`) 或明确 accounted for
  - 相关时存在权限使用字符串 (camera, location, tracking, 等)
  - 避免广泛的 ATS 豁免 (`NSAllowsArbitraryLoads`)
- 第三方 SDK 卫生：许可证、隐私清单、跟踪披露
- 商店列表基础：隐私政策 URL 存在于仓库/文档某处；支持/联系信息

脚本为这些输出 PASS/WARN/FAIL。

### 3) 运行构建准确性检查（macOS + Xcode，可选但高置信度）

仅当你有 **Xcode** 可用时（本地 macOS 网关或配对的 macOS 节点）。

推荐序列（创建构建产物）：

1) 显示 Xcode + SDK 版本：
{ "tool": "exec", "command": "xcodebuild -version" }

2) 列出 schemes（按检测到的 project/workspace）：
{ "tool": "exec", "command": "xcodebuild -list -json -workspace <path>.xcworkspace" }
或
{ "tool": "exec", "command": "xcodebuild -list -json -project <path>.xcodeproj" }

3) 模拟器发布构建（快速，避免签名）：
{ "tool": "exec", "command": "xcodebuild -workspace <...> -scheme <...> -configuration Release -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 15' build" }

4) 如果你需要分发产物（**MUTATING / 签名**）：
- 如果已配置，优先使用 Fastlane
- 否则 `xcodebuild archive` + `xcodebuild -exportArchive`

如果无法进行构建检查，报告必须明确说明，并将裁决保持在 **WARN**（除非有明确的 FAIL 项）。

### 4) 生成最终就绪性报告

- 使用 [references/report-template.md](references/report-template.md)
- 包含 "Go / No‑Go" 建议：
  - **FAIL** → 提交前必须修复
  - **WARN** → 提交可能成功，但风险区域仍然存在
  - **PASS** → 准备提交；剩余项目是行政性的

## 代理无法完全验证的手动检查

始终将这些作为最终检查清单部分包含（即使自动化检查通过）：

- App Store Connect 元数据：截图、描述、关键词、年龄分级、定价、类别
- 隐私营养标签匹配实际行为
- 出口合规（加密）答案正确
- 内容/IP 权利：许可证、第三方资产、商标
- 账户/区域要求（如适用，例如欧盟交易者状态）
- 如果使用，应用内购买/订阅已配置

参见：[references/manual-checklist.md](references/manual-checklist.md)

## 当用户问 "使其合规" 时

切换到修复模式：
1) 识别可以在仓库中安全修复的失败项 (Info.plist 字符串, `PrivacyInfo.xcprivacy` 模板, ATS 例外收紧, 等)
2) 提出最小补丁并使用 `apply_patch` 应用
3) 重新运行 `audit.mjs` 并更新报告

## 快速搜索

- 权限映射：[references/permissions-map.md](references/permissions-map.md)
- Expo 特定检查：[references/expo.md](references/expo.md)
- React Native iOS 检查：[references/react-native.md](references/react-native.md)
- 原生 iOS 检查：[references/native-ios.md](references/native-ios.md)
