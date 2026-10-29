# Hooks API 全量速查

> **版本基线**：`@tanstack/react-query` **v5.104.x**。
> 官方文档：<https://tanstack.com/query/v5/docs/framework/react/reference/useQuery> 等（见文末链接）

## 一、13 个 Hooks 总览

| # | Hook | 用途 | 形态 |
|---|---|---|---|
| 1 | `useQuery` | 基础查询 | 返回 QueryResult |
| 2 | `useSuspenseQuery` | Suspense 查询 | data 非空 |
| 3 | `useQueries` | 并行批量查询（可 combine） | 返回数组/合并结果 |
| 4 | `useSuspenseQueries` | Suspense 并行查询 | data 非空 |
| 5 | `useInfiniteQuery` | 无限滚动查询 | pages/pageParams |
| 6 | `useSuspenseInfiniteQuery` | Suspense 无限查询 | pages 非空 |
| 7 | `useMutation` | 变更（增删改/副作用） | 触发式 |
| 8 | `useMutationState` | 在外部读 mutation 状态 | 按 filters 查询 |
| 9 | `useQueryClient` | 获取 QueryClient 实例 | - |
| 10 | `useIsFetching` | 是否有查询在请求 | 返回计数 |
| 11 | `useIsMutating` | 是否有 mutation 在提交 | 返回计数 |
| 12 | `usePrefetchQuery` | 组件渲染时预取 | 返回 void |
| 13 | `usePrefetchInfiniteQuery` | 组件渲染时预取无限查询 | 返回 void |

## 二、逐 Hook 速查

### 1. useQuery

```ts
const result = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
})
// 常用返回：data / error / isPending / isFetching / isStale / refetch / status / fetchStatus
```

### 2. useSuspenseQuery

```ts
const { data } = useSuspenseQuery({ queryKey: ['todos'], queryFn: fetchTodos })
// data 非空；错误抛给 ErrorBoundary
```

### 3. useQueries

```ts
const results = useQueries({
  queries: ids.map(id => ({ queryKey: ['user', id], queryFn: () => fetchUser(id) })),
  combine: rs => ({ data: rs.map(r => r.data), isLoading: rs.some(r => r.isPending) }),
})
```

### 4. useSuspenseQueries

```ts
const [a, b] = useSuspenseQueries({
  queries: [
    { queryKey: ['a'], queryFn: fetchA },
    { queryKey: ['b'], queryFn: fetchB },
  ],
})
```

### 5. useInfiniteQuery

```ts
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
  queryKey: ['projects'],
  queryFn: ({ pageParam }) => fetchProjects(pageParam),
  initialPageParam: 0,
  getNextPageParam: last => last.nextCursor,
})
// data.pages / data.pageParams
```

### 6. useSuspenseInfiniteQuery

```ts
const { data, fetchNextPage, hasNextPage } = useSuspenseInfiniteQuery({
  queryKey: ['projects'],
  queryFn: ({ pageParam }) => fetchProjects(pageParam),
  initialPageParam: 0,
  getNextPageParam: last => last.nextCursor,
})
```

### 7. useMutation

```ts
const mutation = useMutation({
  mutationFn: updateTodo,
  onSuccess: (data, vars, ctx) => {},
  onError: (err, vars, ctx) => {},
  onSettled: (data, err, vars, ctx) => {},
})
mutation.mutate(payload)      // 触发（不返回 Promise）
await mutation.mutateAsync(payload) // 触发（可 await）
```

### 8. useMutationState

```ts
const states = useMutationState({
  filters: { mutationKey: ['addTodo'] },
  select: m => m.state.variables,
})
```

### 9. useQueryClient

```ts
const queryClient = useQueryClient()
queryClient.invalidateQueries({ queryKey: ['todos'] })
```

### 10. useIsFetching

```ts
// 返回当前正在请求的查询数量（0 = 无）
const isFetching = useIsFetching({ queryKey: ['todos'] })
```

### 11. useIsMutating

```ts
const isMutating = useIsMutating({ mutationKey: ['addTodo'] })
```

### 12 / 13. usePrefetchQuery / usePrefetchInfiniteQuery

```ts
usePrefetchQuery({ queryKey: ['todos'], queryFn: fetchTodos })
usePrefetchInfiniteQuery({ queryKey: ['projects'], queryFn: fetchProjects, initialPageParam: 0 })
```

## 三、返回值字段记忆法（QueryResult）

- 数据相关：`data` / `dataUpdatedAt` / `isFetched` / `isFetching`
- 状态相关：`status` / `fetchStatus` / `isPending` / `isError` / `isSuccess`
- 错误相关：`error` / `errorUpdatedAt` / `failureCount` / `failureReason`
- 新鲜度：`isStale`
- 操作：`refetch` / `remove`（useQuery 返回）

## 四、对应 v4 命名（迁移提示）

| v5 | v4 近似 |
|---|---|
| `isPending` | `isLoading`（部分） |
| `placeholderData: keepPreviousData` | `keepPreviousData` |
| `isInitialLoading` | `isLoading` |
| `useSuspenseQuery` 系列 | 需自行 + React.Suspense |

**相关官方链接（API Reference）**
- useQuery：<https://tanstack.com/query/v5/docs/framework/react/reference/useQuery>
- useInfiniteQuery：<https://tanstack.com/query/v5/docs/framework/react/reference/useInfiniteQuery>
- useMutation：<https://tanstack.com/query/v5/docs/framework/react/reference/useMutation>
- useQueryClient：<https://tanstack.com/query/v5/docs/framework/react/reference/useQueryClient>
- useIsFetching / useIsMutating：<https://tanstack.com/query/v5/docs/framework/react/reference/useIsFetching>


---

> **上一篇**：[15、DevTools、持久化与测试](./15%E3%80%81DevTools%E3%80%81%E6%8C%81%E4%B9%85%E5%8C%96%E4%B8%8E%E6%B5%8B%E8%AF%95.md)　｜　**下一篇**：[17、v4→v5 迁移与选型对比](./17%E3%80%81v4%E2%86%92v5%20%E8%BF%81%E7%A7%BB%E4%B8%8E%E9%80%89%E5%9E%8B%E5%AF%B9%E6%AF%94.md)
