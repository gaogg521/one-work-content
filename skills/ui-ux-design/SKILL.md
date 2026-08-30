---
name: ui-ux-design
description: 现代 UI/UX 设计原则、模式和最佳实践，适用于 Web 和移动应用。包含设计趋势、Tailwind CSS 模式、Shadcn/ui 集成、微交互和响应式设计。
tags:
- Web
- 架构
---

# UI/UX Design

**Name:** ui-ux-design  
**Description:** 现代UI/UX设计原则、模式和最佳实践，适用于Web和移动应用。在构建用户界面、设计布局、选择调色板、实现响应式设计、确保可访问性（WCAG）或创建美观的现代应用时使用。包含2026年设计趋势、Tailwind CSS模式、Shadcn/ui集成、微交互和移动优先的响应式设计。

---

## 何时使用本 Skill

在以下场景激活本 skill：
- 构建或设计Web/移动界面
- 选择颜色、字体或布局系统
- 实现响应式设计（移动优先）
- 确保可访问性合规（WCAG 2.2）
- 设置 Shadcn/ui + Tailwind CSS 项目
- 创建微交互和动画
- 在编码前审查UI/UX决策

---

## 核心设计原则

### 1. 始终移动优先
- 从320px宽度开始（最小手机）
- Breakpoints: 576px (phone), 768px (tablet), 992px (laptop), 1200px (desktop)
- 默认单列布局，仅在空间允许时扩展

### 2. 视觉层级
使用以下方式引导用户注意力：
- **Size:** 越大 = 越重要
- **Color:** 明亮/对比色 = 注意力
- **Whitespace:** 更多空间 = 强调
- **Proximity:** 相关项目分组在一起
- **Contrast:** 深色在浅色上或浅色在深色上（文本最低4.5:1）

### 3. 留白是你的武器
- 以8px的倍数间隔元素（8, 16, 24, 32, 48, 64）
- 区块之间的呼吸空间：最少48-64px
- 卡片内部padding：24-32px

---

## 快速参考

### 配色系统
构建主色色阶（50-900）：
- **Primary:** 品牌色（CTAs, links, active states）
- **Neutrals:** Grays 50-900（text, backgrounds, borders）
- **Semantic:** Success (green), Error (red), Warning (yellow/orange)

Tools: Huevy.app, Coolors.co, Adobe Color

### 字体排版比例 (8px baseline)
```
text-xs:   12px / 16px line-height
text-sm:   14px / 20px
text-base: 16px / 24px (body default)
text-lg:   18px / 28px
text-xl:   20px / 28px
text-2xl:  24px / 32px
text-3xl:  30px / 36px (section headers)
text-4xl:  36px / 40px
text-5xl:  48px / 1 (hero titles)
```

**Font pairing:** 最多2种字体（UI用sans-serif，标题可选serif）

### 布局模式
- **CSS Grid:** 2D布局（页面结构）
- **Flexbox:** 1D布局（组件内部）
- **Auto-fit grid:** `repeat(auto-fit, minmax(280px, 1fr))`（无需media queries！）

### 微交互
- **Hover:** 缩放1.05x（按钮感觉可点击）
- **Click:** 缩放0.95x（触觉反馈）
- **Duration:** 最多0.2-0.3s（保持微妙）
- **Animate only:** `transform` 和 `opacity`（GPU加速）

### 无障碍 (WCAG 2.2)
- **Text contrast:** 最低4.5:1（普通文本），3:1（大文本）
- **UI components:** 最低3:1 contrast
- **Keyboard navigation:** Tab顺序逻辑，focus状态可见（3:1 contrast）
- **ARIA labels:** 始终为按钮、图片、交互元素提供

---

## Shadcn/ui + Tailwind 技术栈

### 初始化 (Next.js)
```bash
npx create-next-app@latest project-name --typescript --tailwind --app
cd project-name
npx shadcn@latest init
```

选择：Style (Default), Base color (Blue or custom), CSS variables (Yes)

### 添加组件
```bash
npx shadcn@latest add button
npx shadcn@latest add card
npx shadcn@latest add dialog
npx shadcn@latest add calendar
```

组件会出现在 `components/ui/` —— 你拥有这些代码，可自由定制。

### Tailwind 最佳实践
- 使用 design tokens（而非任意值）：`p-4` 而非 `p-[17px]`
- 响应式工具类：`w-full md:w-1/2 lg:w-1/3`
- 暗黑模式：`dark:bg-gray-900 dark:text-white`

---

## 构建前检查清单

在编写代码前，确认：
- [ ] 配色方案已定义（primary + neutrals + semantic colors）
- [ ] 字体比例已选定（6-8种尺寸）
- [ ] 组件库已选定（Shadcn + Tailwind）
- [ ] 移动端断点已规划（576px, 768px, 992px）
- [ ] 无障碍对比度已检查（4.5:1 文本，3:1 UI）
- [ ] 微交互清单（hover, click, success states）
- [ ] 网格布局已草拟（mobile → desktop 递进）

---

## 灵感来源

**研究这些产品：**
- Linear (linear.app) —— 最佳键盘优先UI，微妙动画
- Stripe Dashboard —— 简洁数据可视化，完美间距
- Vercel —— 极简，快速，现代渐变
- Notion —— 直观拖放，清晰层级

**工具：**
- Figma（编码前制作mockups）
- WebAIM Contrast Checker（无障碍检查）
- Coolors/Huevy（调色板）

---

## 美观UI的5条法则

1. **对比创造层级**（大与小，深与浅）
2. **留白创造平静**（永远不要害怕空白）
3. **一致性建立信任**（相同模式重复出现）
4. **反馈确认操作**（动画，成功消息）
5. **无障碍包容所有人**（对比度，键盘，屏幕阅读器）

---

## 完整参考

如需全面深入的内容（组件模式、动画示例、响应式网格技巧），请参阅本 skill 目录下的 `UI_UX_MASTER_GUIDE.md`。

---

**Last Updated:** 2026-02-05
