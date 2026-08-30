---
name: abp-mongodb
description: ABP MongoDB 模式 - AbpMongoDbContext、IMongoCollection、MongoDbRepository，无需迁移，嵌入式文档与引用对比，需要手动调用 UpdateAsync。在 MongoDB 项目中工作或实现 ABP 的 MongoDB 仓库时使用。
---

# ABP MongoDB

> **文档**: https://abp.io/docs/latest/framework/data/mongodb

## MongoDbContext 配置

```csharp
[ConnectionStringName("Default")]
public class MyProjectMongoDbContext : AbpMongoDbContext
{
    public IMongoCollection<Book> Books => Collection<Book>();
    public IMongoCollection<Author> Authors => Collection<Author>();

    protected override void CreateModel(IMongoModelBuilder modelBuilder)
    {
        base.CreateModel(modelBuilder);

        modelBuilder.ConfigureMyProject();
    }
}
```

## 实体配置

```csharp
public static class MyProjectMongoDbContextExtensions
{
    public static void ConfigureMyProject(this IMongoModelBuilder builder)
    {
        Check.NotNull(builder, nameof(builder));

        builder.Entity<Book>(b =>
        {
            b.CollectionName = MyProjectConsts.DbTablePrefix + "Books";
        });

        builder.Entity<Author>(b =>
        {
            b.CollectionName = MyProjectConsts.DbTablePrefix + "Authors";
        });
    }
}
```

## 仓库实现

```csharp
public class BookRepository : MongoDbRepository<MyProjectMongoDbContext, Book, Guid>, IBookRepository
{
    public BookRepository(IMongoDbContextProvider<MyProjectMongoDbContext> dbContextProvider)
        : base(dbContextProvider)
    {
    }

    public async Task<Book> FindByNameAsync(
        string name,
        bool includeDetails = true,
        CancellationToken cancellationToken = default)
    {
        return await (await GetQueryableAsync())
            .FirstOrDefaultAsync(
                b => b.Name == name,
                GetCancellationToken(cancellationToken));
    }

    public async Task<List<Book>> GetListByAuthorAsync(
        Guid authorId,
        bool includeDetails = false,
        CancellationToken cancellationToken = default)
    {
        return await (await GetQueryableAsync())
            .Where(b => b.AuthorId == authorId)
            .ToListAsync(GetCancellationToken(cancellationToken));
    }
}
```

## 模块配置

```csharp
[DependsOn(typeof(AbpMongoDbModule))]
public class MyProjectMongoDbModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddMongoDbContext<MyProjectMongoDbContext>(options =>
        {
            // 仅为聚合根添加默认仓库（DDD 最佳实践）
            options.AddDefaultRepositories();
            // 避免使用 includeAllEntities: true - 会破坏 DDD 数据一致性
        });
    }
}
```

## 连接字符串

在 `appsettings.json` 中：
```json
{
  "ConnectionStrings": {
    "Default": "mongodb://localhost:27017/MyProjectDb"
  }
}
```

## 与 EF Core 的关键区别

### 无需迁移
MongoDB 是无模式的；不需要迁移。实体结构的更改会自动处理。

### includeDetails 参数
在 MongoDB 中通常被忽略，因为文档通常嵌入了相关数据：

```csharp
public async Task<List<Book>> GetListAsync(
    bool includeDetails = false, // 通常被忽略
    CancellationToken cancellationToken = default)
{
    // MongoDB 文档已包含嵌套数据
    return await (await GetQueryableAsync())
        .ToListAsync(GetCancellationToken(cancellationToken));
}
```

### 嵌入式文档与引用
```csharp
// 嵌入式（存储在同一文档中）
public class Order : AggregateRoot<Guid>
{
    public List<OrderLine> Lines { get; set; } // 嵌入式
}

// 引用（单独集合，仅存储 ID）
public class Order : AggregateRoot<Guid>
{
    public Guid CustomerId { get; set; } // 通过 ID 引用
}
```

### 无变更追踪
MongoDB 不会自动追踪实体更改：

```csharp
public async Task UpdateBookAsync(Guid id, string newName)
{
    var book = await _bookRepository.GetAsync(id);
    book.SetName(newName);

    // 必须显式更新
    await _bookRepository.UpdateAsync(book);
}
```

## 直接集合访问

```csharp
public async Task CustomOperationAsync()
{
    var collection = await GetCollectionAsync();

    // 直接使用 MongoDB 驱动
    var filter = Builders<Book>.Filter.Eq(b => b.AuthorId, authorId);
    var update = Builders<Book>.Update.Set(b => b.IsPublished, true);

    await collection.UpdateManyAsync(filter, update);
}
```

## 索引

在仓库中或通过 MongoDB 驱动配置索引：

```csharp
public class BookRepository : MongoDbRepository<MyProjectMongoDbContext, Book, Guid>, IBookRepository
{
    public override async Task<IQueryable<Book>> GetQueryableAsync()
    {
        var collection = await GetCollectionAsync();

        // 确保索引存在
        var indexKeys = Builders<Book>.IndexKeys.Ascending(b => b.Name);
        await collection.Indexes.CreateOneAsync(new CreateIndexModel<Book>(indexKeys));

        return await base.GetQueryableAsync();
    }
}
```

## 最佳实践

- 根据查询模式设计文档（需要时进行反规范化）
- 对频繁变化的数据使用引用
- 对总是一起访问的数据使用嵌入
- 为频繁查询的字段添加索引
- 使用 `GetCancellationToken(cancellationToken)` 实现正确的取消
- 记住：ABP 数据过滤器（软删除、多租户）也适用于 MongoDB
