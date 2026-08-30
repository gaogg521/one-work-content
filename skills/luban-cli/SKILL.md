---
name: luban-cli
description: Luban CLI的开发和MLOps管理。在构建或使用Luban CLI管理实验环境、训练任务和在线服务时使用此技能。
tags:
- AI
- CLI
- 效率
---

# Luban CLI Skill

此技能为开发和使用 **Luban CLI** 提供了一个结构化框架，这是一个用于MLOps管理的专用工具。

## 核心功能

Luban CLI专注于三个主要的MLOps支柱：
1. **实验环境 (`env`)**: 管理工作区。
2. **训练任务 (`job`)**: 编排模型训练工作负载。
3. **在线服务 (`svc`)**: 部署和扩展推理服务。

## 开发工作流

在开发或扩展Luban CLI时，请遵循以下步骤：

1. **初始化项目**: 使用 `templates/cli_boilerplate.py` 中的样板作为CLI结构的起点。
2. **定义命令**: 参考 `references/mlops_guide.md` 了解标准命令模式和每个实体所需的属性。
3. **实现CRUD**: 确保每个实体 (`env`, `job`, `svc`) 支持完整的生命周期：
   - **创建**: 配置新资源。
   - **读取**: 列出和描述现有资源。
   - **更新**: 修改配置或扩展。
   - **删除**: 清理资源。

## 使用模式

### 管理环境
```bash
luban env list
luban env create --name research-v1 --image pytorch:2.0
```

### 管理训练任务
```bash
luban job create --script train.py --gpu 1
luban job status --id job_001
```

### 管理在线服务
```bash
luban svc create --model-path ./models/v1 --replicas 3
luban svc scale --id my-service --replicas 5
```

## 资源
- `templates/cli_boilerplate.py`: 使用 `argparse` 的基于Python的CLI结构。
- `references/mlops_guide.md`: MLOps实体和操作的详细规范。
