---
name: oraclecloud-common-errors
description: 诊断和修复 Oracle Cloud Infrastructure API 错误，提供真实错误代码和经过验证的修复方案。 在遇到 OCI ServiceError 异常、认证失败、SSL 问题或超时错误时使用。 使用 \"oci error\"、\"fix oraclecloud\"、\"debug oci\"、\"NotAuthorizedOrNotFound\"、\"oci 401\" 触发。
allowed-tools: Read, Grep, Bash(oci:*), Bash(pip:*), Bash(openssl:*), Bash(curl:*)
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
tags:
- saas
- oraclecloud
- oci
compatible-with: claude-code
---

# Oracle Cloud 常见错误

## 概述

OCI API 错误是出了名的晦涩难懂。`NotAuthorizedOrNotFound` (404) 可能意味着 IAM 策略缺失**或者**您输入的 OCID 有误 —— 出于安全考虑，该错误故意含糊不清。`NotAuthenticated` (401) 涵盖了六种不同的配置文件问题。`CERTIFICATE_VERIFY_FAILED` 的修复方案因 SDK 语言、操作系统以及是否位于企业代理之后而异。

**目的：** 一个诊断解码器，将每个常见的 OCI 错误映射到其真正的根本原因和修复方案，并为每种场景提供可运行的诊断命令。

## 前置条件

- 配置了 `~/.oci/config` 的 OCI 账户（参见 `oraclecloud-install-auth`）
- **Python 3.8+** 并安装了 `pip install oci`
- **OCI CLI** 已安装 (`pip install oci-cli`) 用于诊断命令

## 说明

### 认证错误 (401 NotAuthenticated)

401 错误有六种不同的根本原因。运行此诊断以隔离是哪一种：

```python
import oci
import os

def diagnose_auth():
    """诊断 OCI 401 NotAuthenticated 错误。"""
    config_path = os.path.expanduser("~/.oci/config")

    # 检查 1：配置文件是否存在
    if not os.path.exists(config_path):
        return "原因：配置文件缺失。运行：oci setup config"

    config = oci.config.from_file(config_path)

    # 检查 2：所有必需字段是否存在
    required = ["user", "fingerprint", "tenancy", "region", "key_file"]
    for field in required:
        if field not in config or not config[field]:
            return f"原因：~/.oci/config 中缺少字段 '{field}'"

    # 检查 3：密钥文件是否存在
    key_path = os.path.expanduser(config["key_file"])
    if not os.path.exists(key_path):
        return f"原因：密钥文件未找到：{key_path}"

    # 检查 4：密钥是私钥（不是公钥）
    with open(key_path, "r") as f:
        first_line = f.readline().strip()
    if "PUBLIC" in first_line:
        return "原因：key_file 指向公钥。使用私钥（无 _public 后缀）"

    # 检查 5：密钥文件权限
    perms = oct(os.stat(key_path).st_mode)[-3:]
    if perms != "600":
        return f"原因：密钥权限为 {perms}。运行：chmod 600 {key_path}"

    # 检查 6：指纹匹配
    return "配置看起来有效。指纹不匹配？在控制台 > 用户设置 > API 密钥中验证"

print(diagnose_auth())
```

**401 根本原因表：**

| 症状 | 根本原因 | 修复方案 |
|---------|-----------|-----|
| `did not find a proper configuration for user` | `~/.oci/config` 缺失或格式错误 | 使用 `oci setup config` 重新创建 |
| `could not find private key` | `key_file` 路径错误或文件缺失 | 检查路径；密钥必须存在于该位置 |
| `private key does not match` | 配置中的指纹与上传的公钥不匹配 | 上传正确的公钥或重新生成密钥对 |
| `key_file points to public key` | `key_file=~/.oci/oci_api_key_public.pem` | 更改为 `key_file=~/.oci/oci_api_key.pem`（私钥） |
| `permission denied reading key` | 密钥文件权限过于开放 | `chmod 600 ~/.oci/oci_api_key.pem` |
| `InvalidKeyId` | 用户 OCID 或租户 OCID 错误 | 在控制台 > 个人资料 > 用户设置中验证 OCID |

### 授权错误 (404 NotAuthorizedOrNotFound)

这是最令人困惑的 OCI 错误。404 意味着**要么**资源不存在**要么**您缺少 IAM 权限查看它。OCI 故意隐藏了这种区别。

**诊断步骤：**

```bash
# 步骤 1：验证 OCID 格式是否正确
echo "ocid1.instance.oc1.iad.aaaa..." | grep -P '^ocid1\.\w+\.oc1\.'

# 步骤 2：验证您可以在区间中列出资源
oci compute instance list --compartment-id <COMPARTMENT_OCID> --limit 1

# 步骤 3：检查影响您用户的 IAM 策略
oci iam policy list --compartment-id <TENANCY_OCID> --all \
  --query "data[?contains(statements[0], 'instances')]"
```

**常见 IAM 策略修复：**

```
# 允许组在区间中管理计算资源
allow group Developers to manage instances in compartment MyCompartment

# 允许组读取所有资源（对调试有用）
allow group Developers to read all-resources in tenancy
```

### SSL 证书错误 (CERTIFICATE_VERIFY_FAILED)

```python
# 诊断：检查 Python 使用哪个 CA 捆绑包
import ssl
print(ssl.get_default_verify_paths())

# 修复 1：安装/更新 certifi
# pip install --upgrade certifi

# 修复 2：指向系统 CA 捆绑包（Linux）
import os
os.environ["REQUESTS_CA_BUNDLE"] = "/etc/ssl/certs/ca-certificates.crt"

# 修复 3：企业代理 —— 将代理 CA 添加到 certifi 捆绑包
# cat proxy-ca.pem >> $(python -c "import certifi; print(certifi.where())")
```

**按环境划分的 SSL 修复：**

| 环境 | 修复方案 |
|------------|-----|
| Linux (系统 Python) | `pip install --upgrade certifi` |
| macOS | `brew install ca-certificates` + 更新 certifi |
| 企业代理 | 将代理 CA 证书添加到 certifi 捆绑包 |
| Docker 容器 | 复制 CA 捆绑包：`COPY ca-bundle.crt /etc/ssl/certs/` |
| 隔离网络 | 将 `REQUESTS_CA_BUNDLE` 设置为本地 CA 文件 |

### 速率限制错误 (429 TooManyRequests)

OCI **不**发送 `Retry-After` 头部。您必须实现自己的退避：

```python
import oci
import time
import random

def handle_rate_limit(fn, max_retries=5):
    """处理 OCI 429 错误。无 Retry-After 头部 —— 使用抖动退避。"""
    for attempt in range(max_retries):
        try:
            return fn()
        except oci.exceptions.ServiceError as e:
            if e.status == 429:
                delay = (2 ** attempt) + random.uniform(0, 2)
                print(f"速率受限。等待 {delay:.1f}秒（尝试 {attempt + 1}）")
                time.sleep(delay)
            else:
                raise
    raise RuntimeError("速率限制重试已耗尽")

# 或使用内置策略
compute = oci.core.ComputeClient(
    config, retry_strategy=oci.retry.DEFAULT_RETRY_STRATEGY
)
```

### 超时错误 (ServiceError status -1)

`status == -1` 的 `ServiceError` 意味着 SDK 在服务器响应之前超时：

```python
import oci

config = oci.config.from_file("~/.oci/config")

# 默认没有连接超时。始终同时设置两者：
compute = oci.core.ComputeClient(
    config,
    timeout=(10, 60)  # (connect_timeout, read_timeout)
)

# 对象存储上传需要更长的读取超时
object_storage = oci.object_storage.ObjectStorageClient(
    config,
    timeout=(10, 600)  # 大文件上传 10 分钟
)
```

### 内部服务器错误 (500 InternalError)

```python
import oci

# 首先检查 OCI 服务健康状态
# https://ocistatus.oraclecloud.com

# 500 错误是暂时的 —— 使用重试策略
compute = oci.core.ComputeClient(
    oci.config.from_file("~/.oci/config"),
    retry_strategy=oci.retry.DEFAULT_RETRY_STRATEGY  # 重试 500, 502, 503, 504
)
```

## 输出

完成本诊断指南后，您将：
- 从错误代码和消息中识别出 OCI 错误的确切根本原因
- 一个可运行的 401 认证错误诊断脚本，检查所有六种故障模式
- 解决 404 授权错误的 IAM 策略语句
- 针对 429 和 500 错误的正确超时和重试配置
- 针对您特定环境的 SSL 证书修复

## 错误处理

**完整的 OCI 错误参考表：**

| 错误 | 状态 | 异常 | 根本原因 | 修复方案 |
|-------|--------|-----------|-----------|-----|
| NotAuthenticated | 401 | `ServiceError` | 错误的配置、密钥或指纹 | 运行上面的 `diagnose_auth()` |
| NotAuthorizedOrNotFound | 404 | `ServiceError` | 缺少 IAM 策略或错误的 OCID | 检查 OCID 格式，然后检查 IAM 策略 |
| TooManyRequests | 429 | `ServiceError` | 超出速率限制 | 使用抖动退避（无 Retry-After 头部） |
| InternalError | 500 | `ServiceError` | OCI 服务错误 | 重试；检查 https://ocistatus.oraclecloud.com |
| Timeout | -1 | `ServiceError` | 响应超时 | 在客户端上设置 `timeout=(connect, read)` |
| CERTIFICATE_VERIFY_FAILED | — | `SSLError` | CA 捆绑包过期或代理 | 更新 certifi 或设置 REQUESTS_CA_BUNDLE |
| InvalidParameter | 400 | `ServiceError` | 请求体格式错误 | 检查 OCID 格式和必填字段 |
| Out of host capacity | 500 | `ServiceError` | 无可用主机（ARM 形状） | 重试循环；参见 `oraclecloud-hello-world` |

## 示例

**一键连接测试：**

```bash
# 在一次调用中测试认证 + 网络 + SSL
oci iam region list --output table 2>&1 || echo "失败 —— 运行 oraclecloud-common-errors 诊断"
```

**脚本的万能错误处理程序：**

```python
import oci
import sys

config = oci.config.from_file("~/.oci/config")
compute = oci.core.ComputeClient(config)

try:
    compute.list_instances(compartment_id=config["tenancy"])
    print("OK")
except oci.exceptions.ServiceError as e:
    print(f"OCI 错误：{e.status} {e.code}")
    print(f"消息：{e.message}")
    print(f"请求 ID：{e.request_id}")  # 在支持工单中包含此信息
    if e.status == 401:
        print("操作：检查 ~/.oci/config —— 参见 oraclecloud-install-auth")
    elif e.status == 404:
        print("操作：验证 OCID 和 IAM 策略 —— 参见上面的 404 部分")
    elif e.status == 429:
        print("操作：速率受限 —— 实现退避")
    elif e.status == -1:
        print("操作：超时 —— 增加客户端超时值")
    sys.exit(1)
except oci.exceptions.ConfigFileNotFoundError:
    print("操作：~/.oci/config 未找到。运行：oci setup config")
    sys.exit(1)
```

## 资源

- [OCI SDK 故障排除](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdk_troubleshooting.htm) —— 官方错误解决指南
- [OCI 已知问题](https://docs.oracle.com/en-us/iaas/Content/knownissues.htm) —— 活跃的错误和变通方案
- [OCI 状态页面](https://ocistatus.oraclecloud.com) —— 实时服务健康仪表板
- [OCI API 参考](https://docs.oracle.com/en-us/iaas/api/) —— 每个服务的错误响应模式
- [OCI Python SDK 参考](https://docs.oracle.com/en-us/iaas/tools/python/latest/) —— 异常类层次结构
- [OCI IAM 策略](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policygetstarted.htm) —— 404 修复的策略语法

## 后续步骤

错误解决后，参见 `oraclecloud-sdk-patterns` 了解主动预防错误的重试和超时模式，或参见 `oraclecloud-local-dev-loop` 了解具有快速反馈的高效开发工作流。
