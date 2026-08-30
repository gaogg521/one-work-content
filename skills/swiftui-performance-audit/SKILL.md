---
name: swiftui-performance-audit
description: 审计和优化 SwiftUI 运行时性能，诊断渲染缓慢、滚动卡顿、高 CPU/内存使用、过度视图更新及布局抖动。代码审查不足时，指导用户使用 Instruments 分析。
tags:
- 安全
- 性能优化
---

# SwiftUI 性能审计

_来源：从 @Dimillian 的 `Dimillian/Skills` 复制 (2025-12-31)。_

## 概述

端到端审计 SwiftUI 视图性能，从仪器测量和基线设定到根因分析和具体修复步骤。

## 工作流决策树

- 如果用户提供代码，从 "Code-First Review" 开始。
- 如果用户仅描述症状，请求最少的代码/上下文，然后做 "Code-First Review"。
- 如果代码审查无结论，转到 "Guide the User to Profile" 并请求 trace 或截图。

## 1. Code-First Review

收集：
- 目标视图/功能代码。
- 数据流：state、environment、observable models。
- 症状和复现步骤。

重点关注：
- 来自广泛 state 变化的视图无效化风暴。
- 列表中的不稳定 identity (`id` 抖动、每次渲染的 `UUID()`)。
- `body` 中的重负载工作 (格式化、排序、图像解码)。
- 布局抖动 (深层 stacks、`GeometryReader`、preference chains)。
- 没有降采样或调整大小的大图像。
- 过度动画层级 (大树上的隐式动画)。

提供：
- 带有代码引用的可能根因。
- 建议的修复和重构。
- 如需要，最小复现或仪器测量建议。

## 2. 引导用户进行分析

解释如何用 Instruments 收集数据：
- 在 Instruments 中使用 SwiftUI 模板 (Release build)。
- 复现确切的交互 (滚动、导航、动画)。
- 捕获 SwiftUI timeline 和 Time Profiler。
- 导出或截图相关 lanes 和 call tree。

请求：
- Trace 导出或 SwiftUI lanes + Time Profiler call tree 的截图。
- 设备/OS/build 配置。

## 3. 分析和诊断

优先考虑可能的 SwiftUI 罪魁祸首：
- 来自广泛 state 变化的视图无效化风暴。
- 列表中的不稳定 identity (`id` 抖动、每次渲染的 `UUID()`)。
- `body` 中的重负载工作 (格式化、排序、图像解码)。
- 布局抖动 (深层 stacks、`GeometryReader`、preference chains)。
- 没有降采样或调整大小的大图像。
- 过度动画层级 (大树上的隐式动画)。

用来自 traces/logs 的证据总结发现。

## 4. 修复

应用针对性修复：
- 缩小 state 范围 (`@State`/`@Observable` 更接近叶子视图)。
- 为 `ForEach` 和列表稳定 identity。
- 将重负载工作移出 `body` (预计算、缓存、`@State`)。
- 对昂贵的子树使用 `equatable()` 或 value wrappers。
- 在渲染前对图像进行降采样。
- 减少布局复杂性或在可能时使用固定大小。

## 常见代码异味 (及修复)

在代码审查期间寻找这些模式。

### `body` 中的昂贵 formatter

```swift
var body: some View {
    let number = NumberFormatter() // 慢分配
    let measure = MeasurementFormatter() // 慢分配
    Text(measure.string(from: .init(value: meters, unit: .meters)))
}
```

优先在 model 或专用 helper 中缓存 formatter：

```swift
final class DistanceFormatter {
    static let shared = DistanceFormatter()
    let number = NumberFormatter()
    let measure = MeasurementFormatter()
}
```

### 做重负载工作的计算属性

```swift
var filtered: [Item] {
    items.filter { $0.isEnabled } // 每次 body 评估都运行
}
```

优先在变化时预计算或缓存：

```swift
@State private var filtered: [Item] = []
// 当输入变化时更新 filtered
```

### 在 `body` 或 `ForEach` 中排序/筛选

```swift
List {
    ForEach(items.sorted(by: sortRule)) { item in
        Row(item)
    }
}
```

优先在视图更新前排序一次：

```swift
let sortedItems = items.sorted(by: sortRule)
```

### `ForEach` 中的内联筛选

```swift
ForEach(items.filter { $0.isEnabled }) { item in
    Row(item)
}
```

优先使用具有稳定 identity 的预筛选集合。

### 不稳定 identity

```swift
ForEach(items, id: \.self) { item in
    Row(item)
}
```

对非稳定值避免 `id: \.self`；使用稳定的 ID。

### 在主线程上的图像解码

```swift
Image(uiImage: UIImage(data: data)!)
```

优先在主线程之外解码/降采样并存储结果。

### observable models 中的广泛依赖

```swift
@Observable class Model {
    var items: [Item] = []
}

var body: some View {
    Row(isFavorite: model.items.contains(item))
}
```

优先使用细粒度 view models 或每-item state 以减少更新扩散。

## 5. 验证

请求用户重新运行相同的捕获并与基线指标对比。
如果提供，总结差异 (CPU、帧丢失、内存峰值)。

## 输出

提供：
- 简短指标表 (如有，前后对比)。
- 热门问题 (按影响排序)。
- 建议修复及估计工作量。

## 参考

在 `references/` 下添加 Apple 文档和 WWDC 资源，由用户提供。
- Optimizing SwiftUI performance with Instruments: `references/optimizing-swiftui-performance-instruments.md`
- Understanding and improving SwiftUI performance: `references/understanding-improving-swiftui-performance.md`
- Understanding hangs in your app: `references/understanding-hangs-in-your-app.md`
- Demystify SwiftUI performance (WWDC23): `references/demystify-swiftui-performance-wwdc23.md`
