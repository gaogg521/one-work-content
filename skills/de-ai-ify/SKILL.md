---
name: de-ai-ify
description: 移除 AI 生成的行话，恢复文本的人类声音
version: 1.0.0
author: theflohart
tags:
- 文档
- editing
- voice
- ai-detection
---

# De-AI-ify 文本

移除 AI 生成的模式，恢复自然的人类声音到你的写作中。

## 用法

```
/de-ai-ify <file_path>
```

## 移除的内容

### 1. 过度使用的过渡词

- "Moreover," "Furthermore," "Additionally," "Nevertheless"
- 过多的 "However" 使用
- "While X, Y" 开头

### 2. AI 陈词滥调

- "In today's fast-paced world"
- "Let's dive deep"
- "Unlock your potential"
- "Harness the power of"

### 3. 模糊限定语

- "It's important to note"
- "It's worth mentioning"
- 模糊量词："various," "numerous," "myriad"

### 4. 企业流行语

- "utilize" → "use"
- "facilitate" → "help"
- "optimize" → "improve"
- "leverage" → "use"

### 5. 机械式模式

- 修辞问题后紧跟立即回答
- 强迫的平行结构
- 总是使用恰好三个例子
- 强调声明

## 添加的内容

### 自然声音

- 变化的句子长度
- 对话式语调
- 直接陈述
- 具体例子

### 人类节奏

- 自然的过渡
- 自信的断言
- 个人视角
- 真实的措辞

## 流程

1. **读取原始文件**
2. **创建带 "-HUMAN" 后缀的副本**
3. **应用 de-AI-ification**
4. **提供变更日志**

## 输出

你将得到：

- 一个带有自然人类声音的新文件
- 显示修复内容的变更日志
- 需要具体例子的位置列表

## 示例转换

**之前 (AI):** "In today's rapidly evolving digital landscape, it's crucial to
understand that leveraging AI effectively isn't just about utilizing
cutting-edge technology—it's about harnessing its transformative potential to
unlock unprecedented opportunities."

**之后 (Human):** "AI works best when you use it for specific tasks. Focus on
what it does well: writing code, analyzing data, and answering questions."
