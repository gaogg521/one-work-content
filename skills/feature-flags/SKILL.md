---
name: feature-flags
description: 功能标志管理专家 - 特性开关配置、RedisInsight 集成、灰度发布管理
---

# 功能标志

RedisInsight 有自己的功能标志系统。标志在远程 JSON 配置中定义，由后端获取，并通过 API 提供给前端。本技能涵盖如何添加、提升和删除标志。

## 标志类型

| 类型                         | 命名                            | `flag` 值 | 策略                 | 用途                                                      |
| ---------------------------- | --------------------------------- | ------------ | ------------------------ | ------------------------------------------------------------ |
| **开发标志**                 | `dev-<name>` (例如 `dev-browser`) | `false`      | `CommonFlagStrategy`     | 在开发期间隐藏不完整的功能                  |
| **常规标志**             | `camelCase` (例如 `azureEntraId`) | `true`       | `CommonFlagStrategy`     | 标准开/关切换                                       |
| **带数据的常规标志**        | `camelCase`                       | `true`       | `WithDataFlagStrategy`   | 标志 + `data` 中的额外配置负载                        |
| **可切换（可覆盖）** | `camelCase`                       | `true`       | `SwitchableFlagStrategy` | 用户可以通过 `~/.redis-insight/config.json` 本地覆盖 |

## 需要变更的文件

每个新标志都会触及这些文件（按顺序）：

### 后端（必需）

1. **`redisinsight/api/config/features-config.json`**
   添加带有 `flag`、`perc`、可选 `filters` 和 `data` 的标志条目。提升 `version` 版本号。

2. **`redisinsight/api/src/modules/feature/constants/index.ts`**
   添加到 `KnownFeatures` 枚举。

3. **`redisinsight/api/src/modules/feature/constants/known-features.ts`**
   添加条目到 `knownFeatures` 记录，包含 `name` 和 `storage`（通常是 `FeatureStorage.Database`）。

4. **`redisinsight/api/src/modules/feature/providers/feature-flag/feature-flag.provider.ts`**
   使用其策略注册标志（见下面的策略类型）。

### 前端（如果标志控制 UI 则必需）

5. **`redisinsight/ui/src/constants/featureFlags.ts`**
   添加到 `FeatureFlags` 枚举。

6. **`redisinsight/ui/src/slices/app/features.ts`**
   在 `initialState.featureFlags.features` 中添加默认状态条目，包含 `{ flag: false }`。

### 消费代码

7. 在组件/钩子中使用标志来控制功能。

## 策略选择

根据标志需要的内容选择策略：

```
CommonFlagStrategy        → 大多数标志（开发和常规开/关）
WithDataFlagStrategy      → 标志需要携带额外数据负载
SwitchableFlagStrategy    → 标志应该可以通过本地 config.json 覆盖
```

在 `feature-flag.provider.ts` 中注册：

```typescript
this.strategies.set(
  KnownFeatures.YourFeature,
  new CommonFlagStrategy(this.featuresConfigService, this.settingsService),
);
```

## 配置 JSON 结构

### 最小化（开发标志）

```json
"dev-myFeature": {
  "flag": false,
  "perc": [[0, 100]]
}
```

### 带过滤器（仅限 Electron）

```json
"myFeature": {
  "flag": true,
  "perc": [[0, 100]],
  "filters": [
    { "name": "config.server.buildType", "value": "ELECTRON", "cond": "eq" }
  ]
}
```

### 逐步推出（10% 用户）

```json
"myFeature": {
  "flag": true,
  "perc": [[0, 10]]
}
```

### 带数据负载

```json
"myFeature": {
  "flag": true,
  "perc": [[0, 100]],
  "data": { "strategy": "ioredis" }
}
```

## 过滤器条件

过滤器将服务器状态中的值与过滤器值进行比较。

| 条件    | 含义                         |
| ------------ | ------------------------------- |
| `eq`         | 等于                          |
| `neq`        | 不等于                      |
| `gt` / `gte` | 大于 / 大于或等于 |
| `lt` / `lte` | 小于 / 小于或等于       |

常见的 `name` 路径：`config.server.buildType` (ELECTRON, DOCKER_ON_PREMISE, REDIS_STACK), `config.server.packageVersion` (使用 semver), `agreements.analytics`, `env.<VAR_NAME>`。

过滤器支持用于复杂条件的 `and`/`or` 组合。

## 工作流程

### 添加开发功能标志

用于正在积极开发中不应在生产环境中可见的功能。

1. `features-config.json` → 添加 `"dev-myFeature": { "flag": false, "perc": [[0, 100]] }`
2. `constants/index.ts` → 添加 `DevMyFeature = 'dev-myFeature'` 到 `KnownFeatures`
3. `constants/known-features.ts` → 添加记录条目
4. `feature-flag.provider.ts` → 使用 `CommonFlagStrategy` 注册
5. `ui/src/constants/featureFlags.ts` → 添加 `devMyFeature = 'dev-myFeature'`
6. `ui/src/slices/app/features.ts` → 添加默认 `{ flag: false }`

### 将开发标志提升为常规标志

当功能完成并准备推出时。

1. 在所有上述文件中将 `dev-myFeature` 重命名为 `myFeature`
2. 在 `features-config.json` 中设置 `flag: true`
3. 可选地设置 `perc` 以进行逐步推出（例如 `[[0, 10]]`）
4. 如果需要，更改策略（例如改为 `SwitchableFlagStrategy` 以可覆盖）
5. 提升配置 `version`

### 清理标志

当功能完全推出且不再需要标志时。

1. 从 `features-config.json` 中移除
2. 从 `KnownFeatures` 枚举中移除
3. 从 `knownFeatures` 记录中移除
4. 从 `feature-flag.provider.ts` 中移除策略注册
5. 从前端 `FeatureFlags` 枚举中移除
6. 从 `features.ts` 中移除默认状态
7. 在消费组件中移除所有门控代码（条件语句、`FeatureFlagComponent` 包装器）

## 前端使用模式

### 在组件中检查标志

```typescript
import { FeatureFlags } from 'uiSrc/constants';
import { appFeatureFlagsFeaturesSelector } from 'uiSrc/slices/app/features';

const features = useSelector(appFeatureFlagsFeaturesSelector);
const isEnabled = features[FeatureFlags.myFeature]?.flag;
```

### 复杂逻辑的自定义选择器

```typescript
export const isMyFeatureEnabledSelector = (state: RootState): boolean => {
  const features = state.app.features.featureFlags.features;
  return features[FeatureFlags.myFeature]?.flag ?? false;
};
```
