---
name: azure-storage-blob-py
description: Azure Blob Storage SDK for Python。用于上传、下载、列出 blobs、管理 containers 以及 blob 生命周期。 触发词：\"blob storage\", \"BlobServiceClient\", \"ContainerClient\", \"BlobClient\", \"upload blob\", \"download blob\"。
package: azure-storage-blob
tags:
- Docker
- Azure
- Ceph
- Python
---

# Azure Blob Storage SDK for Python

Azure Blob Storage 的 client library —— 面向非结构化数据的 object storage。

## 安装

```bash
pip install azure-storage-blob azure-identity
```

## 环境变量

```bash
AZURE_STORAGE_ACCOUNT_NAME=<your-storage-account>
# 或使用完整 URL
AZURE_STORAGE_ACCOUNT_URL=https://<account>.blob.core.windows.net
```

## 认证

```python
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

credential = DefaultAzureCredential()
account_url = "https://<account>.blob.core.windows.net"

blob_service_client = BlobServiceClient(account_url, credential=credential)
```

## Client 层级

| Client | 用途 | 获取方式 |
|--------|---------|----------|
| `BlobServiceClient` | Account-level operations | 直接实例化 |
| `ContainerClient` | Container operations | `blob_service_client.get_container_client()` |
| `BlobClient` | Single blob operations | `container_client.get_blob_client()` |

## 核心工作流

### 创建 Container

```python
container_client = blob_service_client.get_container_client("mycontainer")
container_client.create_container()
```

### 上传 Blob

```python
# 从文件路径
blob_client = blob_service_client.get_blob_client(
    container="mycontainer",
    blob="sample.txt"
)

with open("./local-file.txt", "rb") as data:
    blob_client.upload_blob(data, overwrite=True)

# 从 bytes/string
blob_client.upload_blob(b"Hello, World!", overwrite=True)

# 从 stream
import io
stream = io.BytesIO(b"Stream content")
blob_client.upload_blob(stream, overwrite=True)
```

### 下载 Blob

```python
blob_client = blob_service_client.get_blob_client(
    container="mycontainer",
    blob="sample.txt"
)

# 到文件
with open("./downloaded.txt", "wb") as file:
    download_stream = blob_client.download_blob()
    file.write(download_stream.readall())

# 到内存
download_stream = blob_client.download_blob()
content = download_stream.readall()  # bytes

# 读入已有 buffer
stream = io.BytesIO()
num_bytes = blob_client.download_blob().readinto(stream)
```

### 列出 Blobs

```python
container_client = blob_service_client.get_container_client("mycontainer")

# 列出所有 blobs
for blob in container_client.list_blobs():
    print(f"{blob.name} - {blob.size} bytes")

# 使用 prefix（类似文件夹）
for blob in container_client.list_blobs(name_starts_with="logs/"):
    print(blob.name)

# 遍历 blob 层级（虚拟目录）
for item in container_client.walk_blobs(delimiter="/"):
    if item.get("prefix"):
        print(f"Directory: {item['prefix']}")
    else:
        print(f"Blob: {item.name}")
```

### 删除 Blob

```python
blob_client.delete_blob()

# 连同 snapshots 一起删除
blob_client.delete_blob(delete_snapshots="include")
```

## 性能调优

```python
# 为大型上传/下载配置 chunk 大小
blob_client = BlobClient(
    account_url=account_url,
    container_name="mycontainer",
    blob_name="large-file.zip",
    credential=credential,
    max_block_size=4 * 1024 * 1024,  # 4 MiB blocks
    max_single_put_size=64 * 1024 * 1024  # 64 MiB single upload limit
)

# 并行上传
blob_client.upload_blob(data, max_concurrency=4)

# 并行下载
download_stream = blob_client.download_blob(max_concurrency=4)
```

## SAS Tokens

```python
from datetime import datetime, timedelta, timezone
from azure.storage.blob import generate_blob_sas, BlobSasPermissions

sas_token = generate_blob_sas(
    account_name="<account>",
    container_name="mycontainer",
    blob_name="sample.txt",
    account_key="<account-key>",  # 或使用 user delegation key
    permission=BlobSasPermissions(read=True),
    expiry=datetime.now(timezone.utc) + timedelta(hours=1)
)

# 使用 SAS token
blob_url = f"https://<account>.blob.core.windows.net/mycontainer/sample.txt?{sas_token}"
```

## Blob Properties 和 Metadata

```python
# 获取 properties
properties = blob_client.get_blob_properties()
print(f"Size: {properties.size}")
print(f"Content-Type: {properties.content_settings.content_type}")
print(f"Last modified: {properties.last_modified}")

# 设置 metadata
blob_client.set_blob_metadata(metadata={"category": "logs", "year": "2024"})

# 设置 content type
from azure.storage.blob import ContentSettings
blob_client.set_http_headers(
    content_settings=ContentSettings(content_type="application/json")
)
```

## Async Client

```python
from azure.identity.aio import DefaultAzureCredential
from azure.storage.blob.aio import BlobServiceClient

async def upload_async():
    credential = DefaultAzureCredential()
    
    async with BlobServiceClient(account_url, credential=credential) as client:
        blob_client = client.get_blob_client("mycontainer", "sample.txt")
        
        with open("./file.txt", "rb") as data:
            await blob_client.upload_blob(data, overwrite=True)

# 异步下载
async def download_async():
    async with BlobServiceClient(account_url, credential=credential) as client:
        blob_client = client.get_blob_client("mycontainer", "sample.txt")
        
        stream = await blob_client.download_blob()
        data = await stream.readall()
```

## 最佳实践

1. **使用 DefaultAzureCredential** 替代 connection strings
2. **对 async clients 使用 context managers**
3. **重新上传时显式设置 `overwrite=True`**
4. **对大型文件传输使用 `max_concurrency`**
5. **为内存效率优先使用 `readinto()` 而非 `readall()`**
6. **对层级列表使用 `walk_blobs()`**
7. **为 web-served blobs 设置合适的 content types**
