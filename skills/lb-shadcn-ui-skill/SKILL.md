---
name: lb-shadcn-ui-skill
description: 完整的 shadcn/ui 设计组件文档，基于 Radix UI 和 Tailwind CSS 构建。支持复制粘贴到项目中，涵盖安装、50+ 组件、主题、表单、图表和 Next.js、Vite、Remix 等框架集成。
---

# shadcn/ui 文档

从官方 shadcn/ui 仓库提取的完整 shadcn/ui 文档。

## 内容

此技能包含 **201 个 MDX 文件**（1.4MB），涵盖：

### 入门
- **安装** - Next.js, Vite, Remix, Astro, Laravel, Gatsby, React Router, Tanstack Router 的设置
- **CLI** - shadcn/ui CLI 命令和用法
- **Components JSON** - 配置和自定义
- **主题** - CSS 变量、暗模式、主题自定义
- **排版** - 字体设置和排版工具

### 组件 (50+)
- **Accordion** - 可折叠内容区域
- **Alert** - 上下文反馈消息
- **Avatar** - 用户头像图片
- **Badge** - 状态指示器
- **Button** - 带变体的交互式按钮
- **Calendar** - 日期选择器和日历视图
- **Card** - 内容容器
- **Checkbox** - 选择控件
- **Combobox** - 可搜索选择
- **Command** - 命令面板
- **Context Menu** - 右键菜单
- **Data Table** - 可排序、可过滤的表格
- **Date Picker** - 日期选择
- **Dialog** - 模态对话框
- **Drawer** - 滑出面板
- **Dropdown Menu** - 操作菜单
- **Form** - 带验证的表单组件
- **Hover Card** - 悬停内容卡片
- **Input** - 文本输入框
- **Label** - 表单标签
- **Menubar** - 应用菜单栏
- **Navigation Menu** - 网站导航
- **Pagination** - 页面导航
- **Popover** - 浮动内容
- **Progress** - 进度指示器
- **Radio Group** - 单选按钮组
- **Resizable** - 可调整大小的面板
- **Scroll Area** - 自定义滚动条
- **Select** - 下拉选择
- **Separator** - 视觉分隔线
- **Sheet** - 滑出面板
- **Skeleton** - 加载占位符
- **Slider** - 范围输入
- **Switch** - 切换开关
- **Table** - 数据表格
- **Tabs** - 标签页界面
- **Textarea** - 多行文本输入
- **Toast** - Toast 通知
- **Toggle** - 切换按钮
- **Tooltip** - 上下文提示
- **以及更多...**

### 高级功能
- **Charts** - Recharts 集成（Area, Bar, Line, Pie, Radar, Radial）
- **Forms** - React Hook Form, Tanstack Form, Zod 集成
- **Data Tables** - 高级表格模式
- **Dark Mode** - 主题切换
- **Figma** - 设计系统集成
- **Icons** - 图标库（Lucide, Radix Icons）

### 框架集成
- **Next.js** - App Router, Pages Router
- **Vite** - React + Vite 设置
- **Remix** - Remix 集成
- **Astro** - Astro 集成
- **Laravel** - Inertia.js + Laravel
- **Gatsby** - Gatsby 集成
- **React Router** - React Router v7
- **Tanstack Router** - Tanstack Router 集成

### 注册表与分发
- **Registry** - 组件注册表系统
- **Custom Registry** - 构建你自己的组件注册表
- **Namespace** - 自定义命名空间
- **Authentication** - 注册表认证
- **MCP** - Model Context Protocol 集成

### 开发者资源
- **Changelog** - 版本历史和更新
- **About** - 项目理念和原则
- **FAQ** - 常见问题
- **CLI Reference** - 完整的 CLI 文档
- **RTL Support** - 从右到左语言支持

## 用法

此技能为以下方面提供全面的文档：

1. **组件安装** - 将组件添加到你的项目
2. **自定义** - 主题、样式和变体
3. **框架设置** - 与 Next.js, Vite, Remix 等集成
4. **表单与验证** - React Hook Form + Zod 模式
5. **图表与数据** - Recharts 集成
6. **设计系统** - 构建自定义设计系统
7. **无障碍** - 符合 WCAG 的组件

## 理念

shadcn/ui **不是一个组件库**。它是一个可复用组件的集合，你可以复制并粘贴到你的应用中。

**主要优势：**
- **拥有代码** - 组件被复制到你的项目
- **可定制** - 完全控制样式和行为
- **可访问** - 基于 Radix UI 原语
- **可主题化** - 使用 CSS 变量轻松主题化
- **框架无关** - 适用于任何 React 框架

## 文件结构

```
docs/
├── installation/        # 框架特定的设置指南
├── components/          # 组件文档 (50+)
├── charts/              # 图表组件 (Recharts)
├── forms/               # 表单集成指南
├── cli.mdx              # CLI 参考
├── components-json.mdx  # 配置参考
├── theming.mdx          # 主题指南
├── dark-mode.mdx        # 暗模式实现
├── typography.mdx       # 排版设置
├── changelog.mdx        # 版本历史
├── about.mdx            # 项目理念
├── figma.mdx            # Figma 集成
└── registry/            # 注册表文档
```

## 文档来源

从以下来源提取的官方 shadcn/ui 文档：
- 仓库: https://github.com/shadcn-ui/ui
- 网站: https://ui.shadcn.com
- 最后更新: 2026-02-07

## 许可证

文档内容 © shadcn。根据合理使用原则用于教育目的。
Skill 包 © OpenClaw Community, MIT License。
