---
name: oraclecloud-install-auth
description: 安装和配置 Oracle Cloud Infrastructure (OCI) SDK 和 CLI 认证。 在设置新的 OCI 集成、生成 API 签名密钥或调试配置文件错误时使用。 使用 \"install oraclecloud\"、\"setup oci auth\"、\"oraclecloud credentials\"、\"oci config\" 触发。
allowed-tools: Read, Write, Edit, Bash(pip:*), Bash(oci:*), Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
tags:
- saas
- oraclecloud
- oci
compatible-with: claude-code
---

# Oracle Cloud 安装与认证

## 概述

为 Oracle Cloud Infrastructure (OCI) 配置 API 密钥认证。OCI 认证需要一个包含**五个必填字段**的 `~/.oci/config` 文件 —— 用户 OCID、指纹、租户 OCID、区域和 RSA 私钥路径。一个错误的字段会产生晦涩的 `ConfigFileNotFound` 或 `InvalidKeyFilePath` 错误，不提示哪个字段失败。

**目的：** 生成一个经验证的 `~/.oci/config` 文件，生成 RSA 密钥对，将公钥上传到 OCI，并通过 Python SDK 和 OCI CLI 验证连接。

## 前置条件

- 具有活跃租户的 **OCI 账户** —— 在 https://cloud.oracle.com 注册
- **Python 3.8+**（OCI Python SDK 是最成熟的 SDK）
- 已安装 **OpenSSL**（用于 RSA 密钥生成）
- 您的 **用户 OCID**（OCI 控制台中的个人资料 > 用户设置）—— 格式：`ocid1.user.oc1..aaaa...`
- 您的 **租户 OCID**（管理 > 租户详情）—— 格式：`ocid1.tenancy.oc1..aaaa...`
- 您的 **主区域**（例如，`us-ashburn-1`、`eu-frankfurt-1`）

## 说明

### 步骤 1：安装 OCI Python SDK 和 CLI

```bash
pip install oci oci-cli
```

### 步骤 2：生成 RSA 密钥对

```bash
mkdir -p ~/.oci
openssl genrsa -out ~/.oci/oci_api_key.pem 2048
chmod 600 ~/.oci/oci_api_key.pem
openssl rsa -pubout -in ~/.oci/oci_api_key.pem -out ~/.oci/oci_api_key_public.pem
```

### 步骤 3：获取密钥指纹

```bash
openssl rsa -pubout -outform DER -in ~/.oci/oci_api_key.pem | openssl md5 -c
# 输出：ab:cd:ef:12:34:56:78:90:ab:cd:ef:12:34:56:78:90
```

### 步骤 4：将公钥上传到 OCI 控制台

导航至：**个人资料（右上角）> 用户设置 > API 密钥 > 添加 API 密钥 > 粘贴公钥**

粘贴 `~/.oci/oci_api_key_public.pem` 的内容。控制台会显示指纹 —— 它必须与步骤 3 匹配。

### 步骤 5：创建配置文件

```bash
cat > ~/.oci/config << 'EOF'
[DEFAULT]
user=ocid1.user.oc1..aaaa_YOUR_USER_OCID
fingerprint=ab:cd:ef:12:34:56:78:90:ab:cd:ef:12:34:56:78:90
tenancy=ocid1.tenancy.oc1..aaaa_YOUR_TENANCY_OCID
region=us-ashburn-1
key_file=~/.oci/oci_api_key.pem
EOF
chmod 600 ~/.oci/config
```

所有五个字段都是必需的。`key_file` 必须指向**私钥**（不是公钥 `.pem`）。

### 步骤 6：使用 Python SDK 验证

```python
import oci

config = oci.config.from_file("~/.oci/config")
oci.config.validate_config(config)

identity = oci.identity.IdentityClient(config)
user = identity.get_user(config["user"]).data
print(f"已认证为：{user.name} ({user.email})")
print(f"租户：{config['tenancy']}")
print(f"区域：{config['region']}")
```

### 步骤 7：使用 OCI CLI 验证

```bash
oci iam user get --user-id "$(grep ^user ~/.oci/config | cut -d= -f2)" \
  --query 'data.name' --raw-output
```

### 步骤 8：配置验证脚本

将此保存为 `validate_oci_config.py` 以捕获常见配置错误：

```python
import oci
import os

def validate():
    """验证 OCI 配置文件和密钥访问。"""
    config_path = os.path.expanduser("~/.oci/config")
    if not os.path.exists(config_path):
        raise FileNotFoundError(f"配置未找到：{config_path}")

    config = oci.config.from_file(config_path)
    oci.config.validate_config(config)

    key_path = os.path.expanduser(config.get("key_file", ""))
    if not os.path.exists(key_path):
        raise FileNotFoundError(f"私钥未找到：{key_path}")

    perms = oct(os.stat(key_path).st_mode)[-3:]
    if perms != "600":
        print(f"警告：密钥文件权限为 {perms}，应为 600")

    identity = oci.identity.IdentityClient(config)
    identity.get_user(config["user"])
    print("配置有效。认证成功。")

validate()
```

## 输出

成功完成将产生：
- 位于 `~/.oci/oci_api_key.pem`（私钥）和 `~/.oci/oci_api_key_public.pem`（公钥）的 RSA 密钥对
- 包含所有五个必需字段的经验证的 `~/.oci/config`
- 使用匹配指纹上传到您的 OCI 用户个人资料的公钥
- 通过 Python SDK 或 OCI CLI 确认的 API 连接

## 错误处理

| 错误 | 代码 | 原因 | 解决方案 |
|-------|------|-------|----------|
| NotAuthenticated | 401 | 指纹错误或密钥不匹配 | 验证指纹匹配：`openssl rsa -pubout -outform DER -in ~/.oci/oci_api_key.pem \| openssl md5 -c` |
| ConfigFileNotFound | — | `~/.oci/config` 缺失 | 运行 `oci setup config` 或按步骤 5 手动创建 |
| InvalidKeyFilePath | — | `key_file` 指向错误路径或公钥 | 确保 `key_file=~/.oci/oci_api_key.pem`（私钥，无 `_public`） |
| InvalidPrivateKey | — | 密钥文件是公钥，不是私钥 | 私钥以 `-----BEGIN RSA PRIVATE KEY-----` 开头 |
| NotAuthorizedOrNotFound | 404 | 用户 OCID 错误或缺少 IAM 策略 | 在控制台 > 个人资料 > 用户设置中仔细检查用户 OCID |
| CERTIFICATE_VERIFY_FAILED | — | 企业代理后的 SSL 证书问题 | 设置 `OCI_PYTHON_SDK_NO_SERVICE_IMPORTS=1` 或安装 `certifi` |

## 示例

**使用 curl 快速认证测试（无需 SDK）：**

```bash
# 验证 OCI CLI 可以访问 API
oci iam region list --output table
```

**用于开发/测试/生产的多个配置文件：**

```ini
# ~/.oci/config
[DEFAULT]
user=ocid1.user.oc1..aaaa_PROD_USER
tenancy=ocid1.tenancy.oc1..aaaa_PROD
region=us-ashburn-1
fingerprint=ab:cd:...
key_file=~/.oci/oci_api_key.pem

[STAGING]
user=ocid1.user.oc1..aaaa_STAGING_USER
tenancy=ocid1.tenancy.oc1..aaaa_STAGING
region=us-phoenix-1
fingerprint=12:34:...
key_file=~/.oci/oci_api_key_staging.pem
```

```python
# 加载特定配置文件
config = oci.config.from_file("~/.oci/config", profile_name="STAGING")
```

## 资源

- [OCI API 密钥认证](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm) —— 密钥生成和配置文件格式
- [OCI Python SDK 参考](https://docs.oracle.com/en-us/iaas/tools/python/latest/) —— 完整 API 文档
- [OCI CLI 参考](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cliconcepts.htm) —— 命令行界面指南
- [SDK 故障排除](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdk_troubleshooting.htm) —— 常见认证和连接问题
- [OCI 控制台](https://cloud.oracle.com) —— 用于密钥上传和 OCID 查找的 Web 仪表板
- [Always Free 层级](https://www.oracle.com/cloud/free/) —— 免费 OCI 开发资源

## 后续步骤

认证正常工作后，继续到 `oraclecloud-hello-world` 启动您的第一个计算实例，或如果遇到认证问题请参见 `oraclecloud-common-errors`。
