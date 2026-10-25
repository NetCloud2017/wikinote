# Suspense 与 useSuspenseQuery

> **版本基线**：`@tanstack/react-query` **v5.104.x**；涉及 Next.js 内容以 **Next.js 16** 为基线。
> 官方文档：<https://tanstack.com/query/v5/docs/framework/react/guides/suspense>

## 一、什么是 Suspense 模式

**Suspense 是 React 的「异步渲染」机制**：组件在数据未就绪时可以「暂停」，由父级 `<Suspense>` 展示 fallback。TanStack Query v5 提供 `useSuspenseQuery`，让查询**融入 Suspense 数据流**，免去手写 `isPending` 判断。

## 二、对比：普通 useQuery vs useSuspenseQuery

| 维度 | `useQuery` | `useSuspenseQuery` |
|---|---|---|
| loading 处理 | 手动 `if (isPending)` | 自动抛给 `<Suspense fallback>` |
| `data` 类型 | `TData \| undefined` | **必为 `TData`**（非空） |
| `error` 处理 | 手动 `if (isError)` | 抛给**最近的 ErrorBoundary** |
| 适合场景 | 局部/条件渲染 | 整块页面数据流 |

```tsx
import { Suspense } from 'react'
import { useSuspenseQuery } from '@tanstack/react-query'

function TodoList() {
  // data 一定是非空的 Todo[]，无需判空
  const { data } = useSuspenseQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  })
  return <ul>{data.map(t => <li key={t.id}>{t.title}</li>)}</ul>
}

export default function Page() {
  return (
    <Suspense fallback={<div>加载中...</div>}>
      <TodoList />
    </Suspense>
  )
}
```

## 三、useSuspenseQuery 注意点

- **`enabled` 不能用在 suspense 版**（会导致 Suspense 挂死）——需要用条件加载就拆组件或用普通 `useQuery`。
- `data` 非空 → 不必写 `data?.`。
- 错误必须由 **ErrorBoundary** 捕获：

```tsx
import { ErrorBoundary } from 'react-error-boundary'

<ErrorBoundary fallback={<div>出错了</div>}>
  <Suspense fallback={<div>加载中...</div>}>
    <TodoList />
  </Suspense>
</ErrorBoundary>
```

## 四、配套的 Suspense 系列 hooks（v5）

| Hook | 用途 |
|---|---|
| `useSuspenseQuery` | 单查询（Suspense） |
| `useSuspenseQueries` | 并行查询（Suspense） |
| `useSuspenseInfiniteQuery` | 无限查询（Suspense） |
| `usePrefetchQuery` / `usePrefetchInfiniteQuery` | 组件内预取（配合 Suspense 免闪） |

```ts
// useSuspenseQueries 并行
const [a, b] = useSuspenseQueries({
  queries: [
    { queryKey: ['a'], queryFn: fetchA },
    { queryKey: ['b'], queryFn: fetchB },
  ],
})

// useSuspenseInfiniteQuery
const { data, fetchNextPage, hasNextPage } = useSuspenseInfiniteQuery({
  queryKey: ['projects'],
  queryFn: ({ pageParam }) => fetchProjects(pageParam),
  initialPageParam: 0,
  getNextPageParam: last => last.nextCursor,
})
```

## 五、Streaming SSR 里的 Suspense

在 Next.js App Router 中，`<Suspense>` 可开启**流式渲染**：首屏先输出 fallback，数据就绪后补流内容（详见第 14 篇）：

```tsx
// app/todos/page.tsx (Server Component 里套 Client Suspense)
export default function Page() {
  return (
    <Suspense fallback={<p>Loading todos...</p>}>
      <TodoList />
    </Suspense>
  )
}
```

> 结合 `prefetchQuery` + `HydrationBoundary` 可在服务端预取、客户端水合时立即命中缓存，避免闪烁（见第 14 篇）。

## 六、什么时候用 Suspense 模式

| 场景 | 推荐 |
|---|---|
| 整页/整块数据依赖 | ✅ `useSuspenseQuery` + Suspense |
| 局部、带条件、表单内联动 | ❌ 用普通 `useQuery`（需要 `enabled`） |
| 首屏 SSR 流式 | ✅ Suspense + 服务端预取 |
| 错误需要单独 UI | ✅ 配 ErrorBoundary |

**相关官方链接**
- Suspense 指南：<https://tanstack.com/query/v5/docs/framework/react/guides/suspense>
- useSuspenseQuery API：<https://tanstack.com/query/v5/docs/framework/react/reference/useSuspenseQuery>
