# One Work 官方内容源

企业版「内容市场」的官方技能 / MCP 源。管理控制台 → 资源管理 → 内容市场 → 添加来源，
填入本仓库 `index.json` 的 raw URL，例如：

```
https://raw.githubusercontent.com/<your-org>/one-work-content/main/index.json
```

添加后在同一页点「同步」，技能和 MCP 条目就进入本租户的注册表（`origin='market'`、
默认已发布）。同步是幂等的：再次同步只拉取变更条目（按 SHA-256 比对）。

## 信任模型

同步等同于「管理员手动上传」——没有导入期沙箱，导入的内容是数据，运行时由
SendGate / 终端工具安全门 / 内容检查 / DLP 全套治理。技能正文经过与上传路径
**完全相同**的 frontmatter 校验器。

## 布局

```
index.json                     清单（version 1）
skills/<slug>/SKILL.md          每个技能一个目录，frontmatter 的 name 必须与清单 name 一致
```

## 增加一个技能

1. `skills/<slug>/SKILL.md`，开头是 YAML frontmatter：
   ```
   ---
   name: <与清单 name 完全一致>
   description: <一句话，会显示在市场列表>
   ---

   <正文：给 agent 的指令>
   ```
2. `index.json` 的 `items` 加一项 `{ "kind": "skill", "name": "<name>", "path": "skills/<slug>/SKILL.md" }`。
3. （可选）算好 `sha256` 填进条目 —— 匹配则同步时免拉取，不匹配则判为篡改并报错。

## 增加一个 MCP

`index.json` 加 `{ "kind": "mcp", "name": "<name>", "mcp": { "type": "sse", "endpoint": "https://…" } }`。
清单不分发凭证 —— 密钥是每部署自行配置的。
