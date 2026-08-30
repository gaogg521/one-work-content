---
name: stash-name-gen
description: 根据你的更改生成有意义的 git stash 名称。在暂存工作时使用。
---

# Stash Namer

停止将 stash 命名为 "WIP" 或留空。此工具会读取你的更改并创建一个有意义 stash 名称。

**一个命令。零配置。开箱即用。**

## 快速开始

```bash
npx ai-stash-name
```

## 功能

- 读取你的已暂存和未暂存更改
- 生成描述性的 stash 名称
- 实际使用该名称运行 git stash
- 不再有神秘的 stash

## 使用示例

```bash
# 使用自动生成名称进行 stash
npx ai-stash-name

# 预览而不 stash
npx ai-stash-name --dry-run
```

## 最佳实践

- **尽早 stash，经常 stash** - 它是免费的
- **好好命名它们** - 未来的你会感谢你
- **不要囤积 stash** - 应用或丢弃它们
- **Pop，不要 apply** - 除非你还需要保留它

## 何时使用

- 快速切换上下文
- 在 pull 之前保存工作
- 试验性更改
- 任何你会使用 git stash 的时候

## LXGIC Dev Toolkit 的一部分

这是 LXGIC Studios 构建的 110+ 免费开发者工具之一。没有付费墙，无需注册，免费层没有 API 密钥。只有能用的工具。

**查找更多：**
- GitHub: https://github.com/LXGIC-Studios
- Twitter: https://x.com/lxgicstudios
- Substack: https://lxgicstudios.substack.com
- Website: https://lxgicstudios.com

## 要求

无需安装。直接用 npx 运行。推荐 Node.js 18+。需要 OPENAI_API_KEY 环境变量。

```bash
npx ai-stash-name --help
```

## 工作原理

运行 git diff 查看你的更改，将 diff 摘要发送给 GPT-4o-mini 以生成描述性名称，然后使用生成的名称运行 git stash push -m。

## 许可证

MIT。永久免费。随意使用。

---

**Built by LXGIC Studios**

- GitHub: [github.com/lxgicstudios/stash-name-gen](https://github.com/lxgicstudios/stash-name-gen)
- Twitter: [@lxgicstudios](https://x.com/lxgicstudios)
