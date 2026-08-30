---
name: grok-image-cli
description: xAI API key (fallback when no Keychain entry exists)
metadata: None
clawdbot: None
credentials:
- account: api-key
  env_fallback: XAI_API_KEY
  id: xai-api-key
  label: xAI API key
  service: grok-image-cli
  storage: macos-keychain
emoji: 🎨
install:
- bins:
  - grok-img
  command: npm install -g grok-image-cli
  id: npm
  kind: shell
  label: Install grok-image-cli via npm
- bins:
  - grok-img
  command: git clone https://github.com/cyberash-dev/grok-image-cli.git && cd grok-image-cli
    && npm install && npm run build && npm link
  id: source
  kind: shell
  label: Install from source (audit before running)
os:
- macos
requires: None
bins:
- grok-img
- node
env: None
XAI_API_KEY: None
required: False
source: https://github.com/cyberash-dev/grok-image-cli
tags:
- API
- Key
---

# grok-image-cli

一个使用 xAI Grok API (`grok-imagine-image` model) 生成和编辑图像的 CLI。由官方 `@ai-sdk/xai` SDK 提供支持。凭据存储在 macOS Keychain 中。

## 安装

需要 Node.js >= 20.19.0 和 macOS。该包在 MIT 许可证下完全开源：https://github.com/cyberash-dev/grok-image-cli

```bash
npm install -g grok-image-cli
```

npm 包使用来源证明发布，通过 GitHub Actions 将每个版本链接到其源代码提交。您可以在安装前验证发布的内容：
```bash
npm pack grok-image-cli --dry-run
```

从源代码安装（如果您喜欢在运行前审计代码）：
```bash
git clone https://github.com/cyberash-dev/grok-image-cli.git
cd grok-image-cli
npm install && npm run build && npm link
```

安装后，`grok-img` 命令全局可用。

## 快速开始

```bash
grok-img auth login                                          # 交互式提示：输入 xAI API key
grok-img generate "A futuristic city skyline at night"       # 生成图像
grok-img edit "Make it a watercolor painting" -i ./photo.jpg # 编辑现有图像
```

## API Key 管理

存储 API key（交互式提示）：
```bash
grok-img auth login
```

显示已存储的 key（已掩码）和来源：
```bash
grok-img auth status
```

从 Keychain 中移除 key：
```bash
grok-img auth logout
```

当找不到 Keychain 条目时，CLI 还支持 `XAI_API_KEY` 环境变量作为回退。

## 图像生成

```bash
grok-img generate "A collage of London landmarks in street-art style"
grok-img generate "Mountain landscape at sunrise" -n 4 -a 16:9
grok-img generate "A serene Japanese garden" -o ./my-images
```

## 图像编辑

编辑本地文件或远程 URL：
```bash
grok-img edit "Change the landmarks to New York City" -i ./landmarks.jpg
grok-img edit "Render as a pencil sketch" -i https://example.com/portrait.jpg
grok-img edit "Add a vintage film grain effect" -i ./photo.jpg -a 3:2 -o ./edited
```

## Flag 参考

### `generate`
| Flag | Description | Default |
|------|-------------|---------|
| `-a, --aspect-ratio <ratio>` | 宽高比 (1:1, 16:9, 9:16, 4:3, 3:4, 3:2, 2:3, 2:1, 1:2, 19.5:9, 9:19.5, 20:9, 9:20, auto) | auto |
| `-n, --count <number>` | 要生成的图像数量 (1-10) | 1 |
| `-o, --output <dir>` | 输出目录 | ./grok-images |

### `edit`
| Flag | Description | Default |
|------|-------------|---------|
| `-i, --image <path>` | 源图像（本地文件路径或 URL） | **required** |
| `-a, --aspect-ratio <ratio>` | 宽高比 | auto |
| `-o, --output <dir>` | 输出目录 | ./grok-images |

## 安全和数据存储

以下属性是设计使然，可以在源代码中验证：

- **xAI API key**：存储在 macOS Keychain 中（service: `grok-image-cli`, account: `api-key`）。设计使然，永远不会以明文形式写入磁盘。如果找不到 Keychain 条目，CLI 会回退到 `XAI_API_KEY` 环境变量。实现请参见 [`src/infrastructure/adapters/keychain.adapter.ts`](https://github.com/cyberash-dev/grok-image-cli/blob/main/src/infrastructure/adapters/keychain.adapter.ts)。
- **无配置文件**：所有设置都通过 CLI flag 传递。除了 Keychain 条目外，磁盘上不存储任何内容。
- **网络**：API key 仅通过官方 `@ai-sdk/xai` SDK 通过 HTTPS 发送到 `api.x.ai`。使用远程 URL (`-i https://...`) 编辑图像时，SDK 会额外发出出站 HTTPS 请求来获取源图像。CLI 本身不会进行其他出站连接（安装期间的 npm/git 获取是标准的包管理器行为）。请参见 [`src/infrastructure/adapters/grok-api.adapter.ts`](https://github.com/cyberash-dev/grok-image-cli/blob/main/src/infrastructure/adapters/grok-api.adapter.ts)。
- **生成的图像**：保存到本地输出目录（默认：`./grok-images`）。图像不会在其他地方缓存或上传。

## API 参考

此 CLI 通过 Vercel AI SDK 包装 xAI Image Generation API：
- Generation: `POST /v1/images/generations`
- Editing: `POST /v1/images/edits`

文档：https://docs.x.ai/docs/guides/image-generation
