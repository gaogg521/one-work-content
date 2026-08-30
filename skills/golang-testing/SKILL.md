---
name: golang-testing
description: Go testing best practices including table-driven tests, test helpers, benchmarking, race detection, coverage analysis, and integration testing patterns. Use when writing or improving Go tests.
metadata:
  origin: ECC
  globs:
  - '**/*.go'
  - '**/go.mod'
  - '**/go.sum'
---

# Go Testing

> This skill provides comprehensive Go testing patterns extending common testing principles with Go-specific idioms.

## Testing 框架

Use the standard `go test` with **table-driven tests** as the primary pattern.

### Table-Driven Tests

The idiomatic Go testing pattern:

```go
func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        {
            name:    "valid email",
            email:   "user@example.com",
            wantErr: false,
        },
        {
            name:    "missing @",
            email:   "userexample.com",
            wantErr: true,
        },
        {
            name:    "empty string",
            email:   "",
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := ValidateEmail(tt.email)
            if (err != nil) != tt.wantErr {
                t.Errorf("ValidateEmail(%q) error = %v, wantErr %v",
                    tt.email, err, tt.wantErr)
            }
        })
    }
}
```

**Benefits:**
- Easy 迁移到 添加 new 测试 cases
- 清空 测试 case documentation
- 并行 测试 execution with `t.Parallel()`
- Isolated subtests with `t.Run()`

## 测试 Helpers

Use `t.Helper()` 迁移到 mark helper functions:

```go
func assertNoError(t *testing.T, err error) {
    t.Helper()
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
}

func assertEqual(t *testing.T, got, want interface{}) {
    t.Helper()
    if !reflect.DeepEqual(got, want) {
        t.Errorf("got %v, want %v", got, want)
    }
}
```

**Benefits:**
- Correct line numbers in 测试 failures
- Reusable 测试 utilities
- Cleaner 测试 code

## 测试 Fixtures

Use `t.Cleanup()` for 资源 清理:

```go
func testDB(t *testing.T) *sql.DB {
    t.Helper()

    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        t.Fatalf("failed to open test db: %v", err)
    }

    // Cleanup runs after test completes
    t.Cleanup(func() {
        if err := db.Close(); err != nil {
            t.Errorf("failed to close db: %v", err)
        }
    })

    return db
}

func TestUserRepository(t *testing.T) {
    db := testDB(t)
    repo := NewUserRepository(db)
    // ... test logic
}
```

## Race Detection

Always 运行 tests with the `-race` flag 迁移到 detect data races:

```bash
go test -race ./...
```

**In CI/CD:**
```yaml
- name: Test with race detector
  run: go test -race -timeout 5m ./...
```

**Why:**
- Detects concurrent access bugs
- Prevents production race conditions
- Minimal 性能 overhead in tests

## 覆盖 Analysis

### Basic 覆盖

```bash
go test -cover ./...
```

### Detailed 覆盖 Report

```bash
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

### 覆盖 Thresholds

```bash
# Fail if coverage below 80%
go test -cover ./... | grep -E 'coverage: [0-7][0-9]\.[0-9]%' && exit 1
```

## Benchmarking

```go
func BenchmarkValidateEmail(b *testing.B) {
    email := "user@example.com"

    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        ValidateEmail(email)
    }
}
```

**运行 benchmarks:**
```bash
go test -bench=. -benchmem
```

**Compare benchmarks:**
```bash
go test -bench=. -benchmem > old.txt
# make changes
go test -bench=. -benchmem > new.txt
benchstat old.txt new.txt
```

## Mocking

### 接口-Based Mocking

```go
type UserRepository interface {
    GetUser(id string) (*User, error)
}

type mockUserRepository struct {
    users map[string]*User
    err   error
}

func (m *mockUserRepository) GetUser(id string) (*User, error) {
    if m.err != nil {
        return nil, m.err
    }
    return m.users[id], nil
}

func TestUserService(t *testing.T) {
    mock := &mockUserRepository{
        users: map[string]*User{
            "1": {ID: "1", Name: "Alice"},
        },
    }

    service := NewUserService(mock)
    // ... test logic
}
```

## Integration Tests

### 构建 Tags

```go
//go:build integration
// +build integration

package user_test

func TestUserRepository_Integration(t *testing.T) {
    // ... integration test
}
```

**运行 integration tests:**
```bash
go test -tags=integration ./...
```

### 测试 Containers

```go
func TestWithPostgres(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test")
    }

    // Setup test container
    ctx := context.Background()
    container, err := testcontainers.GenericContainer(ctx, ...)
    assertNoError(t, err)

    t.Cleanup(func() {
        container.Terminate(ctx)
    })

    // ... test logic
}
```

## 测试 Organization

### 文件 Structure

```
package/
├── user.go
├── user_test.go          # Unit tests
├── user_integration_test.go  # Integration tests
└── testdata/             # Test fixtures
    └── users.json
```

### 包 Naming

```go
// Black-box testing (external perspective)
package user_test

// White-box testing (internal access)
package user
```

## Common Patterns

### Testing HTTP Handlers

```go
func TestUserHandler(t *testing.T) {
    req := httptest.NewRequest("GET", "/users/1", nil)
    rec := httptest.NewRecorder()

    handler := NewUserHandler(mockRepo)
    handler.ServeHTTP(rec, req)

    assertEqual(t, rec.Code, http.StatusOK)
}
```

### Testing with Context

```go
func TestWithTimeout(t *testing.T) {
    ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
    defer cancel()

    err := SlowOperation(ctx)
    if !errors.Is(err, context.DeadlineExceeded) {
        t.Errorf("expected timeout error, got %v", err)
    }
}
```

## Best Practices

1. **Use `t.Parallel()`** for independent tests
2. **Use `testing.Short()`** 迁移到 skip slow tests
3. **Use `t.TempDir()`** for temporary directories
4. **Use `t.Setenv()`** for 环境 variables
5. **Avoid `init()`** in 测试 文件
6. **Keep tests focused** - one behavior per 测试
7. **Use meaningful 测试 names** - describe what's being tested

## When 迁移到 Use This Skill

- Writing new Go tests
- Improving 测试 覆盖
- Setting up 测试 infrastructure
- Debugging flaky tests
- Optimizing 测试 性能
- Implementing integration tests