---
name: table-image
description: 从表格生成图片，以便在 Telegram 等消息应用中更好地阅读。在展示表格数据时使用。
metadata:
  clawdis:
    emoji: 📊
tags:
- 即时通讯
- 数据
---

# Table Image Skill

将 markdown 表格渲染为 PNG 图片，用于不支持 markdown 表格的消息平台。

## 前置条件

安装 tablesnap：

```bash
go install github.com/joargp/tablesnap/cmd/tablesnap@latest
```

或从源码构建：
```bash
git clone https://github.com/joargp/tablesnap.git
cd tablesnap
go build -o tablesnap ./cmd/tablesnap
```

## 用法

```bash
echo "| Header 1 | Header 2 |
|----------|----------|
| Data 1   | Data 2   |" | tablesnap -o /tmp/table.png
```

然后使用 `MEDIA:/tmp/table.png` 发送

## 选项

| 标志 | 默认值 | 描述 |
|------|---------|-------------|
| `-i` | stdin | 输入文件 |
| `-o` | stdout | 输出文件 |
| `--theme` | dark | 主题：`dark` 或 `light` |
| `--font-size` | 14 | 字体大小（像素） |
| `--padding` | 10 | 单元格内边距（像素） |

## Emoji 支持

**内置**（开箱即用）：✅ ❌ 🔴 🟢 🟡 ⭕ ⚠️

**完整 emoji**（一次性下载）：
```bash
tablesnap emojis install
```

不支持的 emoji 在完整集安装前会渲染为 □。

## 示例工作流

```bash
# 创建表格图片
echo "| Task | Status |
|------|--------|
| Build | ✅ |
| Deploy | 🚀 |" | tablesnap -o /tmp/table.png

# 在回复中发送
MEDIA:/tmp/table.png
```

## 注意

- 默认暗色主题（适配 Telegram/Discord 暗色模式）
- 自动调整大小以适应内容
- 输出约 10-20KB（适合消息发送）
- 跨平台（内置 Inter 字体）

## 链接

- [tablesnap 仓库](https://github.com/joargp/tablesnap)
