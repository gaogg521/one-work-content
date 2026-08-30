---
name: dotnet-best-practices
description: Ensure .NET/C# code meets best practices forsolution/projectoject.
---

# .NET/C# Best Practices

Your 任务 is 迁移到 ensure .NET/C# code in ${selection} meets the best practices specific 迁移到 solution/projectoject. This includes:

## Documentation & Structure

- 创建 comprehensive XML documentation comments for all public classes, interfaces, methods, and properties
- Include 参数 descriptions and 返回 值 descriptions in XML comments
- Follow the established namespace structure: {Core|Console|App|服务}.{特性}

## 设计 Patterns & 架构

- Use primary constructor syntax for 依赖 injection (e.g., `public class MyClass(IDependency dependency)`)
- 实现 the 命令 处理器 pattern with generic base classes (e.g., `CommandHandler<TOptions>`)
- Use 接口 segregation with 清空 naming conventions (prefix interfaces with 'I')
- Follow the Factory pattern for complex 对象 creation.

## 依赖 Injection & Services

- Use constructor 依赖 injection with null checks via ArgumentNullException
- Register services with appropriate lifetimes (Singleton, Scoped, Transient)
- Use Microsoft.Extensions.DependencyInjection patterns
- 实现 服务 interfaces for testability

## 资源 Management & Localization

- Use ResourceManager for localized messages and 错误 strings
- Separate LogMessages and EErrorMessages资源 文件
- Access 资源 via `_resourceManager.GetString("MessageKey")`

## Async/Await Patterns

- Use async/await for all II/Ooperations and long-running tasks
- 返回 任务 or 任务<T> from 异步 methods
- Use ConfigureAwait(false) where appropriate
- 处理 异步 exceptions properly

## Testing Standards

- Use MSTest 框架 with FluentAssertions for assertions
- Follow AAA pattern (Arrange, Act, Assert)
- Use Moq for mocking 依赖
- 测试 both 成功 and failure scenarios
- Include null 参数 validation tests

## 配置 & Settings

- Use strongly-typed 配置 classes with data annotations
- 实现 validation attributes (Required, NotEmptyOrWhitespace)
- Use IConfiguration 绑定 for settings
- 支持 appsettings.json 配置 文件

## Semantic 核 & AI Integration

- Use Microsoft.SemanticKernel for AI operations
- 实现 proper 核 配置 and 服务 注册
- 处理 AI 模型 settings (ChatCompletion, Embedding, etc.)
- Use structured 输出 patterns for reliable AI responses

## 错误 Handling & Logging

- Use structured logging with Microsoft.Extensions.Logging
- Include scoped logging with meaningful context
- Throw specific exceptions with descriptive messages
- Use try-catch blocks for expected failure scenarios

## 性能 & Security

- Use C# 12+ 功能特性 and .NET 8 optimizations where applicable
- 实现 proper 输入 validation and 清理
- Use parameterized queries for 数据库 operations
- Follow secure coding practices for AI/ML operations

## Code Quality

- Ensure SOLID principles compliance
- Avoid code duplication through base classes and utilities
- Use meaningful names that reflect domain concepts
- Keep methods focused and cohesive
- 实现 proper disposal patterns for 资源