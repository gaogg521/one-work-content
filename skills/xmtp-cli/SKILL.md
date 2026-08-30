---
name: xmtp-cli
description: 运行和编写 XMTP CLI 脚本进行测试、调试，并与 XMTP 对话、群组和消息交互。支持 init、send、list、groups、debug、sync、permissions 和 content 等命令。
license: MIT
metadata:
  author: xmtp
  version: 1.0.0
tags:
- 即时通讯
- CLI
---

# XMTP CLI

使用 `xmtp` 命令从命令行测试、调试和与 XMTP 对话、群组和消息交互。此技能是入口点；使用下面的子技能获取特定 CLI 任务。

## 何时应用

- 从命令行测试或调试 XMTP
- 发送消息或创建和管理群组
- 列出或查找对话、成员和消息
- 同步对话和消息
- 管理群组权限
- 演示内容类型（文本、markdown、附件、交易、深度链接、迷你应用）

## 子技能

| 子技能 | 使用场景 |
|-----------|----------|
| **setup** | 初始化 CLI 和配置环境（init、环境变量） |
| **groups** | 创建 DM 或群组、更新群组元数据 |
| **send** | 向地址或群组发送消息 |
| **list** | 列出对话、成员、消息；按地址或收件箱查找 |
| **debug** | 获取信息、解析地址、检查收件箱 |
| **sync** | 同步对话或同步所有 |
| **permissions** | 列出/信息群组权限、更新权限 |
| **content** | 演示文本、markdown、附件、交易、深度链接、迷你应用 |
| **debugging** | 启用 CLI 调试日志记录（XMTP_FORCE_DEBUG 环境变量） |

## 如何使用

1. 选择与任务匹配的子技能（例如发送消息 → **send**）。
2. 阅读该子技能的 `SKILL.md` 及其 `rules/` 以获取分步指南。

## 安装

```bash
npm install -g @xmtp/cli
# 或
pnpm add -g @xmtp/cli
# 或
yarn global add @xmtp/cli
```

## 无需安装运行

```bash
npx @xmtp/cli <command> <arguments>
# 或 pnpx / yarn dlx
```

## 帮助

```bash
xmtp --help
xmtp <command> --help
```

完整文档：[docs.xmtp.org](https://docs.xmtp.org)
