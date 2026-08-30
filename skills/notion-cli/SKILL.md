---
name: notion-cli
description: 用于创建和管理页面、数据库和块的 Notion CLI。
homepage: https://github.com/litencatt/notion-cli
metadata:
  openclaw:
    emoji: 📓
    requires:
      env:
      - NOTION_TOKEN
    primaryEnv: NOTION_TOKEN
tags:
- CLI
- 办公
- 数据
- 管理
---

# notion

使用 *notion-cli* 创建/读取/更新页面、数据源（数据库）和块。

## 设置

- 安装 notion-cli：`npm install -g @iansinnott/notion-cli`
- 在 https://notion.so/my-integrations 创建一个集成
- 复制 API 密钥（以 *ntn_* 或 *secret_* 开头）
- 存储它：
  - `mkdir -p ~/.config/notion`
  - `echo "ntn_your_key_here" > ~/.config/notion/api_key`
- 与你的集成共享目标页面/数据库（点击 "..." → "Connect to" → 你的集成名称）

## 用法

所有命令都需要设置 *NOTION_TOKEN* 环境变量：

```bash
export NOTION_TOKEN=$(cat ~/.config/notion/api_key)
```

## 常见操作

- **搜索页面和数据源：**

  `notion-cli search --query "page title"`

- **获取页面：**

  `notion-cli page retrieve <PAGE_ID>`

- **获取页面内容（块）：**

  `notion-cli page retrieve <PAGE_ID> -r`

- **在数据库中创建页面：**

  ```bash
  curl -X POST https://api.notion.com/v1/pages \
    -H "Authorization: Bearer $NOTION_TOKEN" \
    -H "Content-Type: application/json" \
    -H "Notion-Version: 2025-09-03" \
    --data '{
      "parent": { "database_id": "YOUR_DATABASE_ID" },
      "properties": {
        "Name": {
          "title": [
            {
              "text": {
                "content": "Nouvelle idée"
              }
            }
          ]
        }
      }
    }'
  ```

- **查询数据库：**

  `notion-cli db query <DB_ID> -a '{"property":"Status","status":{"equals":"Active"}}'`

- **更新页面属性：**

  ```bash
  curl -X PATCH https://api.notion.com/v1/pages/PAGE_ID \
    -H "Authorization: Bearer $NOTION_TOKEN" \
    -H "Content-Type: application/json" \
    -H "Notion-Version: 2025-09-03" \
    --data '{
      "properties": {
        "Name": {
          "title": [
            {
              "text": {
                "content": "Nouveau titre"
              }
            }
          ]
        },
        "Status": {
          "status": {
            "name": "In progress"
          }
        },
        "Priority": {
          "select": {
            "name": "High"
          }
        },
        "Due date": {
          "date": {
            "start": "2026-02-10"
          }
        },
        "Description": {
          "rich_text": [
            {
              "text": {
                "content": "Description mise à jour"
              }
            }
          ]
        }
      }
    }'
  ```

- **获取数据库信息：**

  `notion-cli db retrieve <DB_ID>`

## 属性类型

数据库项的常见属性格式：

- **Title:** `{"title": [{"text": {"content": "..."}}]}`
- **Rich text:** `{"rich_text": [{"text": {"content": "..."}}]}`
- **Status:** `{"status": {"name": "Option"}}`
- **Select:** `{"select": {"name": "Option"}}`
- **Multi-select:** `{"multi_select": [{"name": "A"}, {"name": "B"}]}`
- **Date:** `{"date": {"start": "2024-01-15", "end": "2024-01-16"}}`
- **Checkbox:** `{"checkbox": true}`
- **Number:** `{"number": 42}`
- **URL:** `{"url": "https://..."}`
- **Email:** `{"email": "a@b.com"}`

## 示例

- **搜索页面：**

  `notion-cli search --query "AIStories"`

- **使用过滤器查询数据库：**

  ```bash
  notion-cli db query 2faf172c094981d3bbcbe0f115457cda \
    -a '{
      "property": "Status",
      "status": { "equals": "Backlog" }
    }'
  ```

- **检索页面内容：**

  `notion-cli page retrieve 2fdf172c-0949-80dd-b83b-c1df0410d91b -r`

- **更新页面状态：**

  ```bash
  curl -X PATCH https://api.notion.com/v1/pages/2fdf172c-0949-80dd-b83b-c1df0410d91b \
    -H "Authorization: Bearer $NOTION_TOKEN" \
    -H "Content-Type: application/json" \
    -H "Notion-Version: 2025-09-03" \
    --data '{
      "properties": {
        "Status": {
          "status": {
            "name": "In progress"
          }
        }
      }
    }'
  ```

## 主要功能

- *交互模式:* 对于复杂查询，运行 `notion-cli db query <DB_ID>` 不带参数以进入交互模式
- *多种输出格式:* table (默认), csv, json, yaml
- *原始 JSON:* 使用 `--raw` 标志获取完整的 API 响应
- *过滤器语法:* 使用 `-a` 标志进行带有 AND/OR 条件的复杂过滤器

## 注意事项

- 页面/数据库 ID 是 UUID（带或不带破折号）
- CLI 通过 *NOTION_TOKEN* 自动处理身份验证
- 速率限制由 CLI 管理
- 使用 `notion-cli help` 获取完整的命令参考

## 参考

- GitHub Notion-CLI: https://github.com/litencatt/notion-cli
- Notion API 文档: https://developers.notion.com
