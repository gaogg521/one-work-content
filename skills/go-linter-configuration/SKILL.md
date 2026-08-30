---
name: go-linter-configuration
description: 为 Go 项目配置和排查 golangci-lint。处理导入解析(import resolution)、类型检查(type checking)，并为本地(local)和 CI 环境优化配置。触发词：Go 代码检查(linting)、golangci-lint 配置(configuration)、CI 优化(CI optimization)
metadata: None
openclaw: None
emoji: 🔍
install:
- bins:
  - go
  id: golang
  kind: script
  label: Install Go
  script: curl -L https://golang.org/dl/go1.21.5.linux-amd64.tar.gz | tar -C /usr/local
    -xzf -
- bins:
  - golangci-lint
  id: golangci
  kind: script
  label: Install golangci-lint
  script: curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh
    | sh -s -- -b $(go env GOPATH)/bin v1.59.1
requires: None
bins:
- go
- golangci-lint
tags:
- Ansible
- CI/CD
- Go
- 性能优化
- 配置
---

# Go Linter 配置技能

为 Go 项目配置和排查 golangci-lint。此技能帮助处理导入解析问题、类型检查问题，并为本地和 CI 环境优化配置。

## 安装

安装 golangci-lint：

```bash
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

或使用官方安装脚本：

```bash
curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.59.1
```

## 基本用法

在整个项目上运行 linter：

```bash
golangci-lint run ./...
```

使用特定配置运行：

```bash
golangci-lint run --config .golangci.yml ./...
```

## 配置文件 (.golangci.yml)

### 最小配置（用于有导入问题的 CI 环境）
```yaml
run:
  timeout: 5m
  tests: false
  build-tags: []

linters:
  disable-all: true
  enable:
    - gofmt          # 仅格式检查

linters-settings:
  gofmt:
    simplify: true

issues:
  exclude-use-default: false
  max-issues-per-linter: 50
  max-same-issues: 3

output:
  format: tab
```

### 标准配置（用于本地开发）
```yaml
run:
  timeout: 5m
  tests: true
  build-tags: []

linters:
  enable:
    - gofmt
    - govet
    - errcheck
    - staticcheck
    - unused
    - gosimple
    - ineffassign

linters-settings:
  govet:
    enable:
      - shadow
  errcheck:
    check-type-assertions: true
  staticcheck:
    checks: ["all"]

issues:
  exclude-use-default: false
  max-issues-per-linter: 50
  max-same-issues: 3

output:
  format: tab
```

## 常见问题排查

### "undefined: package" 错误
问题：Linter 报告导入包的未定义引用
解决方案：使用最小配置，`disable-all: true` 并仅启用 `gofmt` 等基本 linter

### 导入解析问题
问题：CI 环境无法正确解析依赖
解决方案：
1. 确保 go.mod 和 go.sum 是最新的
2. 在 CI 中运行 linter 前使用 `go mod download`
3. 在 CI 环境中考虑使用更简单的 linter

### 类型检查失败
问题：Linter 在类型检查阶段失败
解决方案：
1. 暂时禁用需要类型检查的复杂 linter
2. 使用 `--fast` 标志进行更快、更轻量的检查
3. 验证所有导入都已正确声明

## CI/CD 优化

GitHub Actions 工作流示例：

```yaml
name: Code Quality

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Set up Go
      uses: actions/setup-go@v4
      with:
        go-version: '1.21'
        cache: true

    - name: Download dependencies
      run: go mod download

    - name: Install golangci-lint
      run: |
        curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.59.1

    - name: Lint
      run: golangci-lint run --config .golangci.yml ./...
```

## Linter 选择指南

- **gofmt**：用于格式一致性
- **govet**：用于语义错误
- **errcheck**：用于未检查的错误
- **staticcheck**：用于静态分析
- **unused**：用于死代码检测
- **gosimple**：用于简化建议
- **ineffassign**：用于无效赋值

根据项目需求和 CI 性能要求选择 linter。