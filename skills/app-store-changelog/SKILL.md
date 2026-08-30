---
name: app-store-changelog
description: 收集并汇总自上一个 git 标签（或指定引用）以来所有影响用户的变更，生成面向用户的 App Store 发布说明。支持基于 git 历史或标签生成全面的发布变更日志与“新功能”文本。触发词：发布说明(release notes)、变更日志(changelog)、App Store 发布(App Store release)、git 标签(git tag)。
tags:
- Git
- 发布工程
---

# App Store 变更日志

## 概述
从自上一个标签以来的 git 历史生成全面的、面向用户的变更日志，然后将提交转换为清晰的 App Store 发布说明。

## 工作流程

### 1) 收集变更
- 从仓库根目录运行 `scripts/collect_release_changes.sh` 以收集提交和受影响的文件。
- 如果需要，传递一个特定的标签或引用：`scripts/collect_release_changes.sh v1.2.3 HEAD`。
- 如果没有标签存在，脚本会回退到完整历史。

### 2) 筛选用户影响
- 扫描提交和文件以识别用户可见的变更。
- 按主题分组变更（新增、改进、修复）并去重重叠项。
- 丢弃仅内部的工作（构建脚本、重构、依赖项升级、CI）。

### 3) 起草 App Store 说明
- 为每个面向用户的变更编写简短、以收益为重点的要点。
- 使用清晰的动词和简单的语言；避免内部术语。
- 优先 5 到 10 个要点，除非用户要求不同的长度。

### 4) 验证
- 确保每个要点都映射回范围内的真实变更。
- 检查重复和过于技术化的措辞。
- 如果任何变更不明确或可能仅内部相关，请求澄清。

## 输出格式
- 标题 (可选)："新功能" 或产品名称 + 版本。
- 仅要点列表；每个要点一句话。
- 如果用户提供了商店限制，则遵守该限制。

## 资源
- `scripts/collect_release_changes.sh`：收集自上一个标签以来的提交和受影响的文件。
- `references/release-notes-guidelines.md`：App Store 说明的语言、筛选和 QA 规则。
