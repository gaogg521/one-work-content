---
name: react-modernization
model: reasoning
description: React 现代化技能。将 class 组件升级为 hooks，采用 React 18/19 并发(concurrent)特性，执行版本迁移与 TypeScript 转换。适用于 class-to-hooks 重构、版本升级与性能优化。触发词：React 升级(React upgrade)、class 转 hooks、并发特性(concurrent features)、TypeScript 迁移、codemod
tags:
- 前端
---

# React 现代化

将 React 应用从 class 组件升级到 hooks，采用并发特性，并在主要版本之间迁移。

## WHAT

现代化 React 代码库的系统性模式：
- 带生命周期方法映射的 class-to-hooks 迁移
- React 18/19 并发特性采用
- React 组件的 TypeScript 迁移
- 批量重构的自动化 codemods
- 使用现代 API 的性能优化

## WHEN

- 将 class 组件迁移到带 hooks 的 function 组件
- 将 React 16/17 应用升级到 React 18/19
- 采用并发特性（Suspense, transitions, use）
- 将 HOC 和 render props 转换为 custom hooks
- 为 React 项目添加 TypeScript

## KEYWORDS

react upgrade, class to hooks, useEffect, useState, react 18, react 19, concurrent, suspense, transition, codemod, migrate, modernize, functional component


## 安装

### OpenClaw / Moltbot / Clawbot

```bash
npx clawhub@latest install react-modernization
```


---

## 版本升级路径

### React 17 → 18 破坏性变更

| 变更 | 影响 | 迁移 |
|--------|--------|-----------|
| 新 root API | 必需 | `ReactDOM.render` → `createRoot` |
| 自动批处理 | 行为 | 状态更新现在在异步代码中批处理 |
| Strict Mode | 仅开发 | Effects 触发两次（mount/unmount/mount） |
| 服务端 Suspense | 可选 | 启用 SSR streaming |

### React 18 → 19 破坏性变更

| 变更 | 影响 | 迁移 |
|--------|--------|-----------|
| `use()` hook | 新 API | 在 render 中读取 promises/context |
| `ref` 作为 prop | 简化 | 不再需要 `forwardRef` |
| Context 作为 provider | 简化 | `<Context>` 而非 `<Context.Provider>` |
| 异步 actions | 新模式 | `useActionState`, `useOptimistic` |

---

## Class 到 Hooks 迁移

### 生命周期方法映射

```tsx
// componentDidMount → 带空依赖的 useEffect
useEffect(() => {
  fetchData()
}, [])

// componentDidUpdate → 带依赖的 useEffect
useEffect(() => {
  updateWhenIdChanges()
}, [id])

// componentWillUnmount → useEffect cleanup
useEffect(() => {
  const subscription = subscribe()
  return () => subscription.unsubscribe()
}, [])

// shouldComponentUpdate → React.memo
const Component = React.memo(({ data }) => <div>{data}</div>)

// getDerivedStateFromProps → useMemo
const derivedValue = useMemo(() => computeFrom(props), [props])
```

### State 迁移模式

```tsx
// 之前：带多个 state 属性的 class
class UserProfile extends React.Component {
  state = { user: null, loading: true, error: null }
  
  componentDidMount() {
    fetchUser(this.props.id)
      .then(user => this.setState({ user, loading: false }))
      .catch(error => this.setState({ error, loading: false }))
  }
  
  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.setState({ loading: true })
      fetchUser(this.props.id)
        .then(user => this.setState({ user, loading: false }))
    }
  }
  
  render() {
    const { user, loading, error } = this.state
    if (loading) return <Spinner />
    if (error) return <Error message={error.message} />
    return <Profile user={user} />
  }
}

// 之后：Custom hook + function 组件
function useUser(id: string) {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<Error | null>(null)

  useEffect(() => {
    let cancelled = false
    setLoading(true)
    
    fetchUser(id)
      .then(data => {
        if (!cancelled) {
          setUser(data)
          setLoading(false)
        }
      })
      .catch(err => {
        if (!cancelled) {
          setError(err)
          setLoading(false)
        }
      })

    return () => { cancelled = true }
  }, [id])

  return { user, loading, error }
}

function UserProfile({ id }: { id: string }) {
  const { user, loading, error } = useUser(id)
  
  if (loading) return <Spinner />
  if (error) return <Error message={error.message} />
  return <Profile user={user} />
}
```

### HOC 到 Hook 迁移

```tsx
// 之前：Higher-Order Component
function withUser(Component) {
  return function WithUser(props) {
    const [user, setUser] = useState(null)
    useEffect(() => { fetchUser().then(setUser) }, [])
    return <Component {...props} user={user} />
  }
}

const ProfileWithUser = withUser(Profile)

// 之后：Custom hook（更简单，可组合）
function useCurrentUser() {
  const [user, setUser] = useState(null)
  useEffect(() => { fetchUser().then(setUser) }, [])
  return user
}

function Profile() {
  const user = useCurrentUser()
  return user ? <div>{user.name}</div> : null
}
```

---

## React 18+ 并发特性

### 新 Root API（必需）

```tsx
// 之前：React 17
import ReactDOM from 'react-dom'
ReactDOM.render(<App />, document.getElementById('root'))

// 之后：React 18+
import { createRoot } from 'react-dom/client'
const root = createRoot(document.getElementById('root')!)
root.render(<App />)
```

### 用于非紧急更新的 useTransition

```tsx
function SearchResults() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isPending, startTransition] = useTransition()

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    // 紧急：立即更新 input
    setQuery(e.target.value)
    
    // 非紧急：可以被中断
    startTransition(() => {
      setResults(searchDatabase(e.target.value))
    })
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending ? <Spinner /> : <ResultsList data={results} />}
    </>
  )
}
```

### 用于数据获取的 Suspense

```tsx
// 使用 React 19 的 use() hook
function ProfilePage({ userId }: { userId: string }) {
  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <ProfileDetails userId={userId} />
    </Suspense>
  )
}

function ProfileDetails({ userId }: { userId: string }) {
  // use() 在 promise resolve 前 suspend
  const user = use(fetchUser(userId))
  return <h1>{user.name}</h1>
}
```

### React 19: use() Hook

```tsx
// 直接在 render 中读取 promises
function Comments({ commentsPromise }) {
  const comments = use(commentsPromise)
  return comments.map(c => <Comment key={c.id} {...c} />)
}

// 读取 context（比 useContext 更简单）
function ThemeButton() {
  const theme = use(ThemeContext)
  return <button className={theme}>Click</button>
}
```

### React 19: Actions

```tsx
// 用于表单提交的 useActionState
function UpdateName() {
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const error = await updateName(formData.get('name'))
      if (error) return error
      redirect('/profile')
    },
    null
  )

  return (
    <form action={submitAction}>
      <input name="name" />
      <button disabled={isPending}>Update</button>
      {error && <p>{error}</p>}
    </form>
  )
}
```

---

## 自动化 Codemods

### 运行官方 React Codemods

```bash
# 更新到新 JSX transform（不需要 React import）
npx codemod@latest react/19/replace-reactdom-render

# 更新废弃 API
npx codemod@latest react/19/replace-string-ref

# Class 到 function 组件
npx codemod@latest react/19/replace-use-form-state
```

### 手动搜索模式

```bash
# 查找 class 组件
rg "class \w+ extends (React\.)?Component" --type tsx

# 查找废弃生命周期方法
rg "componentWillMount|componentWillReceiveProps|componentWillUpdate" --type tsx

# 查找 ReactDOM.render（需要迁移到 createRoot）
rg "ReactDOM\.render" --type tsx
```

---

## TypeScript 迁移

```tsx
// 为 function 组件添加类型
interface ButtonProps {
  onClick: () => void
  children: React.ReactNode
  variant?: 'primary' | 'secondary'
}

function Button({ onClick, children, variant = 'primary' }: ButtonProps) {
  return (
    <button onClick={onClick} className={variant}>
      {children}
    </button>
  )
}

// 类型化 event handlers
function Form() {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
  }
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value)
  }

  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleChange} />
    </form>
  )
}

// 泛型组件
interface ListProps<T> {
  items: T[]
  renderItem: (item: T) => React.ReactNode
}

function List<T>({ items, renderItem }: ListProps<T>) {
  return <>{items.map(renderItem)}</>
}
```

---

## 迁移检查清单

### 迁移前
- [ ] 增量升级依赖
- [ ] 查看 release notes 中的破坏性变更
- [ ] 建立全面的测试覆盖
- [ ] 创建功能分支

### Class → Hooks
- [ ] 从叶子组件开始（无子组件）
- [ ] 将 state 转换为 `useState`
- [ ] 将生命周期转换为 `useEffect`
- [ ] 将共享逻辑提取到 custom hooks
- [ ] 尽可能将 HOC 转换为 hooks

### React 18+ 升级
- [ ] 更新到 `createRoot` API
- [ ] 用 StrictMode 双调用测试
- [ ] 处理 hydration 不匹配
- [ ] 在有益处的地方采用 Suspense boundaries
- [ ] 对昂贵更新使用 transitions

### 迁移后
- [ ] 运行完整测试套件
- [ ] 检查 console 警告
- [ ] 对比迁移前后性能
- [ ] 为团队记录变更

---

## NEVER

- 迁移后跳过测试
- 在一个 commit 中迁移多个组件
- 忽略 StrictMode 警告（它们揭示 bug）
- 不理解原因就使用 `// eslint-disable-next-line react-hooks/exhaustive-deps`
- 在同一个组件中混用 class 和 hooks
