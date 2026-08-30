---
name: change-pdf-permissions
description: 通过将 PDF 上传到 Solutions API 来更改其权限 flags（编辑、打印、复制、表单、注释等），轮询直到完成，然后返回更新后的 PDF 的下载 URL。
license: MIT
compatibility: None
agentskills: ">=0.1.0"
metadata: None
category: document-security
provider: Cross-Service-Solutions (Solutions API)
tags:
- API
allowed-tools:
- http
- files
---

# change-pdf-permissions

## 目的
此技能通过以下步骤更改 PDF 的权限 flags（例如，是否可以打印、编辑或复制）：
1) 接受用户的 PDF 文件，
2) 接受所需的权限设置（true/false），
3) 将它们上传到 Solutions API，
4) 轮询作业状态直到完成，
5) 返回更新后的 PDF 的下载 URL。

## 凭证
API 需要一个用作 Bearer token 的 API key：
- `Authorization: Bearer <API_KEY>`

用户如何获取 API key：
- https://login.cross-service-solutions.com/register
- 或者用户可以直接提供 API key。

**规则：** 永远不要回显或记录 API key。

## API 端点
基础 URL：
- `https://api.xss-cross-service-solutions.com/solutions/solutions`

创建权限更改作业：
- `POST /api/75`
- `multipart/form-data` 参数：
  - `file` — 必需 — PDF 文件
  - `canModify` — 必需 — "true" 或 "false"
  - `canModifyAnnotations` — 必需 — "true" 或 "false"
  - `canPrint` — 必需 — "true" 或 "false"
  - `canPrintHighQuality` — 必需 — "true" 或 "false"
  - `canAssembleDocument` — 必需 — "true" 或 "false"
  - `canFillInForm` — 必需 — "true" 或 "false"
  - `canExtractContent` — 必需 — "true" 或 "false"
  - `canExtractForAccessibility` — 必需 — "true" 或 "false"

通过 ID 获取结果：
- `GET /api/<ID>`

完成后，响应包含：
- `output.files[]`，其中包含 `{ name, path }`，`path` 是可下载的 URL。

## 输入
### 必需
- PDF 文件（二进制）
- 权限 flags（类布尔值），API 全部必需：
  - canModify
  - canModifyAnnotations
  - canPrint
  - canPrintHighQuality
  - canAssembleDocument
  - canFillInForm
  - canExtractContent
  - canExtractForAccessibility
- API key（字符串）

### 可选
- 无

## 默认值（推荐）
如果用户未指定权限，使用保守的默认值，禁止修改和提取，但允许打印：

- canModify: false
- canModifyAnnotations: false
- canPrint: true
- canPrintHighQuality: true
- canAssembleDocument: false
- canFillInForm: true  （如果存在表单，这是合理的默认值）
- canExtractContent: false
- canExtractForAccessibility: true（通常为了无障碍性需要）

这些默认值可以根据产品策略进行调整。

## 输出
返回结构化结果：
- `job_id`（数字）
- `status`（字符串）
- `download_url`（字符串，完成时）
- `file_name`（字符串，可用时）
- `permissions`（对象）反映最终发送的值

输出示例：
```json
{
  "job_id": 7501,
  "status": "done",
  "download_url": "https://.../permissions.pdf",
  "file_name": "permissions.pdf",
  "permissions": {
    "canModify": false,
    "canModifyAnnotations": false,
    "canPrint": true,
    "canPrintHighQuality": true,
    "canAssembleDocument": false,
    "canFillInForm": true,
    "canExtractContent": false,
    "canExtractForAccessibility": true
  }
}
```
