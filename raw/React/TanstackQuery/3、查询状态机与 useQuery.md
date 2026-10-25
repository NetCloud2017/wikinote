# 查询状态机与 useQuery

> **版本基线**：`@tanstack/react-query` **v5.104.x**。
> 官方文档：<https://tanstack.com/query/v5/docs/framework/react/guides/queries> · <https://tanstack.com/query/v5/docs/framework/react/reference/useQuery>

## 一、useQuery 基本形态（v5 对象式）

```ts
import { useQuery } from '@tanstack/react-query'

const result = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  // 可选配置
  staleTime: 5 * 60 * 1000,
  gcTime: 10 * 60 * 1000,
  retry: 3,
})
```

## 二、双轴状态：`status` × `fetchStatus`（最核心）

TanStack Query 用**两套正交状态**描述查询，别混为一谈：

| 轴 | 字段 | 取值 | 含义 |
|---|---|---|---|
| **数据状态** | `status` | `pending` | 尚无数据 |
| | | `error` | 请求出错 |
| | | `success` | 已有数据 |
| **抓取状态** | `fetchStatus` | `fetching` | 正在请求 |
| | | `paused` | 因网络离线而暂停 |
| | | `idle` | 无请求在跑 |

### 常见组合（背下来）

| status | fetchStatus | 场景 |
|---|---|---|
| `pending` | `fetching` | 首次加载中 |
| `pending` | `idle` | 无缓存且未开始/离线暂停 |
| `success` | `idle` | 数据新鲜，无请求 |
| `success` | `fetching` | **后台静默刷新**（有旧数据同时拉新） |

> ⚠️ 经验：`isLoading` = `isPending && isFetching`（v5 把 v4 的 `isLoading` 语义收敛到「首次加载」）。`isFetching` 只要有任何请求在跑就为 true。

## 三、result 返回值速查

| 字段 | 类型 | 说明 |
|---|---|---|
| `data` | `TData` | 成功数据 |
| `dataUpdatedAt` | `number` | 数据最后更新时间戳 |
| `error` | `Error` | 错误对象 |
| `errorUpdatedAt` | `number` | 错误最后更新时间戳 |
| `failureCount` | `number` | 连续失败次数 |
| `failureReason` | `Error` | 最近失败原因 |
| `errorUpdateCount` | `number` | 错误累计更新次数 |
| `isPending` | `boolean` | `status === 'pending'` |
| `isError` | `boolean` | `status === 'error'` |
| `isSuccess` | `boolean` | `status === 'success'` |
| `isStale` | `boolean` | 是否已过期（超过 staleTime） |
| `isFetching` | `boolean` | 是否有请求在跑 |
| `isPaused` | `boolean` | 是否因离线暂停 |
| `isFetched` | `boolean` | 是否完成过至少一次请求 |
| `isRefetching` | `boolean` | 是否有**后台**刷新在跑 |
| `isInitialLoading` | `boolean` | 首次加载中（`isPending && isFetching`） |
| `refetch` | `function` | 手动重新请求（返回 Promise） |
| `fetchStatus` | `string` | 见上表 |

## 四、常见模式

### 1. 条件渲染

```tsx
function Todos() {
  const { isPending, isError, error, data, isFetching } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  })

  if (isPending) return <span>加载中...</span>
  if (isError) return <span>出错了：{error.message}</span>

  return (
    <div>
      {isFetching ? <span>后台刷新中...</span> : null}
      <ul>{data.map(todo => <li key={todo.id}>{todo.title}</li>)}</ul>
    </div>
  )
}
```

### 2. `isLoading` 语义差异提醒（v4→v5）

| | v4 | v5 |
|---|---|---|
| `isLoading` | 只要无数据就在加载 | 仅 `pending && fetching`（首次加载） |
| 旧缓存 + 后台刷新 | `isFetching=true`，`isLoading` 沿用 | 同 |
| `isRefetching` | 存在 | 存在（等价 `isFetching && !isPending`） |

### 3. `refetch` 手动刷新

```ts
const { refetch } = useQuery({ queryKey: ['todos'], queryFn: fetchTodos })

// 可选：throwOnError 让错误抛给调用方
await refetch({ throwOnError: true })
```

## 五、并行使用多个 useQuery

一个组件里可以安全地多次调用 `useQuery`，它们**互不影响**（去重由内部负责）：

```ts
const users = useQuery({ queryKey: ['users'], queryFn: fetchUsers })
const projects = useQuery({ queryKey: ['projects'], queryFn: fetchProjects })
```

> 想要「一次调用、统一 loading」就用 `useQueries`（见第 4 篇）。

**相关官方链接**
- Queries 指南：<https://tanstack.com/query/v5/docs/framework/react/guides/queries>
- useQuery API：<https://tanstack.com/query/v5/docs/framework/react/reference/useQuery>
