---
name: volcengine-security-kms
description: 使用 Volcengine KMS 进行密钥生命周期管理。当用户需要密钥创建、轮换策略、加密/解密工作流或密钥权限故障排除时使用。
tags:
- 安全
---

# volcengine-security-kms

以生命周期感知和最小权限访问检查操作 KMS 密钥。

## 执行清单

1. 确认密钥用途、算法和使用范围。
2. 创建或选择密钥并验证策略绑定。
3. 执行加密/解密/签名任务。
4. 返回密钥元数据、操作结果和审计提示。

## 安全规则

- 切勿在日志中暴露明文密钥。
- 按照策略窗口轮换密钥。
- 在密钥操作前验证调用者权限。

## 参考

- `references/sources.md`
