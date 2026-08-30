---
name: expanso-json-validate
description: 验证 JSON 语法和结构。支持 CLI 流水线(pipeline)和 MCP 流水线(pipeline)，可部署到 Expanso Cloud。
tags:
- CI/CD
- 云服务
---

# json-validate

"验证 JSON 语法和结构"

## 要求

- Expanso Edge 已安装 (`expanso-edge` 二进制文件在 PATH 中)
- 通过以下方式安装: `clawhub install expanso-edge`

## 用法

### CLI 流水线
```bash
# 独立运行
echo '<input>' | expanso-edge run pipeline-cli.yaml
```

### MCP 流水线
```bash
# 作为 MCP 服务器启动
expanso-edge run pipeline-mcp.yaml
```

### 部署到 Expanso Cloud
```bash
expanso-cli job deploy https://skills.expanso.io/json-validate/pipeline-cli.yaml
```

## 文件

| 文件 | 用途 |
|------|---------|
| `skill.yaml` | 技能元数据 (输入, 输出, 凭证) |
| `pipeline-cli.yaml` | 独立 CLI 流水线 |
| `pipeline-mcp.yaml` | MCP 服务器流水线 |
