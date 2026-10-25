# SSR、SSG 与 Next.js 集成

> **版本基线**：`@tanstack/react-query` **v5.104.x**；涉及 Next.js 内容以 **Next.js 16** 为基线。
> 官方文档：<https://tanstack.com/query/v5/docs/framework/react/guides/ssr> · <https://tanstack.com/query/v5/docs/framework/react/guides/advanced-ssr>

## 一、核心目标

让服务端**预取数据、填入缓存**，客户端**水合（hydration）时直接命中缓存**，避免首屏闪现 loading，并减少重复请求。

## 二、三大构件

| 构件 | 作用 |
|---|---|
| `prefetchQuery` / `prefetchInfiniteQuery` | 服务端/路由层把数据**预取进缓存** |
| `HydrationBoundary` | 把**序列化后的缓存**传给客户端水合 |
| `dehydrate(queryClient)` | 把缓存状态序列化为可传输对象 |

## 三、通用 SSR 模式（服务端预取 + 水合）

```tsx
import { HydrationBoundary, dehydrate, useQueryClient } from '@tanstack/react-query'

export function PrefetchTodos({ children }: { children: React.ReactNode }) {
  const queryClient = useQueryClient()

  // 服务端/构建期已执行；客户端在首次渲染时同样预取
  void queryClient.prefetchQuery(todoOptions)

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      {children}
    </HydrationBoundary>
  )
}
```

> 这是 App Router / 普通 SSR 通用的做法：**预取 + dehydrate + HydrationBoundary** 三位一体。

## 四、Next.js App Router（RSC，Next.js 16）

### 关键约束

- 服务端组件（Server Component）**不能**用 `useQuery`（它需要客户端 hooks）。
- 正确姿势：**在服务端组件里 `prefetchQuery`**，把数据交给 `HydrationBoundary`；**客户端组件**里用 `useQuery` 命中缓存。

### 官方推荐模板（Next.js App Router）

```tsx
// app/posts/page.tsx —— Server Component
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
} from '@tanstack/react-query'
import { Posts } from './posts'

export default async function PostsPage() {
  // 每个请求一个实例（避免跨请求共享脏缓存）
  const queryClient = new QueryClient()

  await queryClient.prefetchQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  })

  return (
    // 把缓存传给客户端水合
    <HydrationBoundary state={dehydrate(queryClient)}>
      <Posts />
    </HydrationBoundary>
  )
}

// app/posts/posts.tsx —— Client Component
'use client'
import { useQuery } from '@tanstack/react-query'

export function Posts() {
  const { data } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts, // 命中已预取缓存，不再重复请求
  })
  return <ul>{data?.map(p => <li key={p.id}>{p.title}</li>)}</ul>
}
```

> ⚠️ 每次请求**新建** `new QueryClient()`，否则多请求间共享脏缓存（官方强烈建议）。

### 与 Suspense + 流式渲染组合（Next.js 16）

```tsx
// Server Component 里套客户端 Suspense → 流式输出
import { Suspense } from 'react'
import { Posts } from './posts'

export default function Page() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <Posts /> {/* useSuspenseQuery 的客户端组件 */}
    </Suspense>
  )
}
```

## 五、SSG（静态生成）

构建期一次性预取，产出静态页面 + 已填充缓存：

```tsx
// app/page.tsx —— 若静态生成，构建时执行
export default async function Home() {
  const queryClient = new QueryClient()
  await queryClient.prefetchQuery(todoOptions)
  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <TodoList />
    </HydrationBoundary>
  )
}
```

> 配合 `generateStaticParams` 批量静态生成；静态页数据会「过期」，客户端可再靠 staleTime/后台刷新更新。

## 六、注意事项清单

| 注意点 | 建议 |
|---|---|
| 服务端能不能用 useQuery | 不能（RSC 需客户端 hooks），用 prefetch |
| QueryClient 生命周期 | **每个请求新建**，防脏缓存 |
| 序列化边界 | `dehydrate` 的 cache 必须是 JSON 可序列化 |
| 冗余请求 | 预取 + 同 key 的 useQuery 自动去重，不重复发 |
| 预取失败 | `prefetchQuery` 失败不抛错（仅 console.error） |
| 数据一致性 | 客户端仍以 staleTime/失效策略控制刷新 |

**相关官方链接**
- SSR 指南：<https://tanstack.com/query/v5/docs/framework/react/guides/ssr>
- Advanced SSR（Next.js App Router）：<https://tanstack.com/query/v5/docs/framework/react/guides/advanced-ssr>
