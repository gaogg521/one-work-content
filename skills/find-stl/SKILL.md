---
name: find-stl
description: 搜索并下载可打印的3D模型文件（STL/3MF/ZIP），通过Printables等平台查询概念或特定零件。当Agent需要查找现有模型、获取许可证/归属信息、下载源文件并输出本地文件夹和清单用于报价/打印时触发。
---

# 查找-stl

This skill provides a deterministic 管道:
- 搜索 Printables for models
- select a candidate
- 下载 模型 文件
- 写入 a `manifest.json` (source URL, 作者, 许可证 id, 文件, hashes)

## 快速开始

### 搜索

```bash
python3 scripts/find_stl.py search "iphone 15 pro dock" --limit 10
```

### 获取

```bash
python3 scripts/find_stl.py fetch 1059554 --outdir out/models
```

By default, 获取 downloads **all 模型 文件** (a ZIP pack) when available.

## 注意

- Printables 下载 links are time-limited; this script resolves them via Printables GraphQL (`getDownloadLink`).
- Always preserve 许可证 + attribution in the manifest.

## 资源

- `scripts/find_stl.py`