# Mutations 与乐观更新

> **版本基线**：`@tanstack/react-query` **v5.104.x**。
> 官方文档：<https://tanstack.com/query/v5/docs/framework/react/guides/mutations> · <https://tanstack.com/query/v5/docs/framework/react/guides/optimistic-updates>

## 一、useMutation 基础

Mutations 用于**创建/更新/删除**或任何**服务端副作用**（POST/PUT/DELETE）。与查询不同，mutation 不是被动缓存，而是**主动触发**。

```tsx
import { useMutation } from '@tanstack/react-query'

function AddTodo() {
  const mutation = useMutation({
    mutationFn: (newTodo: string) => fetch('/api/todos', {
      method: 'POST',
      body: JSON.stringify({ title: newTodo }),
    }),
    onSuccess: () => {
      console.log('新增成功')
    },
  })

  return (
    <button onClick={() => mutation.mutate('写一篇笔记')} disabled={mutation.isPending}>
      {mutation.isPending ? '提交中...' : '新增'}
    </button>
  )
}
```

### 触发方式：`mutate` vs `mutateAsync`

| 方式 | 返回值 | 适用 |
|---|---|---|
| `mutate(data)` | 无（回调 onSuccess/onError） | 只管触发，UI 用状态 |
| `mutateAsync(data)` | `Promise`（可 await/try-catch） | 需要拿到结果或串行控制流 |

```ts
// mutateAsync 可 await
try {
  await mutation.mutateAsync(newTodo)
} catch (e) {
  // 处理错误
}
```

### mutation 返回状态（与 query 类似）

| 字段 | 说明 |
|---|---|
| `data` | 成功返回的数据 |
| `error` | 错误对象 |
| `isPending` | 正在提交 |
| `isError` | 失败 |
| `isSuccess` | 成功 |
| `isIdle` | 未触发过 |
| `reset()` | 重置状态（清 error/data） |
| `mutate` / `mutateAsync` | 触发 |
| `variables` | 最近一次提交的参数 |

## 二、提交后让查询失效（核心联动）

mutation 成功后最常用的一招：**使相关查询失效并重新拉取**。

```ts
import { useQueryClient, useMutation } from '@tanstack/react-query'

const queryClient = useQueryClient()

const mutation = useMutation({
  mutationFn: addTodo,
  onSuccess: () => {
    // 使 ['todos'] 失效 → 标记过期并重新请求
    queryClient.invalidateQueries({ queryKey: ['todos'] })
  },
})
```

> `invalidateQueries` 详见第 10 篇。它按 key 前缀匹配，`['todos']` 会同时失效 `['todos', 5]`、`['todos', {status:'done'}]`。

## 三、乐观更新（Optimistic Updates）

**先改 UI，再发请求；失败则回滚。** 三要素：`onMutate`（临时改缓存）→ `onError`（回滚）→ `onSettled`（失效/收尾）。

```tsx
const queryClient = useQueryClient()

const updateTodo = useMutation({
  mutationFn: (todo: Todo) => updateTodoServer(todo),
  // 1. 请求前：乐观写入缓存
  onMutate: async (newTodo) => {
    // 取消进行中的请求，避免覆盖乐观值
    await queryClient.cancelQueries({ queryKey: ['todos'] })
    // 记录旧值，用于回滚
    const previous = queryClient.getQueryData(['todos'])
    // 乐观更新缓存
    queryClient.setQueryData(['todos'], (old: Todo[] | undefined) =>
      old?.map(t => (t.id === newTodo.id ? newTodo : t)) ?? [],
    )
    return { previous } // 传给 onError/onSettled
  },
  // 2. 失败：回滚到旧值
  onError: (_err, _newTodo, context) => {
    if (context?.previous) queryClient.setQueryData(['todos'], context.previous)
  },
  // 3. 无论成败：等请求稳定后，从服务器重新拉一份权威数据
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] })
  },
})
```

### 乐观更新的完整模式（推荐模板）

```
onMutate
  ├─ cancelQueries（防竞态）
  ├─ getQueryData 备份旧值
  └─ setQueryData 写乐观值
onError → setQueryData 回滚（用 onMutate 返回的 previous）
onSettled → invalidateQueries（以服务器为准）
```

## 四、mutation 的持久副作用

- **mutation 默认不做持久化/重放**（query 会缓存，mutation 不会自动重试提交失败后的数据）。
- 全局默认 `retry` 对 mutation **默认关闭**（`retry: 0` 除非单独配置）。
- 需要「离线重放」需配合 `persistQueryClient` / MutationCache 的 `onSuccess` 等机制。

## 五、useMutationState（v5 增强）

用于在**触发 mutation 的组件之外**读取某个 mutation 的状态（例如提交中的进度条）：

```ts
import { useMutationState } from '@tanstack/react-query'

// 通过 mutationKey 定位
const variables = useMutationState({
  filters: { mutationKey: ['addTodo'] },
  select: mutation => mutation.state.variables,
})
```

mutation 也可配 `mutationKey`：

```ts
useMutation({
  mutationKey: ['addTodo'],
  mutationFn: addTodo,
})
```

**相关官方链接**
- Mutations：<https://tanstack.com/query/v5/docs/framework/react/guides/mutations>
- Optimistic Updates：<https://tanstack.com/query/v5/docs/framework/react/guides/optimistic-updates>


---

> **上一篇**：[7、请求取消与去重](./7%E3%80%81%E8%AF%B7%E6%B1%82%E5%8F%96%E6%B6%88%E4%B8%8E%E5%8E%BB%E9%87%8D.md)　｜　**下一篇**：[9、QueryClient 与缓存生命周期](./9%E3%80%81QueryClient%20%E4%B8%8E%E7%BC%93%E5%AD%98%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.md)
