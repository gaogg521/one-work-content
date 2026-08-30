---
name: swiftui-ui-patterns
description: 构建 SwiftUI 视图和组件的最佳实践和示例驱动指南。在创建或重构 SwiftUI UI、使用 TabView 设计标签架构、组合屏幕或需要组件特定模式和示例时使用。
tags:
- 模式
---

# SwiftUI UI 模式

## 快速开始

根据你的目标选择路径：

### 现有项目

- 确定功能或屏幕以及主要交互模型（列表、详情、编辑器、设置、标签页）。
- 在仓库中使用 `rg "TabView\\("` 或类似命令查找附近的示例，然后阅读最接近的 SwiftUI 视图。
- 应用本地约定：优先使用 SwiftUI 原生状态，尽可能保持状态本地化，并对共享依赖项使用环境注入。
- 从 `references/components-index.md` 中选择相关的组件参考并遵循其指南。
- 使用小型、专注的子视图和 SwiftUI 原生数据流构建视图。

### 新项目脚手架

- 从 `references/app-scaffolding-wiring.md` 开始，连接 TabView + NavigationStack + sheets。
- 基于提供的骨架添加最小的 `AppTab` 和 `RouterPath`。
- 根据你首先需要的 UI 选择下一个组件参考（TabView、NavigationStack、Sheets）。
- 随着新屏幕的添加扩展路由和 sheet 枚举。

## 要遵循的一般规则

- 使用现代 SwiftUI 状态（`@State`、`@Binding`、`@Observable`、`@Environment`）并避免不必要的视图模型。
- 优先组合；保持视图小而专注。
- 使用 async/await 配合 `.task` 和显式的加载/错误状态。
- 仅在编辑旧文件时保持现有的旧模式。
- 遵循项目的格式化程序和样式指南。
- **Sheets**：当状态表示选中的模型时，优先使用 `.sheet(item:)` 而非 `.sheet(isPresented:)`。避免在 sheet 主体内部使用 `if let`。Sheets 应该拥有自己的操作并在内部调用 `dismiss()`，而不是转发 `onCancel`/`onConfirm` 闭包。

## 新 SwiftUI 视图的工作流

1. 定义视图的状态及其所有权位置。
2. 识别要通过 `@Environment` 注入的依赖项。
3. 草拟视图层次结构并将重复部分提取到子视图中。
4. 使用 `.task` 实现异步加载，并在需要时使用显式状态枚举。
5. 当 UI 是交互式时添加无障碍标签或标识符。
6. 通过构建验证并在需要时更新使用调用点。

## 组件参考

使用 `references/components-index.md` 作为入口点。每个组件参考应包括：
- 意图和最佳适用场景。
- 带有本地约定的最小使用模式。
- 陷阱和性能注意事项。
- 当前仓库中现有示例的路径。

## Sheet 模式

### 项目驱动的 sheet（首选）

```swift
@State private var selectedItem: Item?

.sheet(item: $selectedItem) { item in
    EditItemSheet(item: item)
}
```

### Sheet 拥有自己的操作

```swift
struct EditItemSheet: View {
    @Environment(\.dismiss) private var dismiss
    @Environment(Store.self) private var store

    let item: Item
    @State private var isSaving = false

    var body: some View {
        VStack {
            Button(isSaving ? "Saving…" : "Save") {
                Task { await save() }
            }
        }
    }

    private func save() async {
        isSaving = true
        await store.save(item)
        dismiss()
    }
}
```

## 添加新的组件参考

- 创建 `references/<component>.md`。
- 保持简短且可操作；链接到当前仓库中的具体文件。
- 使用新条目更新 `references/components-index.md`。
