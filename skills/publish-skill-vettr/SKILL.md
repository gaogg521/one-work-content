---
name: publish-skill-vettr
description: 第三方 OpenClaw 技能的静态分析安全扫描器。在安装前检测 eval/spawn 风险、恶意依赖、typosquatting 与 prompt injection 模式。适用于审查来自 ClawHub 或不可信来源的技能。触发词：技能审查(skill vetting)、静态分析(static analysis)、typosquatting、安全扫描(security scan)
---

# skill-vettr v2.0.1

第三方 OpenClaw 技能的安全扫描器。在安装前使用 tree-sitter AST 解析和正则表达式模式匹配分析源代码、依赖和元数据。

## 命令

- `/skill:vet --path <directory>` — 审查本地技能目录
- `/skill:vet-url --url <https://...>` — 从 URL 下载并审查
- `/skill:vet-clawhub --skill <slug>` — 从 ClawHub 获取并审查

## 检测类别

| 类别 | 方法 | 示例 |
|----------|--------|----------|
| 代码执行 | AST | eval(), new Function(), vm.runInThisContext() |
| Shell 注入 | AST | exec(), execSync(), spawn("bash"), child_process 导入 |
| 动态 require | AST | require(变量), require(模板字符串) |
| 原型污染 | AST | __proto__ 赋值 |
| Prompt 注入 | 正则 | 指令覆盖、控制 token（在字符串字面量中） |
| 同形异义字攻击 | 正则 | 标识符中的西里尔/希腊相似字符 |
| 编码名称 | 正则 | Unicode/十六进制转义的 "eval", "exec" |
| 凭证路径 | 正则 | .ssh/, .aws/, keychain 路径引用 |
| 网络调用 | AST | fetch() 带有字面量 URL（根据允许列表检查） |
| 恶意依赖 | 配置 | 已知的恶意包、生命周期脚本、git/http 依赖 |
| Typosquatting | Levenshtein | 技能名称与目标编辑距离在 2 以内 |
| 危险权限 | 配置 | SKILL.md 中的 shell:exec, credentials:read |

## 限制

> ⚠️ **这是一个启发式扫描器，具有固有局限性。它不能保证安全。**

- **仅静态分析** — 无法检测运行时行为（例如，安装后获取恶意软件的代码）
- **可能被规避** — 复杂的混淆或多阶段字符串构造可以规避检测
- **仅 JS/TS** — 跳过二进制载荷、图像和非文本文件
- **有限的网络检测** — 仅检测带有字面量 URL 字符串的 `fetch()`；遗漏 axios、http 模块、动态 URL
- **无沙箱** — 不执行或隔离目标代码
- **注释扫描** — Prompt 注入检测扫描字符串字面量，不扫描注释

对于高安全环境，请结合沙箱和手动源代码审查使用。
