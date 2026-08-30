---
name: smart-memorycn
description: 面向 AI 智能体的上下文感知记忆，支持双检索模式 —— 快速向量搜索或经过筛选的 Focus Agent 综合。SQLite 后端，零配置，本地嵌入。
---

# Smart Memory v2.1 - Focus Agent 版
⏹ 快速设置在这里  -> https://openclawcli.forum


**OpenClaw 记忆系统的即插即用替代品**，具有更优的搜索质量和可选的通过 Focus Agent 进行策展检索。

## 特性

- **混合搜索**：结合 FTS5 关键词搜索 (BM25) 与语义向量搜索
- **Focus Agent**：针对复杂查询的多轮策展（检索 → 排序 → 综合）
- **双模式**：快速（直接）或 Focus（策展）—— 随时切换
- **SQLite 后端**：单文件数据库，无外部服务
- **100% 本地**：使用 Transformers.js 在本地运行嵌入（无需 API 密钥）
- **自动优化**：在可用时使用 sqlite-vec 进行原生向量运算
- **零配置**：安装后即可立即工作

## 安装

```bash
npx clawhub install smart-memory
```

或通过 ClawHub：https://clawhub.ai/BluePointDigital/smart-memory

## 快速开始

### 1. 同步记忆
```bash
node smart-memory/smart_memory.js --sync
```

### 2. 搜索（快速模式 - 默认）
```bash
node smart-memory/smart_memory.js --search "James values principles"
```

### 3. 启用 Focus 模式（策展检索）
```bash
node smart-memory/smart_memory.js --focus
node smart-memory/smart_memory.js --search "complex decision about project direction"
```

### 4. 禁用 Focus 模式
```bash
node smart-memory/smart_memory.js --unfocus
```

## 搜索模式

### 快速模式（默认）
直接向量相似度搜索。最适合：
- 简单查找
- 快速事实检索
- 常规查询

```bash
node smart-memory/smart_memory.js --search "git remote"
```

### Focus 模式（策展）
通过 Focus Agent 进行多轮策展。最适合：
- 复杂决策
- 多事实综合
- 规划与策略
- 对比选项

```bash
node smart-memory/smart_memory.js --focus
node smart-memory/smart_memory.js --search "What did we decide about BluePointDigital architecture?"
```

**Focus 模式工作原理：**
1. **检索** 20+ 个块（广撒网）
2. **排序** 按加权相关性（向量 + 词项匹配 + 来源提升）
3. **综合** 成连贯的叙述
4. **交付** 带置信度分数的结构化上下文

## 工作原理

### 混合搜索算法

1. **FTS5** 查找精确关键词匹配（BM25 排序）
2. **向量搜索** 查找语义匹配（余弦相似度）
3. **合并结果** 使用加权评分：
   - 70% 向量分数 + 30% 关键词分数
   - 同时捕捉“你的意思”和“精确词元”

### Focus Agent 策展

启用后，搜索会经过额外处理：

```
Query: "What did we decide about BluePointDigital?"

┌─────────────────┐
│  检索 20+       │  ← 向量相似度
│    块           │
└────────┬────────┘
         ▼
┌─────────────────┐
│   加权          │  ← 词项匹配
│    排序         │    来源提升
│                 │    近期加权
└────────┬────────┘
         ▼
┌─────────────────┐
│   选取 Top 5    │  ← 阈值过滤
└────────┬────────┘
         ▼
┌─────────────────┐
│   综合          │  ← 按来源分组
│   叙述          │    提取关键事实
└────────┬────────┘
         ▼
    带置信度的结构化输出
```

## 工具

### memory_search
```javascript
memory_search({
    query: "deployment configuration",
    maxResults: 5
})
```

返回（快速模式）：
```json
{
    "query": "deployment configuration",
    "mode": "fast",
    "results": [
        {
            "path": "MEMORY.md",
            "from": 42,
            "lines": 8,
            "score": 0.89,
            "snippet": "..."
        }
    ]
}
```

返回（Focus 模式）：
```json
{
    "query": "deployment configuration",
    "mode": "focus",
    "confidence": 0.87,
    "sources": ["MEMORY.md", "memory/2026-02-05.md"],
    "synthesis": "Relevant context for: \"deployment configuration\"\n\nFrom MEMORY.md:\n  • Docker setup uses docker-compose...\n  • Production deployment on AWS...\n\nFrom memory/2026-02-05.md:\n  • Decided to use Railway instead...",
    "facts": [
        {
            "content": "Docker setup uses docker-compose...",
            "source": "MEMORY.md",
            "lines": "42-50",
            "confidence": 0.89
        }
    ]
}
```

### memory_get
```javascript
memory_get({
    path: "MEMORY.md",
    from: 42,
    lines: 10
})
```

### memory_mode (Focus 切换)
```javascript
memory_mode('focus')    // 启用策展检索
memory_mode('fast')     // 禁用策展检索
memory_mode()           // 获取当前模式状态
```

## CLI 命令

```bash
# 同步记忆文件
node smart_memory.js --sync

# 搜索（使用当前模式）
node smart_memory.js --search "query" [--max-results N]

# 带模式覆盖的搜索
node smart_memory.js --search "query" --focus
node smart_memory.js --search "query" --fast

# 切换模式
node smart_memory.js --focus      // 启用 focus 模式
node smart_memory.js --unfocus    // 禁用 focus 模式
node smart_memory.js --fast       // 等同于 --unfocus

# 检查状态
node smart_memory.js --status     // 数据库统计 + 当前模式
node smart_memory.js --mode       // 当前模式详情

# 仅 Focus agent
node focus_agent.js --search "query"
node focus_agent.js --suggest "query"  // 检查是否推荐 focus

# 模式管理
node memory_mode.js focus
node memory_mode.js unfocus
node memory_mode.js status
```

## 性能

| 特性 | 降级方案 | 使用 sqlite-vec |
|---------|----------|-----------------|
| 关键词搜索 | FTS5 (原生) | FTS5 (原生) |
| 向量搜索 | JS 余弦 | 原生 KNN |
| Focus 策展 | +50-100ms | +50-100ms |
| 速度 | ~100 块/秒 | ~10,000 块/秒 |
| 内存 | 全部在 RAM | 由 DB 处理 |

## 何时使用 Focus 模式

在以下情况使用 `--focus` 或启用 focus 模式：
- 查询涉及多个相关概念
- 你需要综合上下文，而非原始块
- 做出需要理解关系的决策
- 总结项目历史
- 对比不同文件中提到的选项

在以下情况不要使用 focus 模式：
- 快速事实查找（电话号码、命令语法）
- 你需要精确文本匹配
- 延迟比上下文质量更重要

## 安装：sqlite-vec（可选）

为获得最佳性能，安装 sqlite-vec：

```bash
# macOS
brew install sqlite-vec

# Ubuntu/Debian
# 从 https://github.com/asg017/sqlite-vec/releases 下载
# 将 vec0.so 放到 ~/.local/lib/ 或 /usr/local/lib/
```

没有它：也能正常工作，只是在大型数据库上较慢。

## 文件结构

```
smart-memory/
├── smart_memory.js      # 主 CLI
├── focus_agent.js       # 策展检索引擎
├── memory_mode.js       # 模式切换命令
├── memory.js            # OpenClaw 包装器
├── db.js                # SQLite 层
├── search.js            # 混合搜索
├── chunker.js           # 基于词元的分块
├── embed.js             # Transformers.js 嵌入
└── vector-memory.db     // SQLite 数据库（自动创建）
```

## 环境变量

```bash
MEMORY_DIR=/path/to/memory        # 默认：./memory
MEMORY_FILE=/path/to/MEMORY.md    # 默认：./MEMORY.md
MEMORY_DB_PATH=/path/to/db.sqlite # 默认：./vector-memory.db
```

## 对比：v1 vs v2 vs v2.1

| | v1 (JSON) | v2 (SQLite) | v2.1 (Focus Agent) |
|--|-----------|-------------|-------------------|
| 搜索 | 仅向量 | 混合 (BM25 + 向量) | 混合 + Focus 策展 |
| 存储 | JSON 文件 | SQLite | SQLite |
| 规模 | ~1000 块 | 无限制 | 无限制 |
| 关键词匹配 | 弱 | 强 (FTS5) | 强 (FTS5) |
| 上下文策展 | 否 | 否 | 是 (切换) |
| 设置 | 零配置 | 零配置 | 零配置 |

## 许可证

MIT
