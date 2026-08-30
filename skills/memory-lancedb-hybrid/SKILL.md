---
name: memory-lancedb-hybrid
description: LanceDB 长期记忆插件，带有 BM25 + 向量混合搜索（RRF 或 linear reranking）。
tags:
- Linear
---

# LanceDB Hybrid Search（Memory Plugin）

本技能打包了一个 **即插即用的 OpenClaw 记忆插件**，为 LanceDB 记忆添加 **混合搜索**：

- **Vector search**（语义）
- **BM25 full-text search**（精确词项）
- 可配置的 reranking：
  - `rrf`（Reciprocal Rank Fusion，推荐）
  - `linear`（加权组合）

它基于（并致谢）OpenClaw PR **openclaw/openclaw#7636**。

## 你能得到什么

一个位于以下位置的本地插件（extension）：

- `plugin/` → **覆盖内置插件 id** `memory-lancedb`（添加混合搜索）

启用后，它提供与捆绑的 LanceDB 记忆插件相同的工具：

- `memory_store`
- `memory_recall`
- `memory_forget`

…but `memory_recall`/auto-recall/forget 现在在启用时使用混合搜索。

## 安装 / 启用

1) 确保技能文件夹存在（ClawHub 安装将其放在你的工作区下）：

- `~/.openclaw/workspace/skills/memory-lancedb-hybrid/plugin`

2) 安装插件依赖（一次）：

```bash
cd ~/.openclaw/workspace/skills/memory-lancedb-hybrid/plugin
npm install --omit=dev
```

3) 将插件添加到 OpenClaw 的插件加载路径。

此插件保持 id `memory-lancedb`，因此当通过 `plugins.load.paths` 发现时，它将 **覆盖** 捆绑的 `memory-lancedb` extension（优先级高于捆绑）。

编辑 `~/.openclaw/openclaw.json`：

```json5
{
  plugins: {
    load: {
      // 指向本技能内部的插件目录
      paths: ["~/.openclaw/workspace/skills/memory-lancedb-hybrid/plugin"]
    },

    // 确保 memory slot 指向 LanceDB memory
    slots: {
      memory: "memory-lancedb"
    },

    // 配置 LanceDB memory（此覆盖添加 `hybrid` 配置块）
    entries: {
      "memory-lancedb": {
        enabled: true,
        config: {
          embedding: {
            apiKey: "${OPENAI_API_KEY}",
            model: "text-embedding-3-small"
          },

          // 可选
          dbPath: "~/.openclaw/memory/lancedb",

          // 可选
          autoCapture: true,
          autoRecall: true,

          // 混合搜索选项
          hybrid: {
            enabled: true,
            reranker: "rrf"

            // 如果使用 reranker: "linear"，还可以设置：
            // vectorWeight: 0.7,
            // textWeight: 0.3,
          }
        }
      }
    }
  }
}
```

4) 重启 Gateway。

混合搜索需要在 `text` 列上建立 FTS index；插件将尝试自动创建它。如果 FTS 设置因任何原因失败，插件会记录 debug 消息并回退到仅向量搜索。

## 配置参考

所有配置位于 `plugins.entries.memory-lancedb.config` 下。

- `hybrid.enabled`（boolean，默认 `true`）
- `hybrid.reranker`（`rrf` | `linear`，默认 `rrf`）
- `hybrid.vectorWeight`（number 0–1，默认 `0.7`，仅用于 `linear`）
- `hybrid.textWeight`（number 0–1，默认 `0.3`，仅用于 `linear`）

## 注意事项 / 故障排查

- 此插件不会修改磁盘上的 OpenClaw 安装；它在运行时 **覆盖** 捆绑的 `memory-lancedb`（移除 `plugins.load.paths` 以恢复）。
- 如果你已经在磁盘上有 LanceDB 记忆数据，你可以继续使用相同的 `dbPath`。
- 如果你看不到混合效果，请确保 `hybrid.enabled` 为 true 并且 FTS index 已创建（检查 Gateway logs）。

## 文件

- `plugin/index.ts` – 插件实现（混合搜索）
- `plugin/config.ts` – 配置解析 + UI hints
- `plugin/openclaw.plugin.json` – manifest + JSON Schema（用于严格的配置验证）
