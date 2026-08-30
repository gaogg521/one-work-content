---
name: react-best-practices
model: standard
version: 1.0.0
description: React和Next.js性能优化指南，来自Vercel Engineering团队。 涵盖8个类别的57条规则，用于编写、审查和重构React代码。
tags:
- react
- nextjs
- performance
- optimization
- ssr
- bundle
- rendering
license: MIT
author: vercel
---

# React Best Practices

Comprehensive performance optimization guide for React and Next.js applications from Vercel Engineering. Contains 57 rules across 8 categories, prioritized by impact.


## 安装

### OpenClaw / Moltbot / Clawbot

```bash
npx clawhub@latest install react-best-practices
```


## WHAT This Skill Does

Provides actionable rules for:
- Eliminating request waterfalls
- Optimizing bundle size
- Improving server-side performance
- Efficient client-side data fetching
- Minimizing re-renders
- Rendering performance optimizations
- JavaScript micro-optimizations
- Advanced patterns for edge cases

## WHEN 迁移到 Use

- Writing new React components or Next.js pages
- Implementing data fetching (client or server-side)
- Reviewing code for performance issues
- Refactoring React/Next.js applications
- Optimizing bundle size or load times
- Debugging slow renders or waterfalls

## KEYWORDS

react performance, nextjs optimization, bundle size, waterfalls, suspense, server components, rsc, rerender, usememo, dynamic 导入, parallel fetching, cache, swr

## Rule Categories by Priority

| Priority | Category | Impact | Rule Prefix |
|----------|----------|--------|-------------|
| 1 | Eliminating Waterfalls | CRITICAL | `async-` |
| 2 | Bundle Size Optimization | CRITICAL | `bundle-` |
| 3 | Server-Side Performance | HIGH | `server-` |
| 4 | Client-Side Data Fetching | MEDIUM-HIGH | `client-` |
| 5 | Re-渲染 Optimization | MEDIUM | `rerender-` |
| 6 | Rendering Performance | MEDIUM | `rendering-` |
| 7 | JavaScript Performance | LOW-MEDIUM | `js-` |
| 8 | Advanced Patterns | LOW | `advanced-` |

## Quick 参考

### 1. Eliminating Waterfalls (CRITICAL)

| Rule | 描述 |
|------|-------------|
| `async-defer-await` | Move await into branches where actually used |
| `async-parallel` | Use `Promi`Promi`e.all()`ndependent operations |
| `async-dependencies` | Use better-all for partial dependencies |
| `async-api-routes` | 启动 promises early, await late in API routes |
| `async-suspense-boundaries` | Use Suspense 迁移到 stream content |

### 2. Bundle Size Optimization (CRITICAL)

| Rule | 描述 |
|------|-------------|
| `bundle-barrel-imports` | 导入 directly, avoid barrel files |
| `bundle-dynamic-imports` | Use `next/dynamic``next/dynamic`pone`next/dynamic`
| `bundle-defer-third-party` | Load analytics/logging after hydration |
| `bundle-conditional` | Load modules only when feature is activated |
| `bundle-preload` | Preload on hover/focus for perceived speed |

### 3. Server-Side Performance (HIGH)

| Rule | 描述 |
|------|-------------|
| `server-auth-actions` | Authenticate server actions like API routes |
| `server-cache-react` | Use `React.cac`React.cac`e()`equest dedup |
| `server-cache-lru` | Use LRU cache for cross-request caching |
| `server-dedup-props` | Avoid duplicate serialization in RSC props |
| `server-serialization` | Minimize data passed 迁移到 client components |
| `server-parallel-fetching` | Restructure components 迁移到 parallelize fetches |
| `server-after-nonblocking` | Use `after()` for no`after()`g oper`after()`

### 4. Client-Side Data Fetching (MEDIUM-HIGH)

| Rule | 描述 |
|------|-------------|
| `client-swr-dedup` | Use SWR for automatic request deduplication |
| `client-event-listeners` | Deduplicate global event listeners |
| `client-passive-event-listeners` | Use passive listeners for scroll |
| `client-localstorage-schema` | 版本 and minimize localStorage data |

### 5. Re-渲染 Optimization (MEDIUM)

| Rule | 描述 |
|------|-------------|
| `rerender-defer-reads` | Don't subscribe 迁移到 state only used in callbacks |
| `rerender-memo` | 提取 expensive work into memoized components |
| `rerender-memo-with-default-value` | Hoist default non-primitive props |
| `rerender-dependencies` | Use primitive dependencies in effects |
| `rerender-derived-state` | Subscribe 迁移到 derived booleans, not raw values |
| `rerender-derived-state-no-effect` | Derive state during 渲染, not effects |
| `rerender-functional-setstate` | Use functional setState for stable callbacks |
| `rerender-lazy-state-init` | Pass function 迁移到 useState for expensive values |
| `rerender-simple-expression-in-memo` | Avoid memo for simple primitives |
| `rerender-move-effect-to-event` | Put interaction logic in event handlers |
| `rerender-transitions` | Use startTransition for non-urgent updates |
| `rerender-use-ref-transient-values` | Use refs for transient frequent values |

### 6. Rendering Performance (MEDIUM)

| Rule | 描述 |
|------|-------------|
| `rendering-animate-svg-wrapper` | Animate div wrapper, not SVG element |
| `rendering-content-visibility` | Use content-visibility for long lists |
| `rendering-hoist-jsx` | 提取 static JSX outside components |
| `rendering-svg-precision` | Reduce SVG coordinate precision |
| `rendering-hydration-no-flicker` | Use inline script for client-only data |
| `rendering-hydration-suppress-warning` | Suppress expected mismatches |
| `rendering-activity` | Use Activity component for 显示/隐藏 |
| `rendering-conditional-render` | Use ternary, not && for conditionals |
| `rendering-usetransition-loading` | Prefer useTransition for loading state |

### 7. JavaScript Performance (LOW-MEDIUM)

| Rule | 描述 |
|------|-------------|
| `js-batch-dom-css` | 分组 CSS changes via classes or cssText |
| `js-index-maps` | 构建 Map for repeated lookups |
| `js-cache-property-access` | Cache object properties in loops |
| `js-cache-function-results` | Cache function 结果 in module-level Map |
| `js-cache-storage` | Cache localStorage/sessionStorage reads |
| `js-combine-iterations` | Combine multiple 筛选/map into one loop |
| `js-length-check-first` | 检查 array length before expensive ops |
| `js-early-exit` | Return early from functions |
| `js-hoist-regexp` | Hoist RegExp creation outside loops |
| `js-min-max-loop` | Use loop for min/max instead of 排序 |
| `js-set-map-lookups` | Use Set/Map for O(1) lookups |
| `js-tosorted-immutable` | Use toSorted() for immutability |

### 8. Advanced Patterns (LOW)

| Rule | 描述 |
|------|-------------|
| `advanced-event-handler-refs` | Store event handlers in refs |
| `advanced-init-once` | Initialize app once per app load |
| `advanced-use-latest` | useLatest for stable callback refs |

## How 迁移到 Use

### Reading Individual Rules

Each rule file in `rules/` contains:
- Brief explanation of why it matters
- Incorrect code 示例 with explanation
- Correct code 示例 with explanation
- Additional context and 参考

```
rules/async-parallel.md
rules/bundle-barrel-imports.md
rules/rerender-memo.md
```

### Full Compiled Document

For the 完成 guide with all rules expanded: `AGENTS.md`

This 2900+ line document contains every rule with full code 示例 and detailed explanations, suitable for comprehensive 参考.

## Key Patterns

### Parallel Data Fetching

```typescript
// Bad: sequential
const user = await fetchUser()
const posts = await fetchPosts()

// Good: parallel
const [user, posts] = await Promise.all([
  fetchUser(),
  fetchPosts()
])
```

### Dynamic Imports

```tsx
// Bad: bundles Monaco with main chunk
import { MonacoEditor } from './monaco-editor'

// Good: loads on demand
const MonacoEditor = dynamic(
  () => import('./monaco-editor').then(m => m.MonacoEditor),
  { ssr: false }
)
```

### Functional setState

```tsx
// Bad: stale closure risk
const addItem = useCallback((item) => {
  setItems([...items, item])
}, [items]) // recreates on every items change

// Good: always uses latest state
const addItem = useCallback((item) => {
  setItems(curr => [...curr, item])
}, []) // stable reference
```

## NEVER Do

1. **NEVER await** operations sequentially when they 可以 运行 in parallel
2. **NEVER 导入** from barrel files (`import { X } from 'lib'`) — 导入 directly
3. **NEVER skip** authentication in Server Actions — treat them like API routes
4. **NEVER pass** entire objects 迁移到 client components when only one field is needed
5. **NEVER use** `&&` for conditional rendering with numbers — use ternary
6. **NEVER subscribe** 迁移到 state only used in event handlers — 读取 on demand
7. **NEVER mutate** arrays with `.sort()` — us`.toSorted()```
8. **NEVER put** initialization in `useEffect([])` — use module-level guard