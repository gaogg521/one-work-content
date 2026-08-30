---
name: schema-exploration
description: 列出表、描述列和数据类型、识别外键关系并映射数据库中的实体关系。当用户询问数据库模式、表结构、列类型、存在哪些表、ERD、外键或实体如何关联时使用。
---

# 模式探索技能

## 工作流程

### 1. 列出所有表
使用 `sql_db_list_tables` 工具查看数据库中的所有可用表。

这将返回您可以查询的完整表列表。

### 2. 获取特定表的模式
使用 `sql_db_schema` 工具和表名来检查：
- **列名** - 有哪些字段可用
- **数据类型** - INTEGER、TEXT、DATETIME 等
- **示例数据** - 3 行示例以了解内容
- **主键** - 行的唯一标识符
- **外键** - 与其他表的关系

### 3. 映射关系
识别表的连接方式：
- 查找以 "Id" 结尾的列（例如 CustomerId、ArtistId）
- 外键链接到其他表的主键
- 记录父子关系

### 4. 回答问题
提供关于以下内容的清晰信息：
- 可用的表及其用途
- 列名及其包含的内容
- 表之间的关联方式
- 示例数据以说明内容

## 示例："有哪些表可用？"

**步骤 1：** 使用 `sql_db_list_tables`

**响应：**
```
Chinook 数据库有 11 个表：
1. Artist - 音乐艺术家
2. Album - 音乐专辑
3. Track - 单曲
4. Genre - 音乐流派
5. MediaType - 文件格式（MP3、AAC 等）
6. Playlist - 用户创建的播放列表
7. PlaylistTrack - 播放列表中的曲目
8. Customer - 商店客户
9. Employee - 商店员工
10. Invoice - 客户购买
11. InvoiceLine - 发票中的单个项目
```

## 示例："Customer 表有哪些列？"

**步骤 1：** 使用 `sql_db_schema` 工具和表名 "Customer"

**响应：**
```
Customer 表有以下列：

- CustomerId (INTEGER) - 主键，唯一客户 ID
- FirstName (NVARCHAR) - 客户名字
- LastName (NVARCHAR) - 客户姓氏
- Company (NVARCHAR) - 公司名称（可选）
- Address, City, State, Country, PostalCode - 位置字段
- Phone (NVARCHAR) - 电话号码
- Fax (NVARCHAR) - 传真号码
- Email (NVARCHAR) - 电子邮件地址
- SupportRepId (INTEGER) - 指向 Employee 表的外键

示例数据显示客户如：
- Luís Gonçalves 来自巴西
- Leonie Köhler 来自德国
- François Tremblay 来自加拿大
```

## 示例："如何按艺术家查找收入？"

**步骤 1：** 识别需要的表
- Artist（有艺术家名称）
- Album（将艺术家链接到曲目）
- Track（将专辑链接到销售）
- InvoiceLine（有销售数据）
- Invoice（有收入总计）

**步骤 2：** 映射关系
```
Artist (ArtistId)
  ↓ 1:多
Album (ArtistId, AlbumId)
  ↓ 1:多
Track (AlbumId, TrackId)
  ↓ 1:多
InvoiceLine (TrackId, UnitPrice, Quantity)
```

**响应：**
```
要按艺术家查找收入，您需要：
1. 从 Artist 表开始
2. 在 ArtistId 上 JOIN 到 Album
3. 在 AlbumId 上 JOIN 到 Track
4. 在 TrackId 上 JOIN 到 InvoiceLine
5. SUM 收入（UnitPrice * Quantity）
6. 按艺术家名称 GROUP BY

这需要查询编写技能来执行。
```

## 质量指南

**对于"列出表"问题：**
- 显示所有表名
- 添加每个表包含内容的简要描述
- 将相关表分组（例如，音乐目录、交易、人员）

**对于"描述表"问题：**
- 列出所有列及其数据类型
- 解释每个列包含的内容
- 显示示例数据以提供上下文
- 注意主键和外键
- 解释与其他表的关系

**对于"如何查询 X"问题：**
- 识别所需的表
- 映射 JOIN 路径
- 解释关系链
- 建议下一步（使用查询编写技能）
