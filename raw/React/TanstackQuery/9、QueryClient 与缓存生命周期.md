# QueryClient 与缓存生命周期

> **版本基线**：`@tanstack/react-query` **v5.104.x**。
> 官方文档：<https://tanstack.com/query/v5/docs/framework/react/guides/caching> · <https://tanstack.com/query/v5/docs/framework/react/reference/QueryClient>

## 一、QueryClient 是什么

`QueryClient` 是**所有查询的容器**：缓存 + 默认配置 + 执行方法。在组件外用 `new QueryClient()` 创建，组件内用 `useQueryClient()` 获取。

```ts
// 组件内拿 client
import { useQueryClient } from '@tanstack/react-query'
const queryClient = useQueryClient()
```

## 二、QueryClient 核心方法速查

| 方法 | 作用 |
|---|---|
| `getQueryData(key)` | 读缓存（同步，不请求） |
| `setQueryData(key, updater)` | 写缓存（同步） |
| `fetchQuery(opts)` | 取数据：命中缓存则返回，未命中则请求并写入（返回 Promise） |
| `prefetchQuery(opts)` | 预取（失败不抛错） |
| `getQueryState(key)` | 读查询状态（含 dataUpdatedAt 等） |
| `invalidateQueries(filters)` | 标记过期并可重拉（见第 10 篇） |
| `refetchQueries(filters)` | 直接重新请求匹配查询 |
| `resetQueries(filters)` | 重置查询到初始状态 |
| `removeQueries(filters)` | 从缓存**移除**（含数据） |
| `cancelQueries(filters)` | 取消进行中的请求 |
| `setDefaultOptions(opts)` | 设置默认选项 |
| `clear()` | 清空整个缓存 |
| `getMutationCache()` | 获取 mutation 缓存 |
| `getQueryCache()` | 获取查询缓存 |

```ts
// 典型：fetchQuery（一次调用拿到数据）
const data = await queryClient.fetchQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
})
```

## 三、缓存生命周期与垃圾回收

### 缓存数据状态流转

```
请求成功 → data 写入缓存
  ├─ 有新订阅者使用 → active（活跃）
  ├─ 无订阅者 → inactive（不活跃）
  └─ inactive 超过 gcTime → 被垃圾回收（GC）
```

### 两个关键时间：`staleTime` vs `gcTime`

| 维度 | `staleTime` | `gcTime` |
|---|---|---|
| 含义 | 数据**过期的判定** | 数据**在缓存中保留多久** |
| 默认值 | `0`（立即过期） | `5 * 60 * 1000`（5 分钟） |
| 过期后果 | 过期数据会被后台自动重拉 | 到点后**数据被删除** |
| 是否触发请求 | 是（stale 才 refetch） | 否（纯内存回收） |

```ts
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  staleTime: 30 * 1000, // 30s 内视为新鲜，不重拉
  gcTime: 10 * 60 * 1000, // 10 分钟内保留，超过回收
})
```

> 记忆：**stale 决定要不要重新请求；gc 决定缓存还能不能复用。**

### `staleTime: Infinity` 与 `'static'`（v5 新增）

| 取值 | 行为 |
|---|---|
| `Infinity` | 永不因过期重拉，但 `invalidateQueries` 仍可使其失效 |
| `'static'` | **彻底不重拉**，连 `invalidateQueries`、`refetchOn*` 都拦下（v5.79+） |

`'static'` 用于运行时不可变的数据（启动时的功能开关、登录权限、静态配置表）。

## 四、重要默认值一览（Important Defaults）

| 默认行为 | 值 |
|---|---|
| 缓存数据是否过期 | **默认视为 stale（过期）** |
| 自动重拉时机 | 挂载新实例 / 窗口聚焦 / 网络重连 |
| 不活跃查询 GC | 5 分钟 |
| 失败重试 | **静默重试 3 次**（指数退避） |
| 引用稳定 | **结构化共享**（structural sharing） |

## 五、内存与性能要点

- **inactive 查询**：`gcTime` 内保留，重新用到即恢复 active。
- 想禁用 GC（常驻缓存）：`gcTime: Infinity`（谨慎，会内存泄漏风险）。
- 超大响应导致性能问题时，可关 `structuralSharing: false` 或用自定义共享函数。
- 非 JSON 值（如 Map、Date、class 实例）**不受**结构化共享优化，始终视为已变化。

**相关官方链接**
- Caching 指南：<https://tanstack.com/query/v5/docs/framework/react/guides/caching>
- QueryClient API：<https://tanstack.com/query/v5/docs/framework/react/reference/QueryClient>
- Important Defaults：<https://tanstack.com/query/v5/docs/framework/react/guides/important-defaults>
