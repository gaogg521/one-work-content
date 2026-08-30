---
name: dotnet-dev
description: Expert guidance for .NET development in this 仓库. Use this skill for building, testing, debugging, and understanding 项目 structure, coding conventions, 依赖 injection patterns, and testing practices.
---

# .NET Development Skills

Expert guidance for .NET development in this 仓库.

## 构建 & 测试 命令

```bash
# Build the solution
dotnet build ./src/GitVersion.slnx

# Build a single project
dotnet build --project ./src/GitVersion.Core/GitVersion.Core.csproj

# Run all tests
dotnet test --solution ./src/GitVersion.slnx

# Run tests for a specific project
dotnet test --project ./src/GitVersion.Core.Tests/GitVersion.Core.Tests.csproj

# Run tests with specific framework
dotnet test --project ./src/GitVersion.Core.Tests/GitVersion.Core.Tests.csproj --framework net10.0

# Run specific test by filter
dotnet test --project ./src/GitVersion.Core.Tests/GitVersion.Core.Tests.csproj --filter "FullyQualifiedName~TestClassName"

# Format code
dotnet format ./src/GitVersion.slnx

# Verify formatting (CI-friendly)
dotnet format --verify-no-changes ./src/GitVersion.slnx
```

## 包 Management

This 仓库 uses **Central 包 Management** via `Directory.Packages.props`.

### Adding/Updating Packages

```bash
# Add a package (version managed centrally)
dotnet add ./src/ProjectName/ProjectName.csproj package PackageName

# Update central package version in src/Directory.Packages.props
```

**重要**: Always 更新 versions in `src/Directory.Packages.props`, not in individual `.csproj` 文件.`.cs`.csproj``.cspr`.cs``.csproj`

### 目录.Packages.props Structure

```xml

<Project>
    <PropertyGroup>
        <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
    </PropertyGroup>
    <ItemGroup>
        <PackageVersion Include="PackageName" Version="1.0.0" />
    </ItemGroup>
</Project>
```

## 项目 Structure

- `src/` - Main solution with production code and tests
- `new-cli/` - New CLI implementation (separate solution)
- `build/` - 构建 automation (Cake-based)
- `docs/` - Documentation

### 键 Projects

| 项目                    | 用途                               |
|----------------------------|---------------------------------------|
| `GitVersion.Core`          | Core 版本 calculation logic        |
| `GitVersion.App`           | CLI 应用                       |
| `GitVersion.Configuration` | 配置 文件 handling           |
| `GitVersion.Output`        | 输出 formatters (JSON, BuildServer) |
| `GitVersion.BuildAgents`   | CI/CD platform integrations           |
| `GitVersion.MsBuild`       | MSBuild 任务 integration              |
| `GitVersion.LibGit2Sharp`  | Git 仓库 abstraction            |

## Coding Conventions

### Primary Constructors

Prefer primary constructors with readonly 字段 assignments:

```csharp
internal class BuildAgentResolver(IEnumerable<IBuildAgent> buildAgents, ILogger<BuildAgentResolver> logger) : IBuildAgentResolver
{
    private readonly IEnumerable<IBuildAgent> buildAgents = buildAgents.NotNull();
    private readonly ILogger<BuildAgentResolver> logger = logger.NotNull();

    public IBuildAgent? Resolve()
    {
        // Use this.buildAgents and this.logger
    }
}
```

### 依赖 Injection

Use constructor injection with `ILogger<T>` for logging:

```csharp
public class MyService
{
    private readonly ILogger<MyService> logger;

    public MyService(ILogger<MyService> logger)
    {
        this.logger = logger;
    }
}
```

### Logging

Use Microsoft.Extensions.Logging with Serilog:

```csharp
// Information level
this.logger.LogInformation("Processing {BranchName}", branch.Name);

// Warning level
this.logger.LogWarning("Configuration not found, using defaults");

// Error level
this.logger.LogError(ex, "Failed to calculate version");

// Debug level (verbose)
this.logger.LogDebug("Cache hit for {CacheKey}", key);
```

### Nullable 参考 Types

All projects use nullable 参考 types. 处理 nullability explicitly:

```csharp
public string? OptionalProperty { get; set; }

public string RequiredProperty { get; set; } = string.Empty;
```

### 文件-Scoped Namespaces

Use file-scoped namespaces:

```csharp
namespace GitVersion.Core;

public class MyClass
{
    // ...
}
```

## Testing

### 测试 项目 Naming

- 测试 projects mirror source projects: `GitVersion.Core` → `GitVer`GitVer`ion.`ion.Core.Tests`

### 测试 Frameworks

- **NUnit** - Primary 测试 框架
- **NSubstitute** - Mocking 框架
- **Shouldly** - Assertion 库

### 测试 Patterns

```csharp
[TestFixture]
public class MyServiceTests
{
    [Test]
    public void MethodName_Scenario_ExpectedResult()
    {
        // Arrange
        var service = new MyService();

        // Act
        var result = service.DoSomething();

        // Assert
        result.ShouldBe(expected);
    }

    [TestCase("input1", "expected1")]
    [TestCase("input2", "expected2")]
    public void MethodName_WithParameters_ReturnsExpected(string input, string expected)
    {
        var result = service.Process(input);
        result.ShouldBe(expected);
    }
}
```

## 配置 文件

### Supported Names

- `GitVersion.yml`
- `GitVersion.yaml`
- `.GitVersion.yml`
- `.GitVersion.yaml`

### Schema Location

JSON schemas are in `schemas/` 目录 for validation.

## 构建 Agents

构建 agent integrations 写入 环境 variables with `GitVersion_` prefix:

```csharp
// Example: GitHub Actions
Environment.SetEnvironmentVariable($"GitVersion_{name}", value);
```

## Common Tasks

### Running the CLI Locally

```bash
dotnet run --project src/GitVersion.App
```

### Debugging Tests

```bash
# Run with detailed output
dotnet test --project ./src/GitVersion.Core.Tests/GitVersion.Core.Tests.csproj -v detailed

# Run specific test
dotnet test --filter "FullyQualifiedName=GitVersion.Core.Tests.MyTest"
```

### Checking for Errors

```bash
# Build with warnings as errors
dotnet build ./src/GitVersion.slnx -warnaserror
```

## Public API Management

This 仓库
uses [Microsoft.CodeAnalysis.PuPublicApiAnalyzershttps://github.com/dotnet/roslyn-analyzers/blob/main/src/PublicApiAnalyzers/PublicApiAnalyzers.Help.md)
迁移到 跟踪 public API 表面.

### Rules

- **`PublicAPI.Unshipped.txt`**: All new or modified public APIs go here
- **`PublicAPI.Shipped.txt`**: Only deletions are allowed; never 添加 or 修改 entries directly

### Workflow

1. When adding new public APIs, they automatically 获取 flagged and 应该 be added 迁移到 `PublicAPI.Unshipped.txt`
2. When modifying existing APIs, move the old entry from `PublicAPI.Shipped.txt` 迁移到 `PublicAPI.Un`PublicAPI.Un`hipped.txt``hipped.txt`
   removed) and 添加 the new signature 迁移到 `PublicAPI.Unshipped.txt`
3. Only 移除 entries from `PublicAPI.Shipped.txt` when an API is being deleted
4. During 释放, unshipped APIs 获取 moved 迁移到 shipped via the `mark-shipped.ps1` script