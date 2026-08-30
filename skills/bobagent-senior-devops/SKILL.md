---
name: bobagent-senior-devops
description: 综合 DevOps 技能，覆盖 CI/CD、基础设施自动化、容器化及云平台(AWS, GCP, Azure)。支持流水线设置、基础设施即代码、部署自动化与监控。触发词：DevOps、CI/CD、基础设施(infrastructure)、部署(deployment)、监控(monitoring)。
---

# 高级 Devops

面向高级 devops 的完整工具包，包含现代工具和最佳实践。

## 快速开始

### 主要功能

此技能通过自动化脚本提供三个核心功能：

```bash
# 脚本 1: 流水线生成器
python scripts/pipeline_generator.py [options]

# 脚本 2: Terraform 脚手架
python scripts/terraform_scaffolder.py [options]

# 脚本 3: 部署管理器
python scripts/deployment_manager.py [options]
```

## 核心功能

### 1. 流水线生成器

用于流水线生成器任务的自动化工具。

**功能:**
- 自动化脚手架
- 内置最佳实践
- 可配置模板
- 质量检查

**用法:**
```bash
python scripts/pipeline_generator.py <project-path> [options]
```

### 2. Terraform 脚手架

全面的分析和优化工具。

**功能:**
- 深度分析
- 性能指标
- 建议
- 自动修复

**用法:**
```bash
python scripts/terraform_scaffolder.py <target-path> [--verbose]
```

### 3. 部署管理器

用于专项任务的高级工具。

**功能:**
- 专家级自动化
- 自定义配置
- 集成就绪
- 生产级输出

**用法:**
```bash
python scripts/deployment_manager.py [arguments] [options]
```

## 参考文档

### CI/CD 流水线指南

`references/cicd_pipeline_guide.md` 中的综合指南：

- 详细模式和实践
- 代码示例
- 最佳实践
- 需要避免的反模式
- 真实场景

### 基础设施即代码

`references/infrastructure_as_code.md` 中的完整工作流文档：

- 分步流程
- 优化策略
- 工具集成
- 性能调优
- 故障排除指南

### 部署策略

`references/deployment_strategies.md` 中的技术参考指南：

- 技术栈详情
- 配置示例
- 集成模式
- 安全注意事项
- 可扩展性指南

## 技术栈

**语言:** TypeScript, JavaScript, Python, Go, Swift, Kotlin
**前端:** React, Next.js, React Native, Flutter
**后端:** Node.js, Express, GraphQL, REST APIs
**数据库:** PostgreSQL, Prisma, NeonDB, Supabase
**DevOps:** Docker, Kubernetes, Terraform, GitHub Actions, CircleCI
**云:** AWS, GCP, Azure

## 开发工作流

### 1. 设置和配置

```bash
# 安装依赖
npm install
# 或
pip install -r requirements.txt

# 配置环境
cp .env.example .env
```

### 2. 运行质量检查

```bash
# 使用分析器脚本
python scripts/terraform_scaffolder.py .

# 审查建议
# 应用修复
```

### 3. 实施最佳实践

遵循文档化的模式和实践：
- `references/cicd_pipeline_guide.md`
- `references/infrastructure_as_code.md`
- `references/deployment_strategies.md`

## 最佳实践总结

### 代码质量
- 遵循既定模式
- 编写全面的测试
- 记录决策
- 定期审查

### 性能
- 优化前进行测量
- 使用适当的缓存
- 优化关键路径
- 在生产环境中监控

### 安全
- 验证所有输入
- 使用参数化查询
- 实施适当的认证
- 保持依赖项更新

### 可维护性
- 编写清晰的代码
- 使用一致的命名
- 添加有用的注释
- 保持简单

## 常用命令

```bash
# 开发
npm run dev
npm run build
npm run test
npm run lint

# 分析
python scripts/terraform_scaffolder.py .
python scripts/deployment_manager.py --analyze

# 部署
docker build -t app:latest .
docker-compose up -d
kubectl apply -f k8s/
```

## 故障排除

### 常见问题

在 `references/deployment_strategies.md` 中查看全面的故障排除部分。

### 获取帮助

- 审查参考文档
- 检查脚本输出消息
- 查阅技术栈文档
- 审查错误日志

## 资源

- 模式参考: `references/cicd_pipeline_guide.md`
- 工作流指南: `references/infrastructure_as_code.md`
- 技术指南: `references/deployment_strategies.md`
- 工具脚本: `scripts/` 目录
