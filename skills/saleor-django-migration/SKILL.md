---
name: saleor-django-migration
description: 使用 manage.py makemigrations 为 Saleor 代码库生成 Django 模式迁移。在以下情况使用：(1) 创建新的 Django 模型，(2) 添加/修改/删除模型字段，(3) 用户明确要求创建迁移。此技能仅涵盖同步模式迁移（CreateModel、AddField、AlterField、RemoveField 等）——不包括使用 RunPython 或 RunSQL 的数据迁移。如果不确定迁移是否仅为模式迁移，请询问用户。
---

# Saleor Django 模式迁移

## 范围

此技能处理**仅模式**迁移：CreateModel、AddField、AlterField、RemoveField、AddIndex、AddConstraint 等。它不包括：
- 数据迁移（RunPython、使用数据更改的 RunSQL）
- 异步/后台迁移（celery 任务、post_migrate 信号）

如果更改需要数据转换，请停止并询问用户。

## 关键规则

1. **永远不要手写迁移文件。** 始终让 Django 生成它们。
2. **始终使用 `./manage.py makemigrations`** 生成迁移。
3. **首先激活虚拟环境** — 在任何 manage.py 命令之前运行 `. .venv/bin/activate`。
4. **如果数据库连接失败**，停止并询问用户启动数据库。

## 工作流程

### 步骤 1：进行模型更改

根据需要编辑 Django 模型文件。不要接触任何迁移文件。

### 步骤 2：生成迁移

```bash
. .venv/bin/activate && python manage.py makemigrations
```

如果 Django 询问交互式问题（例如"您是否将字段 X 重命名为 Y？"），根据您进行的模型更改回答。

### 步骤 3：阅读并分析生成的迁移

阅读每个生成的迁移文件。验证：
- 它只包含模式操作（CreateModel、AddField、AlterField、RemoveField、AddIndex 等）
- 依赖项看起来正确
- 没有意外的操作

### 步骤 4：应用分离规则

将以下规则应用于生成的迁移：

**规则 A — 新模型每个迁移一个模型：**
如果迁移创建了**多个**新模型（多个 `CreateModel` 操作），请拆分它们。分别生成每个：
```bash
python manage.py makemigrations <app>
```
如果 Django 将它们一起生成，删除组合迁移，然后逐个创建模型（将一个模型添加到 models.py，运行 makemigrations，重复）。

**规则 B — 每个迁移一个字段更改：**
如果迁移在单个模型中更改了**多个**字段（例如，同一模型上的两个 `AddField` 或两个 `AlterField` 操作），请拆分为单独的迁移 — 每个字段一个。删除组合迁移，一次进行一次字段更改，并在每次之后运行 makemigrations。

**例外（不要拆分）：**
- 一个带有所有字段的 `CreateModel` 是一个操作 — 不要拆分新模型的字段
- Django 需要它们在一起的跨模型依赖（例如 ForeignKey + 相关模型创建）
- 当自动生成在一起时，同一迁移中的字段及其索引/约束是可接受的

### 步骤 5：验证

运行检查命令以确认没有待处理的迁移：
```bash
python manage.py makemigrations --check --dry-run
```

预期输出：`No changes detected` — 意味着所有模型更改都已在迁移中捕获。

## 错误处理

| 错误 | 操作 |
|-------|--------|
| `django.db.utils.OperationalError: could not connect` | 询问用户启动数据库 |
| `Conflicting migrations detected` | 停止并让用户决定 |
| `ModuleNotFoundError` | 虚拟环境未激活 — 运行 `. .venv/bin/activate` |

## 提示

- 依赖自动生成的迁移来获取正确的时间戳、序列号、依赖项和字段定义。如果必须手动编辑（例如拆分），请重新生成而不是编辑 — 删除迁移，调整 models.py，然后重新运行 makemigrations。
- 拆分时，让 Django 确定新较小迁移之间的依赖关系。
- 在生成之前检查应用程序中的最新现有迁移（`ls saleor/<app>/migrations/ | tail -5`），以便您可以确认新迁移号是连续的。
