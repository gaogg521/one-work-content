---
name: expanso-json-to-yaml
description: 将 JSON 转换为 YAML 格式。支持 CLI Pipeline 和 MCP Pipeline 两种模式。
tags:
- CI/CD
---

# json-to-yaml

将 JSON 转换为 YAML 格式

## 要求

- 已安装 Expanso Edge（PATH 中有 `expanso-edge` binary）
- 通过以下方式安装：`clawhub install expanso-edge`

## 用法

### CLI Pipeline
```bash
# 独立运行
echo '<input>' | expanso-edge run pipeline-cli.yaml
```

### MCP Pipeline
```bash
# 作为 MCP server 启动
expanso-edge run pipeline-mcp.yaml
```

### 部署到 Expanso Cloud
```bash
expanso-cli job deploy https://skills.expanso.io/json-to-yaml/pipeline-cli.yaml
```

## 文件

| File | Purpose |
|------|---------|
| `skill.yaml` | Skill metadata（inputs、outputs、credentials） |
| `pipeline-cli.yaml` | 独立 CLI pipeline |
| `pipeline-mcp.yaml` | MCP server pipeline |
