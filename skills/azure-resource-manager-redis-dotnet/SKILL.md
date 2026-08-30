---
name: azure-resource-manager-redis-dotnet
description: Azure Redis 管理专家 - ARM SDK、缓存配置、防火墙规则、异地复制
license: MIT
metadata:
  author: Microsoft
  version: 1.0.0
  package: Azure.ResourceManager.Redis
---

# Azure.ResourceManager.Redis (.NET)

通过 Azure Resource Manager 配置和管理 Azure Cache for Redis 资源的管理平面 SDK。

> **管理与数据平面**
> - **此 SDK (Azure.ResourceManager.Redis)**：创建缓存、配置防火墙规则、管理访问密钥、设置异地复制
> - **数据平面 SDK (StackExchange.Redis)**：Get/set 键、pub/sub、streams、Lua 脚本

## 安装

```bash
dotnet add package Azure.ResourceManager.Redis
dotnet add package Azure.Identity
```

**当前版本**: 1.5.1 (稳定版)
**API 版本**: 2024-11-01
**目标框架**: .NET 8.0, .NET Standard 2.0

## 环境变量

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
# 对于服务主体身份验证（可选）
AZURE_TENANT_ID=<tenant-id>
AZURE_CLIENT_ID=<client-id>
AZURE_CLIENT_SECRET=<client-secret>
```

## 身份验证

```csharp
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.Redis;

// 始终使用 DefaultAzureCredential
var credential = new DefaultAzureCredential();
var armClient = new ArmClient(credential);

// 获取订阅
var subscriptionId = Environment.GetEnvironmentVariable("AZURE_SUBSCRIPTION_ID");
var subscription = armClient.GetSubscriptionResource(
    new ResourceIdentifier($"/subscriptions/{subscriptionId}"));
```

## 资源层次结构

```
ArmClient
└── SubscriptionResource
    └── ResourceGroupResource
        └── RedisResource
            ├── RedisFirewallRuleResource
            ├── RedisPatchScheduleResource
            ├── RedisLinkedServerWithPropertyResource
            ├── RedisPrivateEndpointConnectionResource
            └── RedisCacheAccessPolicyResource
```

## 核心工作流

### 1. 创建 Redis 缓存

```csharp
using Azure.ResourceManager.Redis;
using Azure.ResourceManager.Redis.Models;

// 获取资源组
var resourceGroup = await subscription
    .GetResourceGroupAsync("my-resource-group");

// 定义缓存配置
var cacheData = new RedisCreateOrUpdateContent(
    location: AzureLocation.EastUS,
    sku: new RedisSku(RedisSkuName.Standard, RedisSkuFamily.BasicOrStandard, 1))
{
    EnableNonSslPort = false,
    MinimumTlsVersion = RedisTlsVersion.Tls1_2,
    RedisConfiguration = new RedisCommonConfiguration
    {
        MaxMemoryPolicy = "volatile-lru"
    },
    Tags =
    {
        ["environment"] = "production"
    }
};

// 创建缓存（长时间运行操作）
var cacheCollection = resourceGroup.Value.GetAllRedis();
var operation = await cacheCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "my-redis-cache",
    cacheData);

RedisResource cache = operation.Value;
Console.WriteLine($"Cache created: {cache.Data.HostName}");
```

### 2. 获取 Redis 缓存

```csharp
// 获取现有缓存
var cache = await resourceGroup.Value
    .GetRedisAsync("my-redis-cache");

Console.WriteLine($"Host: {cache.Value.Data.HostName}");
Console.WriteLine($"Port: {cache.Value.Data.Port}");
Console.WriteLine($"SSL Port: {cache.Value.Data.SslPort}");
Console.WriteLine($"Provisioning State: {cache.Value.Data.ProvisioningState}");
```

### 3. 更新 Redis 缓存

```csharp
var patchData = new RedisPatch
{
    Sku = new RedisSku(RedisSkuName.Standard, RedisSkuFamily.BasicOrStandard, 2),
    RedisConfiguration = new RedisCommonConfiguration
    {
        MaxMemoryPolicy = "allkeys-lru"
    }
};

var updateOperation = await cache.Value.UpdateAsync(
    WaitUntil.Completed,
    patchData);
```

### 4. 删除 Redis 缓存

```csharp
await cache.Value.DeleteAsync(WaitUntil.Completed);
```

### 5. 获取访问密钥

```csharp
var keys = await cache.Value.GetKeysAsync();
Console.WriteLine($"Primary Key: {keys.Value.PrimaryKey}");
Console.WriteLine($"Secondary Key: {keys.Value.SecondaryKey}");
```

### 6. 重新生成访问密钥

```csharp
var regenerateContent = new RedisRegenerateKeyContent(RedisRegenerateKeyType.Primary);
var newKeys = await cache.Value.RegenerateKeyAsync(regenerateContent);
Console.WriteLine($"New Primary Key: {newKeys.Value.PrimaryKey}");
```

### 7. 管理防火墙规则

```csharp
// 创建防火墙规则
var firewallData = new RedisFirewallRuleData(
    startIP: System.Net.IPAddress.Parse("10.0.0.1"),
    endIP: System.Net.IPAddress.Parse("10.0.0.255"));

var firewallCollection = cache.Value.GetRedisFirewallRules();
var firewallOperation = await firewallCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    "allow-internal-network",
    firewallData);

// 列出所有防火墙规则
await foreach (var rule in firewallCollection.GetAllAsync())
{
    Console.WriteLine($"Rule: {rule.Data.Name} ({rule.Data.StartIP} - {rule.Data.EndIP})");
}

// 删除防火墙规则
var ruleToDelete = await firewallCollection.GetAsync("allow-internal-network");
await ruleToDelete.Value.DeleteAsync(WaitUntil.Completed);
```

### 8. 配置补丁计划（Premium SKU）

```csharp
// 补丁计划需要 Premium SKU
var scheduleData = new RedisPatchScheduleData(
    new[]
    {
        new RedisPatchScheduleSetting(RedisDayOfWeek.Saturday, 2) // 周六凌晨 2 点
        {
            MaintenanceWindow = TimeSpan.FromHours(5)
        },
        new RedisPatchScheduleSetting(RedisDayOfWeek.Sunday, 2) // 周日凌晨 2 点
        {
            MaintenanceWindow = TimeSpan.FromHours(5)
        }
    });

var scheduleCollection = cache.Value.GetRedisPatchSchedules();
await scheduleCollection.CreateOrUpdateAsync(
    WaitUntil.Completed,
    RedisPatchScheduleDefaultName.Default,
    scheduleData);
```

### 9. 导入/导出数据（Premium SKU）

```csharp
// 从 blob 存储导入数据
var importContent = new ImportRdbContent(
    files: new[] { "https://mystorageaccount.blob.core.windows.net/container/dump.rdb" },
    format: "RDB");

await cache.Value.ImportDataAsync(WaitUntil.Completed, importContent);

// 导出数据到 blob 存储
var exportContent = new ExportRdbContent(
    prefix: "backup",
    container: "https://mystorageaccount.blob.core.windows.net/container?sastoken",
    format: "RDB");

await cache.Value.ExportDataAsync(WaitUntil.Completed, exportContent);
```

### 10. 强制重启

```csharp
var rebootContent = new RedisRebootContent
{
    RebootType = RedisRebootType.AllNodes,
    ShardId = 0 // 用于集群缓存
};

await cache.Value.ForceRebootAsync(rebootContent);
```

## SKU 参考

| SKU | 系列 | 容量 | 功能 |
|-----|--------|----------|----------|
| Basic | C | 0-6 | 单节点，无 SLA，仅开发/测试 |
| Standard | C | 0-6 | 两个节点（主/副本），SLA |
| Premium | P | 1-5 | 集群、异地复制、VNet、持久化 |

**容量大小（系列 C - Basic/Standard）**：
- C0: 250 MB
- C1: 1 GB
- C2: 2.5 GB
- C3: 6 GB
- C4: 13 GB
- C5: 26 GB
- C6: 53 GB

**容量大小（系列 P - Premium）**：
- P1: 每个分片 6 GB
- P2: 每个分片 13 GB
- P3: 每个分片 26 GB
- P4: 每个分片 53 GB
- P5: 每个分片 120 GB

## 关键类型参考

| 类型 | 用途 |
|------|---------|
| `ArmClient` | 所有 ARM 操作的入口点 |
| `RedisResource` | 表示 Redis 缓存实例 |
| `RedisCollection` | 用于缓存 CRUD 操作的集合 |
| `RedisFirewallRuleResource` | 用于 IP 过滤的防火墙规则 |
| `RedisPatchScheduleResource` | 维护窗口配置 |
| `RedisLinkedServerWithPropertyResource` | 异地复制链接服务器 |
| `RedisPrivateEndpointConnectionResource` | 私有端点连接 |
| `RedisCacheAccessPolicyResource` | RBAC 访问策略 |
| `RedisCreateOrUpdateContent` | 缓存创建负载 |
| `RedisPatch` | 缓存更新负载 |
| `RedisSku` | SKU 配置（名称、系列、容量） |
| `RedisAccessKeys` | 主密钥和辅助访问密钥 |
| `RedisRegenerateKeyContent` | 密钥重新生成请求 |

## 最佳实践

1. **对必须在继续之前完成的操作使用 `WaitUntil.Completed`**
2. **当你想要手动轮询或并行运行操作时使用 `WaitUntil.Started`**
3. **始终使用 `DefaultAzureCredential`** — 绝不要硬编码密钥
4. **处理 `RequestFailedException`** 以处理 ARM API 错误
5. **对幂等操作使用 `CreateOrUpdateAsync`**
6. **通过 `Get*` 方法导航层次结构**（例如 `cache.GetRedisFirewallRules()`）
7. **对需要异地复制、集群或持久化的生产工作负载使用 Premium SKU**
8. **启用 TLS 1.2 最低版本** — 设置 `MinimumTlsVersion = RedisTlsVersion.Tls1_2`
9. **禁用非 SSL 端口** — 为安全起见设置 `EnableNonSslPort = false`
10. **定期轮换密钥** — 使用 `RegenerateKeyAsync` 并更新连接字符串

## 错误处理

```csharp
using Azure;

try
{
    var operation = await cacheCollection.CreateOrUpdateAsync(
        WaitUntil.Completed, cacheName, cacheData);
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Cache already exists");
}
catch (RequestFailedException ex) when (ex.Status == 400)
{
    Console.WriteLine($"Invalid configuration: {ex.Message}");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"ARM Error: {ex.Status} - {ex.ErrorCode}: {ex.Message}");
}
```

## 常见陷阱

1. **不允许 SKU 降级** — 你无法从 Premium 降级到 Standard/Basic
2. **集群需要 Premium** — 分片配置仅在 Premium SKU 上可用
3. **异地复制需要 Premium** — 链接服务器仅适用于 Premium 缓存
4. **VNet 注入需要 Premium** — 虚拟网络支持仅限 Premium
5. **补丁计划需要 Premium** — 维护窗口仅在 Premium 上可配置
6. **缓存名称全局唯一** — Redis 缓存名称必须在所有 Azure 订阅中唯一
7. **长时间配置** — 缓存创建可能需要 15-20 分钟；对异步模式使用 `WaitUntil.Started`

## 与 StackExchange.Redis 连接（数据平面）

使用此管理 SDK 创建缓存后，使用 StackExchange.Redis 进行数据操作：

```csharp
using StackExchange.Redis;

// 从管理 SDK 获取连接信息
var cache = await resourceGroup.Value.GetRedisAsync("my-redis-cache");
var keys = await cache.Value.GetKeysAsync();

// 使用 StackExchange.Redis 连接
var connectionString = $"{cache.Value.Data.HostName}:{cache.Value.Data.SslPort},password={keys.Value.PrimaryKey},ssl=True,abortConnect=False";
var connection = ConnectionMultiplexer.Connect(connectionString);
var db = connection.GetDatabase();

// 数据操作
await db.StringSetAsync("key", "value");
var value = await db.StringGetAsync("key");
```

## 相关 SDK

| SDK | 用途 | 安装 |
|-----|---------|---------|
| `StackExchange.Redis` | 数据平面（get/set、pub/sub、streams） | `dotnet add package StackExchange.Redis` |
| `Azure.ResourceManager.Redis` | 管理平面（此 SDK） | `dotnet add package Azure.ResourceManager.Redis` |
| `Microsoft.Azure.StackExchangeRedis` | Azure 特定的 Redis 扩展 | `dotnet add package Microsoft.Azure.StackExchangeRedis` |
