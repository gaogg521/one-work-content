---
name: lastpass-cli
description: 通过lpass CLI从LastPass保险库安全获取凭据。
version: 0.1.0
tags:
- 安全
- passwords
- lastpass
---

# LastPass CLI Skill

## 描述

此技能使代理能够使用本地 `lpass` CLI从本地LastPass保险库检索凭据。它旨在将凭据获取到自动化流程中，而不是用于交互式保险库管理。

## 工具

- `lastpass_get_secret`: 使用本地 `lpass` CLI检索命名LastPass条目的特定字段（密码、用户名、备注）。

## 何时使用

- 当你需要存储在LastPass中的特定帐户的密码、用户名或备注时。
- 在编排需要密钥的部署、API调用或登录时。

## 工具: lastpass_get_secret

### 调用

使用JSON对象调用此工具：

```json
{
  "name": "Exact LastPass entry name",
  "field": "password | username | notes | raw"
}
```
