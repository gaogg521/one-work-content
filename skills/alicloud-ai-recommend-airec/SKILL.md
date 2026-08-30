---
name: alicloud-ai-recommend-airec
description: 通过 OpenAPI/SDK 管理阿里云 AIRec (Airec)。用于列出资源、创建或更新配置、查询状态以及排查该产品的工作流问题。
tags:
- AI
- API
- 云服务
- 阿里云
---

Category: service

# AIRec

使用阿里云 OpenAPI (RPC) 配合官方 SDK 或 OpenAPI Explorer 管理 AIRec 资源。

## 工作流

1) 确认区域、资源标识符和期望的操作。
2) 发现 API 列表和必需参数（见参考）。
3) 使用 SDK 或 OpenAPI Explorer 调用 API。
4) 使用 describe/list API 验证结果。

## AccessKey 优先级（必须遵循）

1) 环境变量：`ALICLOUD_ACCESS_KEY_ID` / `ALICLOUD_ACCESS_KEY_SECRET` / `ALICLOUD_REGION_ID`
Region 策略：`ALICLOUD_REGION_ID` 是可选默认值。如果未设置，为任务决定最合理的区域；如果不清楚，询问用户。
2) 共享配置文件：`~/.alibabacloud/credentials`

## API 发现

- 产品代码：`Airec`
- 默认 API 版本：`2020-11-26`
- 使用 OpenAPI 元数据端点列出 API 和获取 schema（见参考）。

## 高频操作模式

1) 清单/列表：优先使用 `List*` / `Describe*` API 获取当前资源。
2) 变更/配置：优先使用 `Create*` / `Update*` / `Modify*` / `Set*` API 进行变更。
3) 状态/排查：优先使用 `Get*` / `Query*` / `Describe*Status` API 进行诊断。

## 最小可执行快速开始

在调用业务 API 之前使用元数据优先发现：

```bash
python scripts/list_openapi_meta_apis.py
```

可选覆盖：

```bash
python scripts/list_openapi_meta_apis.py --product-code <ProductCode> --version <Version>
```

该脚本将 API 清单产物写入技能输出目录。

## 输出策略

如果需要保存响应或生成的产物，写入：
`output/alicloud-ai-recommend-airec/`

## 参考

- Sources: `references/sources.md`
