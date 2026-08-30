---
name: fosmvvm-viewmodel-generator
description: 为 SwiftUI 屏幕、页面和组件生成 FOSMVVM ViewModels。搭建 RequestableViewModel、本地化绑定(localization binding)和 stub 工厂(factory)。触发词：ViewModel 生成(ViewModel generation)、FOSMVVM 架构(architecture)、SwiftUI 组件(component)
homepage: https://github.com/foscomputerservices/FOSUtilities
metadata: None
clawdbot: None
emoji: 🏗️
os:
- darwin
- linux
tags:
- DNS
- 架构
---

# FOSMVVM ViewModel 生成器

按照 FOSMVVM 架构模式生成 ViewModels。

## 概念基础

> 完整的架构上下文，请参阅 [FOSMVVMArchitecture.md](../../docs/FOSMVVMArchitecture.md) | [OpenClaw 参考]({baseDir}/references/FOSMVVMArchitecture.md)

**ViewModel** 是 Model-View-ViewModel 架构中的桥梁：

```
┌─────────────┐      ┌─────────────────┐      ┌─────────────┐
│    Model    │ ───► │    ViewModel    │ ───► │    View     │
│   (Data)    │      │  (The Bridge)   │      │  (SwiftUI)  │
└─────────────┘      └─────────────────┘      └─────────────┘
```

**关键洞察：** 在 FOSMVVM 中，ViewModels：
- **由 Factory 创建** (服务端或客户端)
- **在编码期间本地化** (解析所有 `@LocalizedString` 引用)
- **由 Views 消费**，Views 只渲染本地化后的数据

---

## 首要决策：托管模式

**这是每个 ViewModel 的决策。** 应用可以混合两种模式——例如，一个独立的 iPhone 应用配合基于服务器的登录。

**关键问题：这个 ViewModel 的数据从哪里来？**

| 数据来源 | 托管模式 | Factory |
|-------------|--------------|---------|
| 服务器/数据库 | Server-Hosted | 手写 |
| 本地状态/偏好设置 | Client-Hosted | 宏生成 |
| **ResponseError (捕获的错误)** | **Client-Hosted** | 宏生成 |

### Server-Hosted 模式

当数据来自服务器时：
- Factory 在服务器上**手写** (`ViewModelFactory` 协议)
- Factory 查询数据库，构建 ViewModel
- 服务器在 JSON 编码期间本地化
- 客户端接收完全本地化的 ViewModel

**示例：** 登录屏幕、来自 API 的用户资料、带服务器数据的仪表板

### Client-Hosted 模式

当数据在设备本地时：
- 使用 `@ViewModel(options: [.clientHostedFactory])`
- 宏**自动生成**来自 init 参数的 factory
- 客户端打包 YAML 资源
- 客户端在编码期间本地化

**示例：** 设置屏幕、引导页、离线优先功能、**错误显示**

### 错误显示模式

错误显示是典型的 client-hosted 场景。你已经从 `ResponseError` 获得了数据——只需将其包装在该错误的**特定** ViewModel 中：

```swift
// 针对 MoveIdeaRequest 错误的特定 ViewModel
@ViewModel(options: [.clientHostedFactory])
struct MoveIdeaErrorViewModel {
    let message: LocalizableString
    let errorCode: String

    public var vmId = ViewModelId()

    // 接收特定的 ResponseError
    init(responseError: MoveIdeaRequest.ResponseError) {
        self.message = responseError.message
        self.errorCode = responseError.code.rawValue
    }
}
```

用法：
```swift
catch let error as MoveIdeaRequest.ResponseError {
    let vm = MoveIdeaErrorViewModel(responseError: error)
    return try await req.view.render("Shared/ToastView", vm)
}
```

**每个错误场景都有自己的 ViewModel：**
- `MoveIdeaErrorViewModel` 用于 `MoveIdeaRequest.ResponseError`
- `CreateIdeaErrorViewModel` 用于 `CreateIdeaRequest.ResponseError`
- `SettingsValidationErrorViewModel` 用于设置表单错误

不要创建通用的 "ToastViewModel" 或 "ErrorViewModel"——那是统一错误架构，我们避免使用。

**关键洞察：**
- 不需要服务器请求——你已经捕获了错误
- `ResponseError` 中的 `LocalizableString` 属性**已经本地化** (服务器已完成)
- 标准 ViewModel → View 编码链正确处理；已本地化的字符串原样传递
- Client-hosted ViewModel 包装现有数据；宏生成 factory

### 混合应用

许多应用同时使用两者：
```
┌───────────────────────────────────────────────┐
│               iPhone App                       │
├───────────────────────────────────────────────┤
│ SettingsViewModel           → Client-Hosted   │
│ OnboardingViewModel         → Client-Hosted   │
│ MoveIdeaErrorViewModel      → Client-Hosted   │  ← 错误显示
│ SignInViewModel             → Server-Hosted   │
│ UserProfileViewModel        → Server-Hosted   │
└───────────────────────────────────────────────┘
```

**两种模式使用相同的 ViewModel 模式**——只有 factory 创建不同。

### 核心职责：数据塑形

ViewModel 的工作是**为展示塑形数据**。这发生在两个地方：

1. **Factory** - *什么*数据需要，*如何*转换
2. **Localization** - *如何*在上下文中呈现 (包括区域感知排序)

**View 只负责渲染**——它永远不应该组合、格式化或重新排序 ViewModel 属性。

### ViewModel 包含什么

ViewModel 回答：**"View 需要显示什么？"**

| 内容类型 | 表示方式 | 示例 |
|--------------|---------------------|---------|
| 静态 UI 文本 | `@LocalizedString` | 页面标题、按钮标签 (固定文本) |
| 动态枚举值 | `LocalizableString` (存储) | 状态/状态显示 (参见枚举本地化模式) |
| 文本中的动态数据 | `@LocalizedSubs` | "Welcome, %{name}!" 带替换 |
| 组合文本 | `@LocalizedCompoundString` | 由片段组成的全名 (区域感知顺序) |
| 格式化日期 | `LocalizableDate` | `createdAt: LocalizableDate` |
| 格式化数字 | `LocalizableInt` | `totalCount: LocalizableInt` |
| 动态数据 | 普通属性 | `content: String`, `count: Int` |
| 嵌套组件 | 子 ViewModels | `cards: [CardViewModel]` |

### ViewModel 不包含什么

- 数据库关系 (`@Parent`, `@Siblings`)
- 业务逻辑或验证 (在 Fields 协议中)
- 暴露给模板的原始数据库 ID (使用类型化属性)
- View 必须查找的未本地化字符串

### 反模式：在 View 中组合

```swift
// ❌ 错误 - View 正在组合
Text(viewModel.firstName) + Text(" ") + Text(viewModel.lastName)

// ✅ 正确 - ViewModel 提供塑形结果
Text(viewModel.fullName)  // 通过 @LocalizedCompoundString
```

如果你在 View 中看到 `+` 或字符串插值，塑形属于 ViewModel。

## ViewModel 协议层级

```swift
public protocol ViewModel: ServerRequestBody, RetrievablePropertyNames, Identifiable, Stubbable {
    var vmId: ViewModelId { get }
}

public protocol RequestableViewModel: ViewModel {
    associatedtype Request: ViewModelRequest
}
```

**ViewModel** 提供：
- `ServerRequestBody` - 可以作为 JSON 通过 HTTP 发送
- `RetrievablePropertyNames` - 启用 `@LocalizedString` 绑定 (通过 `@ViewModel` 宏)
- `Identifiable` - 具有 `vmId` 用于 SwiftUI identity
- `Stubbable` - 具有 `stub()` 用于测试/预览

**RequestableViewModel** 增加：
- 用于从服务器获取的关联 `Request` 类型

## 两类 ViewModels

### 1. 顶级 (RequestableViewModel)

代表完整页面或屏幕。具有：
- 关联的 `ViewModelRequest` 类型
- 从数据库构建它的 `ViewModelFactory`
- 嵌入其中的子 ViewModels

```swift
@ViewModel
public struct DashboardViewModel: RequestableViewModel {
    public typealias Request = DashboardRequest

    @LocalizedString public var pageTitle
    public let cards: [CardViewModel]  // 子项
    public var vmId: ViewModelId = .init()
}
```

### 2. 子级 (普通 ViewModel)

由父级 factory 构建的嵌套组件。没有 Request 类型。

```swift
@ViewModel
public struct CardViewModel: Codable, Sendable {
    public let id: ModelIdType
    public let title: String
    public let createdAt: LocalizableDate
    public var vmId: ViewModelId = .init()
}
```

---

## Display vs Form ViewModels

ViewModels 服务于两个不同的目的：

| 目的 | ViewModel 类型 | 是否采纳 Fields? |
|---------|----------------|----------------|
| **显示数据** (只读) | Display ViewModel | No |
| **收集用户输入** (可编辑) | Form ViewModel | Yes |

### Display ViewModels

用于显示数据——卡片、行、列表、详情视图：

```swift
@ViewModel
public struct UserCardViewModel {
    public let id: ModelIdType
    public let name: String
    @LocalizedString public var roleDisplayName
    public let createdAt: LocalizableDate
    public var vmId: ViewModelId = .init()
}
```

**特征：**
- 属性为 `let` (只读)
- 不需要验证
- 没有 FormField 定义
- 只是为显示投影 Model 数据

### Form ViewModels

用于收集输入——创建表单、编辑表单、设置：

```swift
@ViewModel
public struct UserFormViewModel: UserFields {  // ← 采纳 Fields!
    public var id: ModelIdType?
    public var email: String
    public var firstName: String
    public var lastName: String

    public let userValidationMessages: UserFieldsMessages
    public var vmId: ViewModelId = .init()
}
```

**特征：**
- 属性为 `var` (可编辑)
- **采纳 Fields 协议** 用于验证
- 从 Fields 获取 FormField 定义
- 从 Fields 获取验证逻辑
- 从 Fields 获取本地化错误消息

### 连接关系

```
┌─────────────────────────────────────────────────────────────────┐
│                    UserFields Protocol                          │
│        (定义可编辑属性 + 验证)                                   │
│                                                                 │
│  采纳者:                                                        │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │ CreateUserReq   │  │ UserFormVM      │  │ User (Model)    │ │
│  │ .RequestBody    │  │ (UI form)       │  │ (persistence)   │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
│                                                                 │
│  到处使用相同的验证逻辑！                                        │
└─────────────────────────────────────────────────────────────────┘
```

### 快速决策指南

**关键问题："用户是否在这个 ViewModel 中编辑数据？"**

- **否** → Display ViewModel (无 Fields)
- **是** → Form ViewModel (采纳 Fields)

| ViewModel | 用户编辑? | 采纳 Fields? |
|-----------|-------------|---------------|
| `UserCardViewModel` | No | No |
| `UserRowViewModel` | No | No |
| `UserDetailViewModel` | No | No |
| `UserFormViewModel` | Yes | `UserFields` |
| `CreateUserViewModel` | Yes | `UserFields` |
| `EditUserViewModel` | Yes | `UserFields` |
| `SettingsViewModel` | Yes | `SettingsFields` |

---

## 何时使用此技能

- 创建新页面或屏幕
- 添加新 UI 组件 (卡片、行、模态框等)
- 在 View 中显示数据库数据
- 遵循需要新 ViewModels 的实施计划

## 此技能生成的内容

### Server-Hosted：顶级 ViewModel (4 个文件)

| 文件 | 位置 | 用途 |
|------|----------|---------|
| `{Name}ViewModel.swift` | `{ViewModelsTarget}/` | ViewModel struct |
| `{Name}Request.swift` | `{ViewModelsTarget}/` | ViewModelRequest 类型 |
| `{Name}ViewModel.yml` | `{ResourcesPath}/` | 本地化字符串 |
| `{Name}ViewModel+Factory.swift` | `{WebServerTarget}/` | 从数据库构建的 Factory |

### Client-Hosted：顶级 ViewModel (2 个文件)

| 文件 | 位置 | 用途 |
|------|----------|---------|
| `{Name}ViewModel.swift` | `{ViewModelsTarget}/` | 带 `clientHostedFactory` 选项的 ViewModel |
| `{Name}ViewModel.yml` | `{ResourcesPath}/` | 本地化字符串 (打包在应用中) |

*不需要 Request 或 Factory 文件——宏生成它们！*

### 子 ViewModels (1-2 个文件，任一模式)

| 文件 | 位置 | 用途 |
|------|----------|---------|
| `{Name}ViewModel.swift` | `{ViewModelsTarget}/` | ViewModel struct |
| `{Name}ViewModel.yml` | `{ResourcesPath}/` | 本地化 (如果有 `@LocalizedString`) |

**注意：** 如果子项仅被一个父项使用，并且代表摘要/引用 (不是完整的 ViewModel)，则将其嵌套在父文件内。参见 **Key Patterns** 下的 **Nested Child Types Pattern**。

## 项目结构配置

| 占位符 | 描述 | 示例 |
|-------------|-------------|---------|
| `{ViewModelsTarget}` | 共享 ViewModels SPM target | `ViewModels` |
| `{ResourcesPath}` | 本地化资源 | `Sources/Resources` |
| `{WebServerTarget}` | 服务端 target | `WebServer`, `AppServer` |

## 如何使用此技能

**调用：**
/fosmvvm-viewmodel-generator

**先决条件：**
- 从对话上下文中理解视图需求
- 确定数据源 (服务器/数据库 vs 本地状态)
- 做出 Display vs Form 决策 (如果涉及用户输入，Fields 协议已存在)

**工作流集成：**
此技能通常在讨论视图需求或阅读规范文件后使用。技能自动引用对话上下文——无需文件路径或问答。对于 Form ViewModels，先运行 fosmvvm-fields-generator 以创建 Fields 协议。

## 模式实现

此技能引用对话上下文来确定 ViewModel 结构：

### 托管模式检测

从对话上下文中，技能识别：
- **数据源** (服务器/数据库 vs 本地状态/偏好设置)
- Server-hosted → 手写 factory，服务端本地化
- Client-hosted → 宏生成 factory，客户端本地化

### ViewModel 设计

从上下文中的需求：
- **视图用途** (页面、模态框、卡片、行组件)
- **数据需求** (来自数据库查询、来自 AppState、来自捕获的错误)
- **静态 UI 文本** (标题、标签、按钮需要 @LocalizedString)
- **子 ViewModels** (嵌套组件)
- **层级级别** (顶级 RequestableViewModel vs 子 ViewModel)

### 属性规划

基于视图需求：
- **显示属性** (要渲染的数据)
- **本地化需求** (哪些属性使用 @LocalizedString)
- **Identity 策略** (单例 vmId vs 基于实例的 vmId)
- **Form 采纳** (ViewModel 是否采纳 Fields 协议)

### 文件生成

**Server-Hosted 顶级：**
1. ViewModel struct (带 `RequestableViewModel`)
2. Request 类型
3. YAML 本地化
4. Factory 实现

**Client-Hosted 顶级：**
1. ViewModel struct (带 `clientHostedFactory` 选项)
2. YAML 本地化

**子级 (任一模式)：**
1. ViewModel struct
2. YAML 本地化 (如需要)

### 上下文来源

技能引用来自以下信息：
- **先前对话**：视图需求、与用户讨论的数据源
- **规范文件**：如果 Claude 已将 UI 规范或功能文档读入上下文
- **Fields 协议**：来自代码库或之前的 fosmvvm-fields-generator 调用

## 关键模式

### @ViewModel 宏

始终使用 `@ViewModel` 宏——它生成本地化绑定所需的 `propertyNames()` 方法。

**Server-Hosted** (基本宏)：
```swift
@ViewModel
public struct MyViewModel: RequestableViewModel {
    public typealias Request = MyRequest
    @LocalizedString public var title
    public var vmId: ViewModelId = .init()
    public init() {}
}
```

**Client-Hosted** (带 factory 生成)：
```swift
@ViewModel(options: [.clientHostedFactory])
public struct SettingsViewModel {
    @LocalizedString public var pageTitle
    public var vmId: ViewModelId = .init()

    public init(theme: Theme, notifications: NotificationSettings) {
        // Init 参数成为 AppState 属性
    }
}

// 宏自动生成：
// - typealias Request = ClientHostedRequest
// - struct AppState { let theme: Theme; let notifications: NotificationSettings }
// - class ClientHostedRequest: ViewModelRequest { ... }
// - static func model(context:) async throws -> Self { ... }
```

### Stubbable 模式

所有 ViewModels 必须支持 `stub()` 用于测试和 SwiftUI 预览：

```swift
public extension MyViewModel {
    static func stub() -> Self {
        .init(/* 默认值 */)
    }
}
```

### Identity: vmId

每个 ViewModel 都需要一个 `vmId` 用于 SwiftUI 的 identity 系统：

**单例** (每页一个): `vmId = .init(type: Self.self)`
**实例** (每页多个): `vmId = .init(id: id)` 其中 `id: ModelIdType`

### 本地化

静态 UI 文本使用 `@LocalizedString`：

```swift
@LocalizedString public var pageTitle
```

对应的 YAML：
```yaml
en:
  MyViewModel:
    pageTitle: "Welcome"
```

### 日期和数字

永远不要发送预格式化字符串。使用可本地化类型：

```swift
public let createdAt: LocalizableDate    // 不是 String
public let itemCount: LocalizableInt     // 不是 String
```

客户端根据用户的区域和时区格式化这些。

### 枚举本地化模式

对于动态枚举值 (状态、state、类别)，使用**存储的 `LocalizableString`**——不是 `@LocalizedString`。

`@LocalizedString` 始终查找相同的 key (属性名)。存储的 `LocalizableString` 从枚举 case 携带动态 key。

```swift
// 枚举提供 localizableString
public enum SessionState: String, CaseIterable, Codable, Sendable {
    case pending, running, completed, failed

    public var localizableString: LocalizableString {
        .localized(for: Self.self, propertyName: rawValue)
    }
}

// ViewModel 存储它 (不是 @LocalizedString)
@ViewModel
public struct SessionCardViewModel {
    public let state: SessionState                // 用于数据属性的原始枚举
    public let stateDisplay: LocalizableString   // 本地化显示文本

    public init(session: Session) {
        self.state = session.state
        self.stateDisplay = session.state.localizableString
    }
}
```

```yaml
# YAML key 匹配枚举类型和 case 名
en:
  SessionState:
    pending: "Pending"
    running: "Running"
    completed: "Completed"
    failed: "Failed"
```

**约束：** `LocalizableString` 仅在以 `localizingEncoder()` 编码的 ViewModels 中工作。不要在 Fluent JSONB 字段或其他持久化类型中使用。

### 子 ViewModels

顶级 ViewModels 包含它们的子项：

```swift
@ViewModel
public struct BoardViewModel: RequestableViewModel {
    public let columns: [ColumnViewModel]
    public let cards: [CardViewModel]
}
```

Factory 在构建父项时构建所有子项。

#### 嵌套子类型模式

当子类型**仅被一个父项使用**，并且代表摘要或引用 (不是完整的 ViewModel) 时，将其嵌套在父项内部：

```swift
@ViewModel
public struct GovernancePrincipleCardViewModel: Codable, Sendable, Identifiable {
    // 属性在前
    public let versionHistory: [GovernancePrincipleVersionSummary]?
    public let referencingDecisions: [GovernanceDecisionReference]?

    // MARK: - 嵌套类型

    /// 用于在版本历史中显示的原则版本摘要。
    public struct GovernancePrincipleVersionSummary: Codable, Sendable, Identifiable, Stubbable {
        public let id: ModelIdType
        public let version: Int
        public let createdAt: Date

        public init(id: ModelIdType, version: Int, createdAt: Date) {
            self.id = id
            self.version = version
            self.createdAt = createdAt
        }
    }

    /// 引用此原则的决策引用。
    public struct GovernanceDecisionReference: Codable, Sendable, Identifiable, Stubbable {
        public let id: ModelIdType
        public let title: String
        public let decisionNumber: String
        public let createdAt: Date

        public init(id: ModelIdType, title: String, decisionNumber: String, createdAt: Date) {
            self.id = id
            self.title = title
            self.decisionNumber = decisionNumber
            self.createdAt = createdAt
        }
    }

    // vmId 和父 init 在后
    public let vmId: ViewModelId
    // ...
}
```

**参考：** `Sources/KairosModels/Governance/GovernancePrincipleCardViewModel.swift`

**放置规则：**
1. 嵌套类型放在引用它们的属性**之后**
2. 在父级的 `vmId` 和 init 之前
3. 使用 `// MARK: - Nested Types` 分段标记
4. 每个嵌套类型有自己的文档注释

**嵌套类型的遵循：**
- `Codable` - 用于 ViewModel 编码
- `Sendable` - 用于 Swift 6 并发
- `Identifiable` - 如果在数组中使用，用于 SwiftUI ForEach
- `Stubbable` - 用于测试/预览

**两级 Stubbable 模式：**

嵌套类型在扩展中使用完全限定名：

```swift
public extension GovernancePrincipleCardViewModel.GovernancePrincipleVersionSummary {
    // 第 1 级：零参数便捷方法 (始终委托给第 2 级)
    static func stub() -> Self {
        .stub(id: .init())
    }

    // 第 2 级：带默认值的全参数
    static func stub(
        id: ModelIdType = .init(),
        version: Int = 1,
        createdAt: Date = .now
    ) -> Self {
        .init(id: id, version: version, createdAt: createdAt)
    }
}

public extension GovernancePrincipleCardViewModel.GovernanceDecisionReference {
    static func stub() -> Self {
        .stub(id: .init())
    }

    static func stub(
        id: ModelIdType = .init(),
        title: String = "A Title",
        decisionNumber: String = "DEC-12345",
        createdAt: Date = .now
    ) -> Self {
        .init(id: id, title: title, decisionNumber: decisionNumber, createdAt: createdAt)
    }
}
```

**为什么两级：**
- 测试通常只需要 `[.stub()]` 而不关心值
- 其他测试需要特定值：`.stub(name: "Specific Name")`
- 零参数**始终**调用参数化版本 (单一真相来源)

**何时嵌套 vs 保持顶级：**

| 嵌套在父项内部 | 保持顶级 |
|-------------------|----------------|
| 子项**仅**被此父项使用 | 子项被多个父项共享 |
| 子项代表子集/摘要 | 子项是完整的 ViewModel |
| 子项没有 @ViewModel 宏 | 子项有 @ViewModel 宏 |
| 子项不是 RequestableViewModel | 子项是 RequestableViewModel |
| 示例：VersionSummary, Reference | 示例：CardViewModel, ListViewModel |

**示例：**

带嵌套摘要的卡片：
```swift
@ViewModel
public struct TaskCardViewModel {
    public let assignees: [AssigneeSummary]?

    public struct AssigneeSummary: Codable, Sendable, Identifiable, Stubbable {
        public let id: ModelIdType
        public let name: String
        public let avatarUrl: String?
        // ...
    }
}
```

带嵌套引用的列表：
```swift
@ViewModel
public struct ProjectListViewModel {
    public let relatedProjects: [ProjectReference]?

    public struct ProjectReference: Codable, Sendable, Identifiable, Stubbable {
        public let id: ModelIdType
        public let title: String
        public let status: String
        // ...
    }
}
```

### Codable 和计算属性

Swift 的合成 `Codable` 仅编码**存储属性**。由于 ViewModels 被序列化 (用于 JSON 传输、Leaf 渲染等)，计算属性将不可用。

```swift
// 计算属性 - 不编码，序列化后不可见
public var hasCards: Bool { !cards.isEmpty }

// 存储属性 - 编码，序列化后可用
public let hasCards: Bool
```

**何时预计算：**

对于 Leaf 模板，你通常可以直接使用 Leaf 的内置函数：
- `#if(count(cards) > 0)` - 不需要 `hasCards` 属性
- `#count(cards)` - 不需要 `cardCount` 属性

仅在以下情况下预计算：
- 需要直接数组下标 (`firstCard` - Leaf 中未记录数组索引)
- 在 Swift 中比在模板中更清晰的复杂逻辑
- 性能敏感的重复计算

Leaf 模板模式请参阅 [fosmvvm-leaf-view-generator](../fosmvvm-leaf-view-generator/SKILL.md)。

## 文件模板

完整的文件模板请参阅 [reference.md](reference.md)。

## 命名约定

| 概念 | 约定 | 示例 |
|---------|------------|---------|
| ViewModel struct | `{Name}ViewModel` | `DashboardViewModel` |
| Request class | `{Name}Request` | `DashboardRequest` |
| Factory 扩展 | `{Name}ViewModel+Factory.swift` | `DashboardViewModel+Factory.swift` |
| YAML 文件 | `{Name}ViewModel.yml` | `DashboardViewModel.yml` |

## 另请参阅

- [Architecture Patterns](../shared/architecture-patterns.md) - 心智模型 (错误是数据、类型安全等)
- [FOSMVVMArchitecture.md](../../docs/FOSMVVMArchitecture.md) - 完整 FOSMVVM 架构
- [fosmvvm-fields-generator](../fosmvvm-fields-generator/SKILL.md) - 用于表单验证
- [fosmvvm-fluent-datamodel-generator](../fosmvvm-fluent-datamodel-generator/SKILL.md) - 用于 Fluent 持久层
- [fosmvvm-leaf-view-generator](../fosmvvm-leaf-view-generator/SKILL.md) - 用于渲染 ViewModels 的 Leaf 模板
- [reference.md](reference.md) - 完整文件模板

## 版本历史

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-12-24 | 初始技能 |
| 2.0 | 2024-12-26 | 从架构完整重写；从 Kairos 特定泛化 |
| 2.1 | 2024-12-26 | 添加 Client-Hosted 模式支持；每个 ViewModel 的托管决策 |
| 2.2 | 2024-12-26 | 添加塑形职责、@LocalizedSubs/@LocalizedCompoundString、反模式 |
| 2.3 | 2025-12-27 | 添加 Display vs Form ViewModels 章节；澄清 Fields 采纳 |
| 2.4 | 2026-01-08 | 添加 Codable/计算属性章节。澄清何时预计算 vs 使用 Leaf 内置功能。 |
| 2.5 | 2026-01-19 | 添加枚举本地化模式章节。澄清 @LocalizedString 仅用于静态文本；存储的 LocalizableString 用于动态枚举值。 |
| 2.6 | 2026-01-24 | 更新为上下文感知方法（移除文件解析/问答）。技能引用对话上下文而不是提问或接受文件路径。 |
| 2.7 | 2026-01-25 | 添加嵌套子类型模式章节，包含两级 Stubbable 模式、放置规则、遵循规范，以及嵌套 vs 保持顶级的决策标准。 |