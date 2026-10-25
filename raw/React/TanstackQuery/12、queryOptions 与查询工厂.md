# queryOptions 与查询工厂

> **版本基线**：`@tanstack/react-query` **v5.104.x**。
> 官方文档：<https://tanstack.com/query/v5/docs/framework/react/guides/query-options>

## 一、为什么要 queryOptions

手写 `useQuery({ queryKey, queryFn })` 时，**key 与函数散落各处**，一旦改 key 会漏同步，类型也容易丢失。`queryOptions` 把「一个查询的完整定义」**收敛到一个可复用对象**，并**推导类型**。

## 二、基本用法

```ts
import { queryOptions } from '@tanstack/react-query'

// 集中定义（通常放 api/模块目录）
export const todoOptions = queryOptions({
  queryKey: ['todos'],
  queryFn: fetchTodos,
})
```

组件里直接复用：

```ts
// 类型自动推导为 Todo[]，且 key/queryFn 保证一致
const { data } = useQuery(todoOptions)
```

预取 / 读缓存也复用同一对象，**杜绝 key 漂移**：

```ts
await queryClient.prefetchQuery(todoOptions)
queryClient.setQueryData(todoOptions.queryKey, ...)
```

## 三、带参数：函数工厂

需要入参的查询，用「工厂函数」返回 options：

```ts
export const todoOptions = {
  all: () => queryOptions({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  }),
  detail: (id: number) => queryOptions({
    queryKey: ['todos', id],
    queryFn: () => fetchTodo(id),
  }),
  list: (filter: Filter) => queryOptions({
    queryKey: ['todos', 'list', filter],
    queryFn: () => fetchTodos(filter),
  }),
}
```

用法：

```ts
const { data } = useQuery(todoOptions.detail(5))
const { data: list } = useQuery(todoOptions.list({ status: 'done' }))

// 跨模块失效也只用同一 key 来源
queryClient.invalidateQueries({ queryKey: todoOptions.all().queryKey })
```

## 四、类型安全收益

`queryOptions` 的关键价值是**让 key 与 data 类型绑定**：

```ts
// data 类型由 queryFn 的返回类型推导，无需手动标注
const detail = todoOptions.detail(5)
// detail.queryKey: readonly ['todos', number]
// useQuery(detail) → data: Todo
```

> 这解决了「`useQuery` 参数松散、key 拼写错也查不到」的工程痛点。

## 五、queryKeyFactory（可选进阶）

若不想每次调用都跑函数，可先建 key 工厂再配 options：

```ts
export const todoKeys = {
  all: ['todos'] as const,
  lists: () => [...todoKeys.all, 'list'] as const,
  list: (filter: Filter) => [...todoKeys.lists(), filter] as const,
  details: () => [...todoKeys.all, 'detail'] as const,
  detail: (id: number) => [...todoKeys.details(), id] as const,
}

export const todoListOptions = (filter: Filter) =>
  queryOptions({
    queryKey: todoKeys.list(filter),
    queryFn: () => fetchTodos(filter),
  })
```

> 好处：失效/手动更新时用 `todoKeys.list(f)` 精确命中，与查询定义天然一致。

## 六、最佳实践

1. **所有查询都走 `queryOptions`**，key 只有一个来源。
2. 参数化查询用**工厂函数**，参数进 key。
3. 失效/预取/读缓存一律复用 `.queryKey`，不手写 key。
4. 复杂层级用 `queryKeyFactory` 管理 key 的继承关系。

**相关官方链接**
- Query Options：<https://tanstack.com/query/v5/docs/framework/react/guides/query-options>
- Query Keys：<https://tanstack.com/query/v5/docs/framework/react/guides/query-keys>
