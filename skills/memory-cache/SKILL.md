---
name: memory-cache
description: 使用 Redis 的高性能临时存储系统。支持命名空间键（mema:*）、TTL 管理和会话上下文缓存。用于：(1) 保存代理状态，(2) 缓存 API 结果，(3) 在子代理之间共享数据。
metadata:
  openclaw:
    requires:
      bins:
      - python3
      env:
      - REDIS_URL
    install:
    - id: pip-dependencies
      kind: exec
      command: pip install -r requirements.txt
---

# 内存缓存

OpenClaw 代理的标准化 Redis 支持缓存系统。

## 先决条件
- **二进制文件**: 主机上必须可用 `python3`。
- **凭证**: `REDIS_URL` 环境变量（例如，`redis://localhost:6379/0`）。

## 设置
1. 将 `env.example.txt` 复制到 `.env`。
2. 在 `.env` 中配置你的连接。
3. 依赖项列在 `requirements.txt` 中。

## 核心工作流

### 1. 存储和检索
- **存储**: `python3 $WORKSPACE/skills/memory-cache/scripts/cache_manager.py set mema:cache:<name> <value> [--ttl 3600]`
- **获取**: `python3 $WORKSPACE/skills/memory-cache/scripts/cache_manager.py get mema:cache:<name>`

### 2. 搜索和维护
- **扫描**: `python3 $WORKSPACE/skills/memory-cache/scripts/cache_manager.py scan [pattern]`
- **Ping**: `python3 $WORKSPACE/skills/memory-cache/scripts/cache_manager.py ping`

## 键命名约定

严格执行 `mema:` 前缀：
- `mema:context:*` – 会话状态。
- `mema:cache:*` – 易失性数据。
- `mema:state:*` – 持久状态。
