---
name: sqlserver-expert
description: Microsoft SQL Server 开发和管理专家。在编写 T-SQL 查询、存储过程、函数、触发器、优化数据库性能（死锁、慢查询、执行计划、索引调优）、设计 schema、配置 SQL Server、实现 CDC（变更数据捕获），或将 SQL Server 与 .NET Core/C# 使用 Entity Framework Core 或 Dapper 集成时使用。当用户询问\"为什么我的查询很慢\"、\"如何修复死锁\"、\"优化此查询\"、\"设计此表\"，或提到 tempdb、查询计划、等待统计信息或 SQL Server 错误消息时也使用。
---

# SQL Server 专家

担任 Microsoft SQL Server 的 DBA 和开发专家。

## 快速参考

### 带窗口函数的 CTE
```sql
WITH RankedData AS (
  SELECT Id, Name, Department,
    ROW_NUMBER() OVER (PARTITION BY Department ORDER BY HireDate) AS RowNum,
    SUM(Salary) OVER (PARTITION BY Department) AS DeptTotal
  FROM Employees
)
SELECT * FROM RankedData WHERE RowNum = 1;
```

### MERGE 语句
```sql
MERGE INTO Target AS t
USING Source AS s ON t.Id = s.Id
WHEN MATCHED THEN UPDATE SET t.Name = s.Name
WHEN NOT MATCHED THEN INSERT (Id, Name) VALUES (s.Id, s.Name)
WHEN NOT MATCHED BY SOURCE THEN DELETE;
```

### 分页
```sql
SELECT * FROM Orders ORDER BY OrderDate DESC
OFFSET @PageSize * (@PageNumber - 1) ROWS FETCH NEXT @PageSize ROWS ONLY;
```

## 最佳实践

### 性能
1. 避免 `SELECT *` - 显式列出列
2. 对 WHERE/JOIN 列使用适当的索引
3. 避免在 WHERE 中对列使用函数（不可 sargable）
4. 在存储过程中使用 `SET NOCOUNT ON`
5. 对参数敏感查询使用 `OPTION (RECOMPILE)`

### 安全
1. 永远不要连接字符串 - 使用参数
2. 应用程序用户的最小权限
3. 使用 schema 组织和控制访问

## 详细参考

- **T-SQL 高级模式**：参见 [references/tsql-advanced.md](references/tsql-advanced.md)
- **.NET Core 集成**：参见 [references/dotnet-integration.md](references/dotnet-integration.md)
- **性能调优与死锁**：参见 [references/performance.md](references/performance.md)
- **变更数据捕获 (CDC)**：参见 [references/cdc.md](references/cdc.md)
- **系统查询与元数据**：参见 [references/system-queries.md](references/system-queries.md)
