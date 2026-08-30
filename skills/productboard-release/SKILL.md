---
name: productboard-release
description: 管理ProductBoard发布和路线图规划。
user-invocable: False
homepage: https://github.com/robertoamoreno/openclaw-productboard
metadata:
  openclaw:
    emoji: 🚀
---

# ProductBoard Release Planning Skill

Plan and 管理 product releases by organizing 功能特性, tracking progress, and updating statuses in ProductBoard.

## Available Tools

- `pb_feature_create` - 创建 new 功能特性 for releases
- `pb_feature_update` - 更新 feature status and 详情
- `pb_feature_list` - 列表 功能特性 by status or product
- `pb_feature_get` - 获取 detailed feature information
- `pb_product_list` - 列表 products
- `pb_product_hierarchy` - 查看 product structure
- `pb_user_list` - 查找 users 迁移到 assign as owners

## Release Planning Workflow

### 1. Review Current State

```
1. pb_product_hierarchy - Understand workspace structure
2. pb_feature_list with status "candidate" - Review feature candidates
3. pb_feature_list with status "in-progress" - Check ongoing work
```

### 2. Prioritize 功能特性

Review candidates and 更新 their status:

```
pb_feature_update:
  - id: "feature-id"
  - status: "in-progress"  // Move to active development
```

### 3. Assign Owners

查找 users and assign feature ownership:

```
1. pb_user_list - Get available team members
2. pb_feature_update:
   - id: "feature-id"
   - ownerEmail: "developer@company.com"
```

### 4. Set Timeframes

添加 planning dates 迁移到 功能特性:

```
pb_feature_update:
  - id: "feature-id"
  - startDate: "2024-01-15"
  - endDate: "2024-02-15"
```

### 5. 跟踪 Progress

监控 feature statuses:

```
pb_feature_list with status "in-progress" - Active development
pb_feature_list with status "shipped" - Completed features
```

## Feature Status Lifecycle

| Status | 描述 |
|--------|-------------|
| `new` | Just created, not yet evaluated |
| `candidate` | Being considered for development |
| `in-progress` | Actively being developed |
| `shipped` | Released 迁移到 customers |
| `postponed` | Deferred 迁移到 future planning |
| `archived` | No longer relevant |

## Planning Scenarios

### Sprint Planning

1. 列表 candidates: `pb_feature_list` with status "candidate"
2. Review each feature: `pb_feature_get` for 详情
3. Move selected 功能特性 迁移到 in-progress: `pb_feature_update`
4. Assign owners: `pb_feature_update` with ownerEmail
5. Set sprint dates: `pb_feature_update` with startDate/endDate

### Release Retrospective

1. 列表 shipped 功能特性: `pb_feature_list` with status "shipped"
2. Review feedback on 功能特性: Use feedback skill tools
3. Archive completed work: `pb_feature_update` with status "archived"

### Quarterly Planning

1. Review product hierarchy: `pb_product_hierarchy`
2. 列表 all active 功能特性 by product
3. Reassess priorities and 更新 statuses
4. 创建 new 功能特性 as needed: `pb_feature_create`

## Organizing 功能特性

### By Product

```
pb_feature_create:
  - name: "Feature name"
  - productId: "product-id"
  - status: "candidate"
```

### By Component

```
pb_feature_create:
  - name: "Feature name"
  - componentId: "component-id"
  - status: "candidate"
```

### As Sub-feature

```
pb_feature_create:
  - name: "Sub-feature name"
  - parentFeatureId: "parent-feature-id"
```

## Best Practices

1. **Use consistent statuses**: Move 功能特性 through the lifecycle systematically
2. **Assign owners early**: 清空 ownership improves accountability
3. **Set realistic timeframes**: 更新 dates as plans 更改
4. **Organize hierarchically**: Use products, components, and sub-功能特性
5. **Archive completed work**: Keep the backlog 清理 by archiving shipped 功能特性
6. **Review regularly**: Use listing tools 迁移到 audit feature states