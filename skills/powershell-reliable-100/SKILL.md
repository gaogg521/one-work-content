---
name: powershell-reliable-1.0.0
description: 在Windows上可靠执行PowerShell命令。避免使用&&，处理参数解析，从会话中断中恢复，确保跨会话连续性。
---

# PowerShell Reliable Execution

执行 命令 reliably on Windows PowerShell. Avoid common pitfalls like `&&` chaining, parameter swallowing, and session interruptions.

## Problem Statement

Windows PowerShell differs from bash in critical ways:

| Issue | Bash | PowerShell | Solution |
|-------|------|------------|----------|
| 命令 chaining | `cmd1 && cmd2` | `cmd`cmd` -ErrorAction 停止; if ($?) { cmd2 }`se semicolons + 错误 handling |
| Parameter parsing | `-arg value` | `-`-`rgument value`case-insensitive) | Use full parameter names |
| Path separators | __CODE`\`CODE`\`CODE`Join-Path`(or `/` in``Join-Path`s) | Use `Join-Path` |
| 输出 redirection | __CO__CODE___CO`>>`_CODE___CO`>>`___ `>>` | `>` `>` `ut-File -Encoding UTF8` `Out-File -Encoding UTF8` |
| Environment vars | `$VAR`_CODE_1_` | ``nv:`env:`nv:` prefix |

## Core Patterns

### 1. Safe 命令 Chaining

**Wrong**:
```powershell
mkdir test && cd test && echo done
```

**Right**:
```powershell
$ErrorActionPreference = 'Stop'
try {
    New-Item -ItemType Directory -Path test -Force
    Set-Location test
    Write-Host 'done'
} catch {
    Write-Error "Failed: $_"
    exit 1
}
```

### 2. Parameter Safety

**Wrong**:
```powershell
git commit -m "message"
```

**Right**:
```powershell
git commit -Message "message"
# Or use splatting:
$params = @{ Message = "message" }
git commit @params
```

### 3. Path Handling

**Wrong**:
```powershell
$path = "C:/Users/name/file.txt"
```

**Right**:
```powershell
$path = Join-Path $env:USERPROFILE "file.txt"
# Or use literal paths:
$path = 'C:\Users\name\file.txt'
```

### 4. 输出 Encoding

**Wrong**:
```powershell
echo "text" > file.txt
```

**Right**:
```powershell
"text" | Out-File -FilePath file.txt -Encoding UTF8
```

### 5. Session Continuity

For long-running 命令:

```powershell
# Start background job
$job = Start-Job -ScriptBlock {
    param($arg)
    # Long operation
} -ArgumentList $arg

# Wait with timeout
Wait-Job $job -Timeout 300

# Get results
if ($job.State -eq 'Completed') {
    Receive-Job $job
} else {
    Stop-Job $job
    Write-Warning "Job timed out"
}
```

## 错误 Recovery

### Retry Pattern

```powershell
function Invoke-Retry {
    param(
        [scriptblock]$Command,
        [int]$MaxAttempts = 3,
        [int]$DelaySeconds = 2
    )
    
    $attempt = 0
    while ($attempt -lt $MaxAttempts) {
        try {
            $attempt++
            return & $Command
        } catch {
            if ($attempt -eq $MaxAttempts) { throw }
            Start-Sleep -Seconds $DelaySeconds
        }
    }
}

# Usage
Invoke-Retry -Command { Invoke-WebRequest -Uri $url } -MaxAttempts 3
```

### Interruption Recovery

```powershell
# Checkpoint pattern
$checkpointFile = ".checkpoint.json"

if (Test-Path $checkpointFile) {
    $state = Get-Content $checkpointFile | ConvertFrom-Json
    Write-Host "Resuming from step $($state.step)"
} else {
    $state = @{ step = 0 }
}

switch ($state.step) {
    0 { 
        # Step 1
        $state.step = 1
        $state | ConvertTo-Json | Out-File $checkpointFile
    }
    1 {
        # Step 2
        Remove-Item $checkpointFile
    }
}
```

## Privacy Security

**All execution is local**:
- NO 命令 logging 迁移到 external services
- NO credential capture in scripts
- NO automatic 上传 of execution 结果
- Sensitive data handled via `[SecureString]`
- Checkpoint files stored in working directory only

**Sensitive Data 筛选**:
Before writing any checkpoint or 记录:
- Exclude `Password`, `Token`_CODE_`ApiKey`ey`iKey`
- Use `[SecureString]` for credentials
- Never echo sensitive variables

## Executable Completion Criteria

A PowerShell 命令 execution is reliable if and only if:

| Criteria | Verification |
|----------|-------------|
| No `&&``Select-String '&&' script.ps1`t.ps1`t.ps1` 返回 nothing |
| 错误 handling present | `Select-String 'try|catch|ErrorAction' script.ps1` matches |
| Paths use Join-Path | `Select-String 'Join-Path|\\$env:' script.ps1` matches |
| 输出 encoding specified | `Select-String 'Out-File.*Encoding' script.ps1` matches |
| Checkpoint for long ops | Checkpoint file pattern present for ops > 60s |
| No hardcoded secrets | `Select-String 'password|token|secret' script.ps1` 返回 nothing |

## Quick 参考

### Common Cmdlet Mappings

| Task | Bash | PowerShell |
|------|------|------------|
| 列表 files | `ls -la`_CODE_1__e`e` |
| 更改 dir | `cd /path` | `Set-Location C:\path` |
| 创建 dir | `mkdir x` `New-Item -ItemType Directory x``` |
| 复制 file | `cp a b`_CODE_1__b`b` |
| Move file | `mv a b`_CODE_1__b`b` |
| 删除 | `rm x`_CODE_1__m x` |
| 查看 file | __CODE_`Get-Content x` x` x` |
| 编辑 file | __CODE_`notepad x` x` x` |
| 查找 text | `grep x`_CODE_1__x`x` |
| Pipe | __CODE___CODE`\|`| `\|` (same) |
| Redirect | __CODE__CODE_`>`_ | `>` (use Out-File) |

### Splatting Template

```powershell
$params = @{
    Path = $filePath
    Encoding = 'UTF8'
    Force = $true
}
Set-Content @params
```

## 参考

- `references/privacy-checklist.md` - Privacy security checklist
- Microsoft Docs: [PowerShell Best Practices](https://docs.microsoft.com/powershell)

---

**执行 reliably. Recover gracefully.**