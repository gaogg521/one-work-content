---
name: go
description: Go 编程语言专家 - 代码规范、调试排查、依赖管理、源码分析
allowed-tools: Read,Bash(go:*)
version: 1.0.0
author: User
license: MIT
---

# Go 编程语言

有效使用 Go 项目的指南。

## 读取依赖源文件

要查看来自依赖的源文件，或回答有关依赖的问题：

```bash
go mod download -json MODULE
```

使用返回的 Dir 路径读取源文件。

## 读取文档

使用 go doc 读取包、类型、函数等的文档：

```bash
go doc foo.Bar       # 特定符号的文档
go doc -all foo      # 包的所有文档
```

## 运行程序

使用 go run 而不是 go build 以避免留下构建产物：

```bash
go run .             # 运行当前包
go run ./cmd/foo     # 运行特定命令
```
