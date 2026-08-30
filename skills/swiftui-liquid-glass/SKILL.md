---
name: swiftui-liquid-glass
description: 使用 iOS 26+ Liquid Glass API 实现、审查或改进 SwiftUI 功能。当被要求在新 SwiftUI UI 中采用 Liquid Glass、将现有功能重构为 Liquid Glass，或审查 Liquid Glass 使用的正确性、性能和设计一致性时使用。
tags:
- API
---

# SwiftUI Liquid Glass

_来源：从 @Dimillian 的 `Dimillian/Skills` (2025-12-31) 复制。_

## 概述

使用此技能构建或审查与 iOS 26+ Liquid Glass API 完全对齐的 SwiftUI 功能。优先使用原生 API（`glassEffect`、`GlassEffectContainer`、glass button styles）和 Apple 设计指南。保持使用一致、在需要时具有交互性，并注意性能。

## 工作流决策树

选择匹配请求的路径：

### 1) 审查现有功能
- 检查应在何处使用 Liquid Glass，以及何处不应使用。
- 验证修饰符顺序、形状使用和容器放置是否正确。
- 检查 iOS 26+ 可用性处理和合理的降级方案。

### 2) 使用 Liquid Glass 改进功能
- 确定玻璃处理的目标组件（surface、chip、button、card）。
- 重构以在出现多个玻璃元素的地方使用 `GlassEffectContainer`。
- 仅为可点击或可聚焦的元素引入交互式玻璃。

### 3) 使用 Liquid Glass 实现新功能
- 首先设计玻璃 surface 和交互（形状、突出程度、分组）。
- 在布局/外观修饰符之后添加玻璃修饰符。
- 仅在视图层次结构随动画更改时添加变形过渡。

## 核心指南

- 优先使用原生 Liquid Glass API 而非自定义模糊。
- 当多个玻璃元素共存时使用 `GlassEffectContainer`。
- 在布局和视觉修饰符之后应用 `.glassEffect(...)`。
- 对响应触摸/指针的元素使用 `.interactive()`。
- 在相关元素之间保持形状一致，以实现统一外观。
- 使用 `#available(iOS 26, *)` 进行门控，并提供非玻璃的降级方案。

## 审查清单

- **可用性**：`#available(iOS 26, *)` 存在并附带降级 UI。
- **组合**：多个玻璃视图包裹在 `GlassEffectContainer` 中。
- **修饰符顺序**：`glassEffect` 在布局/外观修饰符之后应用。
- **交互性**：`interactive()` 仅存在于用户交互的地方。
- **过渡**：`glassEffectID` 与 `@Namespace` 一起用于变形。
- **一致性**：形状、色调和间距在整个功能中对齐。

## 实现清单

- 定义目标元素和期望的玻璃突出程度。
- 将分组的玻璃元素包裹在 `GlassEffectContainer` 中并调整间距。
- 按需使用 `.glassEffect(.regular.tint(...).interactive(), in: .rect(cornerRadius: ...))`。
- 对操作使用 `.buttonStyle(.glass)` / `.buttonStyle(.glassProminent)`。
- 在层次结构更改时，使用 `glassEffectID` 添加变形过渡。
- 为早期 iOS 版本提供降级材质和视觉效果。

## 快速片段

直接应用这些模式，并调整形状/色调/间距。

```swift
if #available(iOS 26, *) {
    Text("Hello")
        .padding()
        .glassEffect(.regular.interactive(), in: .rect(cornerRadius: 16))
} else {
    Text("Hello")
        .padding()
        .background(.ultraThinMaterial, in: RoundedRectangle(cornerRadius: 16))
}
```

```swift
GlassEffectContainer(spacing: 24) {
    HStack(spacing: 24) {
        Image(systemName: "scribble.variable")
            .frame(width: 72, height: 72)
            .font(.system(size: 32))
            .glassEffect()
        Image(systemName: "eraser.fill")
            .frame(width: 72, height: 72)
            .font(.system(size: 32))
            .glassEffect()
    }
}
```

```swift
Button("Confirm") { }
    .buttonStyle(.glassProminent)
```

## 资源

- 参考指南：`references/liquid-glass.md`
- 优先参考 Apple 文档获取最新的 API 详情。
