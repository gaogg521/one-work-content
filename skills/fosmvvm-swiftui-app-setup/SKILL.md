---
name: fosmvvm-swiftui-app-setup
description: 为 FOSMVVM SwiftUI 应用设置 @main App struct。配置 MVVMEnvironment、部署 URL 和测试基础设施。触发词：SwiftUI 应用设置(app setup)、FOSMVVM 环境(environment)、测试基础设施(testing infrastructure)
homepage: https://github.com/foscomputerservices/FOSUtilities
metadata: None
clawdbot: None
emoji: 🚀
os:
- darwin
tags:
- 基础设施
- 测试
---

# FOSMVVM SwiftUI 应用设置

为使用 FOSMVVM 架构的 SwiftUI 应用生成主 App struct。

## 概念基础

> 完整的架构上下文，请参阅 [FOSMVVMArchitecture.md](../../docs/FOSMVVMArchitecture.md) | [OpenClaw 参考]({baseDir}/references/FOSMVVMArchitecture.md)

**App struct** 是 SwiftUI 应用的入口点。在 FOSMVVM 中，它有三个核心职责：

```
┌─────────────────────────────────────────────────────────────┐
│                      @main App Struct                        │
├─────────────────────────────────────────────────────────────┤
│  1. MVVMEnvironment 设置                                    │
│     - Bundles (app + 本地化资源)                             │
│     - 部署 URL (production, staging, debug)                 │
│                                                              │
│  2. 环境注入                                                 │
│     - 在 WindowGroup 上使用 .environment(mvvmEnv)           │
│     - 自定义环境值                                           │
│                                                              │
│  3. 测试基础设施 (仅 DEBUG)                                  │
│     - 用于 UI 测试的 .testHost { } 修饰符                   │
│     - 用于单个视图测试的 registerTestingViews()             │
└─────────────────────────────────────────────────────────────┘
```

## 核心组件

### 1. MVVMEnvironment

`MVVMEnvironment` 为所有视图提供 FOSMVVM 基础设施：

```swift
private var mvvmEnv: MVVMEnvironment {
    MVVMEnvironment(
        appBundle: Bundle.main,
        resourceBundles: [
            MyAppViewModelsResourceAccess.localizationBundle,
            SharedResourceAccess.localizationBundle
        ],
        deploymentURLs: [
            .production: .init(serverBaseURL: URL(string: "https://api.example.com")!),
            .debug: .init(serverBaseURL: URL(string: "http://localhost:8080")!)
        ]
    )
}
```

**关键配置：**
- `appBundle` - 通常为 `Bundle.main` (应用 bundle)
- `resourceBundles` - 来自各模块的本地化 bundle 数组
- `deploymentURLs` - 每个部署环境的 URL

**Resource Bundle 访问器：**

每个包含本地化资源的模块都应提供一个 bundle 访问器：

```swift
// 在你的 ViewModels 模块中 (例如, MyAppViewModels/ResourceAccess.swift)
public enum MyAppViewModelsResourceAccess {
    public static var localizationBundle: Bundle { Bundle.module }
}
```

此模式：
- 使用 SPM 自动为每个模块提供的 `Bundle.module`
- 为访问模块资源提供干净的公共 API
- 将 bundle 访问集中在每个模块的一个位置

### 2. 环境注入

`MVVMEnvironment` 在 WindowGroup 级别注入：

```swift
var body: some Scene {
    WindowGroup {
        MyView()
    }
    .environment(mvvmEnv)  // ← 使 FOSMVVM 基础设施可用
}
```

这使环境在层级中的所有视图中可用。

### 3. 测试基础设施

测试基础设施支持使用特定配置进行 UI 测试：

**`.testHost { }` 修饰符：**
```swift
var body: some Scene {
    WindowGroup {
        ZStack {
            LandingPageView()
        }
        #if DEBUG
        .testHost { testConfiguration, testView in
            // 处理特定测试配置...

            default:
                testView
                    .onAppear {
                        underTest = ProcessInfo.processInfo.arguments.count > 1
                    }
        }
        #endif
    }
}
```

**关键点：**
- **应用于 WindowGroup 中的顶层视图** (层级中的最外层视图)
- 这确保修饰符包装整个视图层级以拦截测试配置
- 始终包含 `default:` case
- default case 通过进程参数检测测试模式
- 设置 `@State private var underTest = false` 标志
- 可选：为高级场景添加特定测试配置

**`registerTestingViews()` 函数：**
```swift
#if DEBUG
private extension MyApp {
    @MainActor func registerTestingViews() {
        mvvmEnv.registerTestView(LandingPageView.self)
        mvvmEnv.registerTestView(SettingsView.self)
        // ... 注册所有 ViewModelView 以进行单独测试
    }
}
#endif
```

**关键点：**
- 在 **App struct** 上的扩展 (不是 MVVMEnvironment)
- 从 `init()` 调用
- 注册每个 ViewModelView 以进行隔离测试
- 仅 DEBUG

## 何时使用此技能

- 启动新的 FOSMVVM SwiftUI 应用
- 将现有 SwiftUI 应用迁移到 FOSMVVM
- 使用正确的 FOSMVVM 基础设施设置 App struct
- 为 UI 测试配置测试基础设施

## 此技能生成的内容

| 组件 | 位置 | 用途 |
|-----------|----------|---------|
| 主 App struct | `Sources/App/{AppName}.swift` | 带 MVVMEnvironment 设置的入口点 |
| MVVMEnvironment 配置 | App struct 中的计算属性 | Bundles 和部署 URL |
| 测试基础设施 | App struct 中的 DEBUG 块 | UI 测试支持 |

## 项目结构配置

| 占位符 | 描述 | 示例 |
|-------------|-------------|---------|
| `{AppName}` | 你的应用名称 | `MyApp`, `AccelApp` |
| `{AppTarget}` | 主应用 target | `App` |
| `{ResourceBundles}` | 带本地化的模块名称 | `MyAppViewModels`, `SharedResources` |

## 如何使用此技能

**调用：**
/fosmvvm-swiftui-app-setup

**先决条件：**
- 从对话上下文中理解应用名称
- 讨论或记录部署 URL
- 识别资源 bundles (带本地化的模块)
- 明确测试支持需求

**工作流集成：**
此技能用于设置新的 FOSMVVM SwiftUI 应用或将 FOSMVVM 基础设施添加到现有应用时。技能自动引用对话上下文——无需文件路径或问答。

## 模式实现

此技能引用对话上下文来确定 App struct 配置：

### 配置检测

从对话上下文中，技能识别：
- **应用名称** (来自项目讨论或现有代码)
- **部署环境** (production, staging, debug URL)
- **资源 bundles** (包含本地化 YAML 文件的模块)
- **测试基础设施** (是否需要 UI 测试支持)

### MVVMEnvironment 设置

基于项目结构：
- **应用 bundle** (通常为 Bundle.main)
- **资源 bundle 访问器** (来自已识别模块)
- **部署 URL** (每个环境)
- **当前版本** (来自共享模块)

### 测试基础设施规划

如果需要测试支持：
- **测试检测** (进程参数检查)
- **测试主机修饰符** (包装顶层视图)
- **视图注册** (所有 ViewModelViews 用于测试)

### 文件生成

1. 带 @main 属性的主 App struct
2. MVVMEnvironment 计算属性
3. 带环境注入的 WindowGroup
4. 测试基础设施 (如果请求，仅 DEBUG)
5. registerTestingViews() 扩展 (如果测试支持)

### 上下文来源

技能引用来自以下信息：
- **先前对话**：应用需求、讨论的部署环境
- **项目结构**：来自代码库分析的模块组织
- **现有模式**：如果上下文可用，来自其他 FOSMVVM 应用

## 关键模式

### MVVMEnvironment 作为计算属性

`MVVMEnvironment` 是计算属性，不是存储属性：

```swift
private var mvvmEnv: MVVMEnvironment {
    MVVMEnvironment(
        appBundle: Bundle.main,
        resourceBundles: [...],
        deploymentURLs: [...]
    )
}
```

**为什么是计算的？**
- 将初始化逻辑分离
- 可在 DEBUG 与 RELEASE 中自定义
- 清晰依赖 bundles 和 URL

### 测试检测模式

默认测试检测使用进程参数：

```swift
@State private var underTest = false

// 在 .testHost default case 中：
testView
    .onAppear {
        // 目前没有别的方法来检测应用是否在测试中。
        // 这只是 debug 代码，所以我们可以暂时继续。
        underTest = ProcessInfo.processInfo.arguments.count > 1
    }
```

**为什么用此方法？**
- 对 DEBUG 构建简单可靠
- 无额外依赖
- 进程参数由测试运行器设置

### 注册所有 ViewModelViews

每个 ViewModelView 都应注册用于测试：

```swift
@MainActor func registerTestingViews() {
    // Landing Page
    mvvmEnv.registerTestView(LandingPageView.self)

    // Settings
    mvvmEnv.registerTestView(SettingsView.self)
    mvvmEnv.registerTestView(ProfileView.self)

    // Dashboard
    mvvmEnv.registerTestView(DashboardView.self)
    mvvmEnv.registerTestView(CardView.self)
}
```

**组织提示：**
- 按功能/屏幕分组并加注释
- 每组内按字母顺序
- 每行一个视图，便于扫描

## 常见自定义

### 多个环境值

你可以注入多个环境值：

```swift
var body: some Scene {
    WindowGroup {
        MyView()
    }
    .environment(mvvmEnv)
    .environment(appState)
    .environment(\.colorScheme, .dark)
    .environment(\.customValue, myCustomValue)
}
```

### 条件测试注册

你可以基于构建配置条件注册视图：

```swift
#if DEBUG
@MainActor func registerTestingViews() {
    mvvmEnv.registerTestView(LandingPageView.self)

    #if INCLUDE_ADMIN_FEATURES
    mvvmEnv.registerTestView(AdminPanelView.self)
    #endif
}
#endif
```

### 高级测试配置

你可以在 `.testHost` 中添加特定测试配置：

```swift
.testHost { testConfiguration, testView in
    switch try? testConfiguration.fromJSON() as MyTestConfiguration {
    case .specificScenario(let data):
        testView.environment(MyState.stub(data: data))
            .onAppear { underTest = true }

    default:
        testView
            .onAppear {
                underTest = ProcessInfo.processInfo.arguments.count > 1
            }
    }
}
```

## 文件模板

完整的文件模板请参阅 [reference.md](reference.md)。

## 命名约定

| 概念 | 约定 | 示例 |
|---------|------------|---------|
| App struct | `{Name}App` | `MyApp`, `AccelApp` |
| 主文件 | `{Name}App.swift` | `MyApp.swift` |
| MVVMEnvironment 属性 | `mvvmEnv` | 始终 `mvvmEnv` |
| 测试标志 | `underTest` | 始终 `underTest` |

## 部署配置

FOSMVVM 通过 Info.plist 支持部署检测：

```
CI Pipeline Sets:
   FOS_DEPLOYMENT build setting (例如, "staging" 或 "production")
        ↓
Info.plist Contains:
   FOS-DEPLOYMENT = $(FOS_DEPLOYMENT)
        ↓
Runtime Detection:
   FOSMVVM.Deployment.current reads from Bundle.main.infoDictionary
```

**本地开发覆盖：**
- Edit Scheme → Run → Arguments → Environment Variables
- 添加: `FOS-DEPLOYMENT = staging`

## 另请参阅

- [Architecture Patterns](../shared/architecture-patterns.md) - 心智模型和模式
- [FOSMVVMArchitecture.md](../../docs/FOSMVVMArchitecture.md) - 完整 FOSMVVM 架构
- [fosmvvm-viewmodel-generator](../fosmvvm-viewmodel-generator/SKILL.md) - 用于创建 ViewModels
- [reference.md](reference.md) - 完整文件模板

## 版本历史

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-23 | SwiftUI 应用设置的初始技能 |
| 1.1 | 2026-01-24 | 更新为上下文感知方法（移除文件解析/问答）。技能引用对话上下文而不是提问或接受文件路径。 |