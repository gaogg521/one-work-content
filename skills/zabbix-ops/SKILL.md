---
name: zabbix-ops
description: Zabbix 运维专家 - 监控配置、告警管理、模板定制、性能优化、分布式部署
---

## 配置说明

### 环境变量配置
```bash
# Zabbix API 配置
export ZABBIX_URL="http://localhost/api_jsonrpc.php"
export ZABBIX_USER="Admin"
export ZABBIX_PASSWORD="zabbix"
export ZABBIX_TOKEN=""

# 服务器配置
export ZABBIX_SERVER_HOST="localhost"
export ZABBIX_SERVER_PORT="10051"
```

### 配置文件示例
```ini
# /etc/zabbix/zabbix_server.conf
ListenPort=10051
LogFile=/var/log/zabbix/zabbix_server.log
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=password
StartPollers=5
StartIPMIPollers=0
StartPollersUnreachable=1
StartTrappers=5
StartPingers=1
StartDiscoverers=1
StartHTTPPollers=1
StartTimers=1
StartEscalators=1
StartAlerters=3
JavaGateway=
JavaGatewayPort=10052
StartJavaPollers=0
StartVMwareCollectors=0
VMwareFrequency=60
VMwarePerfFrequency=60
VMwareCacheSize=8M
VMwareTimeout=10
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
StartSNMPTrapper=0
ListenIP=0.0.0.0
HousekeepingFrequency=1
MaxHousekeeperDelete=5000
CacheSize=8M
CacheUpdateFrequency=60
StartDBSyncers=4
HistoryCacheSize=16M
TrendCacheSize=4M
ValueCacheSize=8M
```

## 输入参数

| 参数名 | 类型 | 必填 | 描述 | 示例 |
|--------|------|------|------|------|
| `host_name` | string | 否 | 主机名称 | `web-server-01` |
| `item_key` | string | 否 | 监控项键值 | `system.cpu.load` |
| `trigger_id` | string | 否 | 触发器 ID | `12345` |
| `time_range` | string | 否 | 时间范围 | `last_hour`, `today` |

## 输出格式

### 主机状态输出
```json
{
  "status": "success",
  "data": {
    "host": {
      "hostid": "10134",
      "host": "web-server-01",
      "name": "Web Server 01",
      "status": 0,
      "available": 1,
      "description": "Production web server",
      "groups": [
        {"name": "Linux servers"},
        {"name": "Web servers"}
      ],
      "interfaces": [
        {
          "ip": "192.168.1.100",
          "dns": "",
          "port": "10050",
          "type": 1
        }
      ],
      "items_count": 156,
      "triggers_count": 23,
      "problems": 2
    }
  }
}
```

# Zabbix 运维助手

你是 Zabbix 监控专家，擅长企业级监控系统部署、模板定制、告警策略配置和性能调优。

## 核心能力

- **Zabbix Server 管理**：高可用架构、数据库优化、Housekeeper 调优
- **Agent 部署管理**：主动/被动模式、自定义 Key、批量部署
- **模板与监控项**：自定义模板、LLD 自动发现、预处理规则
- **告警与通知**：触发器配置、告警升级、媒介类型、动作规则
- **可视化**：仪表盘、拓扑图、聚合图形、SLA 报告
- **API 自动化**：配置管理、批量操作、集成对接
- **性能优化**：历史数据管理、缓存调优、Poller 优化

## 标准诊断流程

### Linux/macOS

```bash
# 1. 检查 Zabbix Server 状态
systemctl status zabbix-server
systemctl status zabbix-agent

# 2. 检查 Zabbix 进程
ps aux | grep zabbix

# 3. 检查端口监听
netstat -tlnp | grep -E "10050|10051"

# 4. 检查数据库连接
mysql -u zabbix -p -e "SELECT COUNT(*) FROM hosts;"

# 5. 查看 Server 日志
tail -f /var/log/zabbix/zabbix_server.log

# 6. 查看 Agent 日志
tail -f /var/log/zabbix/zabbix_agentd.log

# 7. 检查配置语法
zabbix_server -T
zabbix_agentd -t

# 8. 测试 Agent 连通性
zabbix_get -s 127.0.0.1 -k system.uname
```

### Windows (PowerShell)

```powershell
# 1. 检查 Zabbix 服务状态
Get-Service ZabbixServer
Get-Service ZabbixAgent

# 2. 检查 Zabbix 进程
Get-Process | Where-Object {$_.ProcessName -like "*zabbix*"}

# 3. 检查端口监听
Get-NetTCPConnection -LocalPort 10050,10051 | Select-Object LocalAddress, LocalPort, State

# 4. 查看 Server 日志
Get-Content "C:\Program Files\Zabbix\zabbix_server.log" -Tail 100

# 5. 查看 Agent 日志
Get-Content "C:\Program Files\Zabbix\zabbix_agentd.log" -Tail 100

# 6. 检查配置
Test-Path "C:\Program Files\Zabbix\zabbix_server.conf"
Test-Path "C:\Program Files\Zabbix\zabbix_agentd.conf"

# 7. 检查 Windows Agent 安装
Get-ItemProperty "HKLM:\SOFTWARE\Zabbix\Agent"

# 8. 测试 Agent Key
& "C:\Program Files\Zabbix\zabbix_get.exe" -s 127.0.0.1 -k system.uname
```

## 常见故障处理

### 1. Zabbix Server 性能问题

#### Linux/macOS
```bash
# 检查队列长度
mysql -u zabbix -p -e "SELECT COUNT(*) FROM items WHERE status=0 AND nextcheck<NOW();"

# 检查历史数据量
mysql -u zabbix -p -e "
SELECT table_name, ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables
WHERE table_schema = 'zabbix'
GROUP BY table_name
ORDER BY SUM(data_length + index_length) DESC;"

# 查看 Poller 进程数
grep "^StartPollers" /etc/zabbix/zabbix_server.conf

# 优化数据库
mysql -u zabbix -p -e "OPTIMIZE TABLE history;"
mysql -u zabbix -p -e "OPTIMIZE TABLE history_uint;"
```

#### Windows (PowerShell)
```powershell
# 检查 Zabbix 服务资源使用
Get-Process | Where-Object {$_.ProcessName -like "*zabbix*"} |
    Select-Object ProcessName, Id, @{N="MemoryMB";E={[math]::Round($_.WorkingSet64/1MB,2)}}, CPU

# 查看配置文件中的 Poller 设置
Select-String -Path "C:\Program Files\Zabbix\zabbix_server.conf" -Pattern "StartPollers|StartTrappers"

# 检查数据库磁盘空间
Get-WmiObject -Class Win32_LogicalDisk |
    Where-Object {$_.DeviceID -eq "C:"} |
    Select-Object DeviceID,
        @{N="SizeGB";E={[math]::Round($_.Size/1GB,2)}},
        @{N="FreeGB";E={[math]::Round($_.FreeSpace/1GB,2)}},
        @{N="UsedPercent";E={[math]::Round((($_.Size-$_.FreeSpace)/$_.Size)*100,2)}}

# 检查 Windows 事件日志
Get-WinEvent -FilterHashtable @{LogName='Application'; ProviderName='Zabbix*'} -MaxEvents 50
```

### 2. Agent 连接问题

#### Linux/macOS
```bash
# 测试 Agent 连通性
zabbix_get -s <agent_ip> -k system.uname
zabbix_get -s <agent_ip> -k agent.ping

# 检查 Agent 配置
grep -E "^Server|^ListenPort|^Hostname" /etc/zabbix/zabbix_agentd.conf

# 检查防火墙
iptables -L -n | grep 10050

# 重启 Agent
systemctl restart zabbix-agent
```

#### Windows (PowerShell)
```powershell
# 测试 Agent 连通性
& "C:\Program Files\Zabbix\zabbix_get.exe" -s <agent_ip> -k agent.ping

# 检查 Agent 配置
Select-String -Path "C:\Program Files\Zabbix\zabbix_agentd.conf" -Pattern "^Server|^ListenPort|^Hostname"

# 检查 Windows 防火墙
Get-NetFirewallRule | Where-Object {$_.DisplayName -like "*Zabbix*"}

# 检查端口连通性
Test-NetConnection -ComputerName <agent_ip> -Port 10050

# 重启 Zabbix Agent 服务
Restart-Service ZabbixAgent
```

### 3. 告警不触发

#### Linux/macOS
```bash
# 检查触发器状态
mysql -u zabbix -p -e "
SELECT t.description, t.status, h.host
FROM triggers t
JOIN functions f ON t.triggerid = f.triggerid
JOIN items i ON f.itemid = i.itemid
JOIN hosts h ON i.hostid = h.hostid
WHERE t.status = 0
LIMIT 10;"

# 检查动作配置
mysql -u zabbix -p -e "SELECT name, status, eventsource FROM actions;"

# 查看告警日志
grep "action" /var/log/zabbix/zabbix_server.log | tail -50
```

#### Windows (PowerShell)
```powershell
# 使用 Zabbix API 检查触发器
$auth = @{
    jsonrpc = "2.0"
    method = "user.login"
    params = @{ user = "Admin"; password = "zabbix" }
    id = 1
} | ConvertTo-Json

$token = Invoke-RestMethod -Uri "http://localhost/zabbix/api_jsonrpc.php" -Method POST -Body $auth -ContentType "application/json"

# 获取触发器状态
$triggerQuery = @{
    jsonrpc = "2.0"
    method = "trigger.get"
    params = @{
        output = @("triggerid", "description", "status", "value")
        selectHosts = @("host")
        limit = 10
    }
    auth = $token.result
    id = 1
} | ConvertTo-Json -Depth 5

$triggers = Invoke-RestMethod -Uri "http://localhost/zabbix/api_jsonrpc.php" -Method POST -Body $triggerQuery -ContentType "application/json"
$triggers.result | Select-Object @{N="Host";E={$_.hosts[0].host}}, description, status, value

# 检查告警脚本目录
Get-ChildItem "C:\Program Files\Zabbix\alertscripts"
```

## 性能优化配置

### Server 优化

```ini
# /etc/zabbix/zabbix_server.conf

# 进程数调优
StartPollers=100
StartIPMIPollers=10
StartPollersUnreachable=50
StartTrappers=20
StartPingers=20
StartDiscoverers=10
StartHTTPPollers=10
StartTimers=10
StartEscalators=10
StartAlerters=10
StartPreprocessors=20

# 缓存配置
CacheSize=1G
HistoryCacheSize=512M
HistoryIndexCacheSize=128M
TrendCacheSize=256M
ValueCacheSize=512M

# Housekeeper 配置
HousekeepingFrequency=1
MaxHousekeeperDelete=10000

# 超时配置
Timeout=30
TrapperTimeout=300
UnreachablePeriod=45
UnavailableDelay=60
UnreachableDelay=15
```

### Windows 特定优化

```powershell
# 设置 Zabbix 服务启动类型
Set-Service ZabbixServer -StartupType Automatic
Set-Service ZabbixAgent -StartupType Automatic

# 配置 Windows 防火墙
New-NetFirewallRule -DisplayName "Zabbix Server" -Direction Inbound -LocalPort 10051 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "Zabbix Agent" -Direction Inbound -LocalPort 10050 -Protocol TCP -Action Allow
```

## 常用 API 操作

### Linux/macOS

```bash
# 获取认证 Token
TOKEN=$(curl -s -X POST -H "Content-Type: application/json" -d '{
  "jsonrpc": "2.0",
  "method": "user.login",
  "params": { "user": "Admin", "password": "zabbix" },
  "id": 1
}' http://localhost/zabbix/api_jsonrpc.php | jq -r '.result')

# 获取主机列表
curl -s -X POST -H "Content-Type: application/json" -d "{
  \"jsonrpc\": \"2.0\",
  \"method\": \"host.get\",
  \"params\": { \"output\": [\"hostid\", \"host\"], \"selectInterfaces\": [\"ip\"] },
  \"auth\": \"$TOKEN\",
  \"id\": 1
}" http://localhost/zabbix/api_jsonrpc.php | jq

# 获取触发器
curl -s -X POST -H "Content-Type: application/json" -d "{
  \"jsonrpc\": \"2.0\",
  \"method\": \"trigger.get\",
  \"params\": {
    \"output\": [\"triggerid\", \"description\", \"priority\", \"value\"],
    \"filter\": {\"value\": 1},
    \"sortfield\": \"priority\",
    \"sortorder\": \"DESC\"
  },
  \"auth\": \"$TOKEN\",
  \"id\": 1
}" http://localhost/zabbix/api_jsonrpc.php | jq

# 创建主机
curl -s -X POST -H "Content-Type: application/json" -d "{
  \"jsonrpc\": \"2.0\",
  \"method\": \"host.create\",
  \"params\": {
    \"host\": \"new-server\",
    \"interfaces\": [{\"type\": 1, \"main\": 1, \"useip\": 1, \"ip\": \"192.168.1.100\", \"dns\": \"\", \"port\": \"10050\"}],
    \"groups\": [{\"groupid\": \"2\"}],
    \"templates\": [{\"templateid\": \"10001\"}]
  },
  \"auth\": \"$TOKEN\",
  \"id\": 1
}" http://localhost/zabbix/api_jsonrpc.php | jq
```

### Windows (PowerShell)

```powershell
# Zabbix API 基础配置
$zabbixUrl = "http://localhost/zabbix/api_jsonrpc.php"

# 获取认证 Token
$authBody = @{
    jsonrpc = "2.0"
    method = "user.login"
    params = @{ user = "Admin"; password = "zabbix" }
    id = 1
} | ConvertTo-Json

$authResponse = Invoke-RestMethod -Uri $zabbixUrl -Method POST -Body $authBody -ContentType "application/json"
$authToken = $authResponse.result

# 获取主机列表
$hostQuery = @{
    jsonrpc = "2.0"
    method = "host.get"
    params = @{
        output = @("hostid", "host", "status")
        selectInterfaces = @("ip", "dns")
    }
    auth = $authToken
    id = 1
} | ConvertTo-Json -Depth 5

$hosts = Invoke-RestMethod -Uri $zabbixUrl -Method POST -Body $hostQuery -ContentType "application/json"
$hosts.result | ForEach-Object {
    [PSCustomObject]@{
        HostID = $_.hostid
        HostName = $_.host
        Status = $_.status
        IP = $_.interfaces[0].ip
    }
} | Format-Table -AutoSize

# 获取活动告警
$triggerQuery = @{
    jsonrpc = "2.0"
    method = "trigger.get"
    params = @{
        output = @("triggerid", "description", "priority", "value", "lastchange")
        filter = @{value = 1}
        selectHosts = @("host")
        sortfield = "priority"
        sortorder = "DESC"
    }
    auth = $authToken
    id = 1
} | ConvertTo-Json -Depth 5

$triggers = Invoke-RestMethod -Uri $zabbixUrl -Method POST -Body $triggerQuery -ContentType "application/json"
$triggers.result | ForEach-Object {
    [PSCustomObject]@{
        Host = $_.hosts[0].host
        Description = $_.description
        Priority = $_.priority
        LastChange = ([DateTime]::UnixEpoch.AddSeconds($_.lastchange)).ToLocalTime()
    }
} | Format-Table -AutoSize

# 批量添加主机
$newHosts = @(
    @{Host = "server01"; IP = "192.168.1.101"},
    @{Host = "server02"; IP = "192.168.1.102"},
    @{Host = "server03"; IP = "192.168.1.103"}
)

foreach ($host in $newHosts) {
    $createBody = @{
        jsonrpc = "2.0"
        method = "host.create"
        params = @{
            host = $host.Host
            interfaces = @(@{type = 1; main = 1; useip = 1; ip = $host.IP; dns = ""; port = "10050"})
            groups = @(@{groupid = "2"})
            templates = @(@{templateid = "10001"})
        }
        auth = $authToken
        id = 1
    } | ConvertTo-Json -Depth 5

    Invoke-RestMethod -Uri $zabbixUrl -Method POST -Body $createBody -ContentType "application/json"
}

# 导出配置报告
$report = @{
    GeneratedAt = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    TotalHosts = $hosts.result.Count
    ActiveTriggers = ($triggers.result | Where-Object {$_.value -eq 1}).Count
    HostList = $hosts.result | ForEach-Object {$_.host}
}
$report | ConvertTo-Json -Depth 3 | Out-File "C:\Reports\zabbix_status.json" -Encoding UTF8
```

## 输出规范

```
📊 Zabbix 诊断报告

📈 系统状态
- Server 版本：[version]
- 运行时间：[uptime]
- 主机数：[hosts]
- 监控项数：[items]
- 触发器数：[triggers]

💾 资源使用
- 数据库大小：[db_size] GB
- 历史数据：[history_size] GB
- 趋势数据：[trends_size] GB
- Server 内存：[memory_usage] MB
- 队列长度：[queue]

🔍 组件健康
| 组件 | 状态 | 延迟 |
|------|------|------|
| Server | [status] | [latency]ms |
| Agent (Active) | [status] | [latency]ms |
| Agent (Passive) | [status] | [latency]ms |
| Database | [status] | [latency]ms |

🚨 活动告警
| 级别 | 数量 |
|------|------|
| 灾难 | [disaster] |
| 严重 | [high] |
| 一般 | [average] |
| 警告 | [warning] |
| 信息 | [information] |

💡 性能建议
- [建议1]
- [建议2]
- [建议3]
```

## 参考资源

- [Zabbix 官方文档](https://www.zabbix.com/documentation)
- [Zabbix 性能调优指南](https://www.zabbix.com/documentation/current/en/manual/installation/requirements)
- [Zabbix API 文档](https://www.zabbix.com/documentation/current/en/manual/api)
