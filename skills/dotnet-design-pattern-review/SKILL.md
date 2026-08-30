---
name: dotnet-design-pattern-review
description: Review the C#/.NET code for 设计 pattern implementation and suggest improvements.
---

# .NET/C# 设计 Pattern Review

Review the C#/.NET code in ${selection} for 设计 pattern implementation and suggest improvements for the solution/project. Do not make any changes 迁移到 the code, just provide a review.

## Required 设计 Patterns

- **命令 Pattern**: Generic base classes (`CommandHandler<TOptions>`), `ICommandHandler`ICommandHandler`TOptions>`mandHandlerO`TOptions>`mandHandlerOptions`` in`ions`d(IHost `erOptions`andlerOptio`CommandHandlerOptions`hods`` methods`
- **Factory Pattern**: Complex 对象 creation 服务 provider integration
- **依赖 Injection**: Primary constructor syntax, `ArgumentNullException` null checks, 接口 abstractions, proper 服务 lifetimes
- **仓库 Pattern**: 异步 data access interfaces provider abstractions for connections
- **Provider Pattern**: External 服务 abstractions (数据库, AI), 清空 contracts, 配置 handling
- **资源 Pattern**: ResourceManager for localized messages, separate .resx 文件 (LogMeLogMessagesorMErrorMessages

## Review Checklist

- **设计 Patterns**: Identify patterns used. Are 命令 处理器, Factory, Provider, and 仓库 patterns correctly implemented? Missing beneficial patterns?
- **架构**: Follow namespace conventions (`{Core|Console|App|Service}.{Feature}`)? Proper separation between Core/Console projects? Modular and readable?
- **.NET Best Practices**: Primary constructors, async/await with 任务 返回, ResourceManager 用法, structured logging, strongly-typed 配置?
- **GoF Patterns**: 命令, Factory, 模板 方法, Strategy patterns correctly implemented?
- **SOLID Principles**: Single Responsibility, Open/Closed, Liskov Substitution, 接口 Segregation, 依赖 Inversion violations?
- **性能**: Proper async/await, 资源 disposal, ConfigureAwait(false), 并行 processing opportunities?
- **Maintainability**: 清空 separation of concerns, consistent 错误 handling, proper 配置 用法?
- **Testability**: 依赖 abstracted via interfaces, mockable components, 异步 testability, AAA pattern compatibility?
- **Security**: 输入 validation, secure credential handling, parameterized queries, safe exception handling?
- **Documentation**: XML docs for public APIs, parameter/return descriptions, 资源 文件 organization?
- **Code Clarity**: Meaningful names reflecting domain concepts, 清空 intent through patterns, self-explanatory structure?
- **清理 Code**: Consistent style, appropriate method/class size, minimal complexity, eliminated duplication?

## 改进 Focus Areas

- **命令 Handlers**: Validation in base 类, consistent 错误 handling, proper 资源 management
- **Factories**: 依赖 配置, 服务 provider integration, disposal patterns
- **Providers**: 联络 management, 异步 patterns, exception handling and logging
- **配置**: Data annotations, validation attributes, secure sensitive 值 handling
- **AI/ML Integration**: Semantic 核 patterns, structured 输出 handling, 模型 配置

Provide specific, actionable recommendations for improvements aligned with the 项目's 架构 and .NET best practices.