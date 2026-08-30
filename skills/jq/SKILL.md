---
name: jq
description: 命令行JSON处理器。提取、过滤、转换JSON。
tags:
- CLI
---

# jq

用于提取、过滤和转换JSON的命令行JSON处理器。

## 安装

**macOS / Linux (Homebrew):**
```bash
brew install jq
```

**所有平台:** 有关包、二进制文件和构建说明，请参阅 [jqlang.org/download](https://jqlang.org/download/)。

## 用法

```bash
jq '[filter]' [file.json]
cat file.json | jq '[filter]'
```

## 快速参考

```bash
.key                    # 获取键
.a.b.c                  # 嵌套访问
.[0]                    # 第一个元素
.[]                     # 迭代数组
.[] | select(.x > 5)    # 过滤
{a: .x, b: .y}          # 重塑
. + {new: "val"}        # 添加字段
del(.key)               # 删除字段
length                  # 计数
[.[] | .x] | add        # 求和
keys                    # 列出键
unique                  # 数组去重
group_by(.x)            # 分组
```

## 标志

`-r` 原始输出（无引号） · `-c` 紧凑 · `-s` 读取为数组 · `-S` 按键排序

## 示例

```bash
jq '.users[].email' data.json          # 提取电子邮件
jq -r '.name // "default"' data.json   # 带默认值
jq '.[] | select(.active)' data.json   # 过滤活跃项
jq -s 'add' *.json                     # 合并文件
jq '.' file.json                       # 美化打印
```
