---
name: volcengine-storage-tos
description: Volcengine TOS 对象存储操作。适用于上传/下载/同步、检查存储桶策略(bucket policy)、生成签名 URL(signed URL)或排查存储问题。
tags:
- Ceph
- MinIO
---

# volcengine-storage-tos

使用显式的路径映射和权限验证来管理 TOS buckets 和 objects。

## 执行清单

1. 确认 bucket、region 和 object 路径。
2. 验证认证和 bucket policy。
3. 执行上传/下载/同步任务。
4. 返回包含 object keys 和 URLs 的结果清单。

## 安全规则

- 没有明确的确认输入时，避免执行破坏性删除。
- 上传时保留 metadata 和 content type。
- 尽可能提供 checksum 或 size 验证。

## 参考

- `references/sources.md`
