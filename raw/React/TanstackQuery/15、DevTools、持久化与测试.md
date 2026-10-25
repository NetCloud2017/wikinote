# DevTools、持久化与测试

> **版本基线**：`@tanstack/react-query` **v5.104.x**。
> 官方文档：<https://tanstack.com/query/v5/docs/framework/react/devtools> · <https://tanstack.com/query/v5/docs/framework/react/plugins/persistQueryClient> · <https://tanstack.com/query/v5/docs/framework/react/guides/testing>

## 一、DevTools（开发调试）

### 安装与挂载

```bash
npm i -D @tanstack/react-query-devtools
```

```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <AppComponent />
      {/* 只在开发环境挂载 */}
      {process.env.NODE_ENV === 'development' && (
        <ReactQueryDevtools initialIsOpen={false} />
      )}
    </QueryClientProvider>
  )
}
```

### 能力

- 可视化**缓存中的每个查询**：key、status、data、dataUpdatedAt、fetchStatus
- 手动 **refetch / invalidate / remove / reset** 单个查询
- 查看 **mutation** 历史与状态
- 搜索 / 过滤查询

> DevTools 是定位「数据是不是缓存了、为什么没刷新」的首选工具。

## 二、持久化缓存（persistQueryClient）

把缓存写入 **localStorage / IndexedDB / 自定义存储**，刷新后可用旧数据，减少首屏请求。**官方标记为实验性（experimental）**。

### 最简接入（localStorage）

```ts
import { PersistQueryClientProvider } from '@tanstack/react-query-persist-client'
import { createSyncStoragePersister } from '@tanstack/query-persist-client-core'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      gcTime: 1000 * 60 * 60 * 24, // 想持久化，gcTime 要够长（否则秒级回收）
    },
  },
})

const persister = createSyncStoragePersister({
  storage: window.localStorage,
})

// 用 PersistQueryClientProvider 替代 QueryClientProvider
<PersistQueryClientProvider
  client={queryClient}
  persistOptions={{ persister, maxAge: 1000 * 60 * 60 * 24 }}
>
  <App />
</PersistQueryClientProvider>
```

### 关键注意点

| 项 | 说明 |
|---|---|
| 包 | `@tanstack/react-query-persist-client` |
| persister | `createSyncStoragePersister`（localStorage）/ `createAsyncStoragePersister`（IndexedDB） |
| `gcTime` | 必须**大于 maxAge**，否则数据被 GC 提前回收 |
| 特性 | 按 `maxAge` 丢弃过期缓存；写权限/quota 不足会降级 |
| 状态 | **实验性**，生产谨慎 |
| 离线 | 可配合 `onlineManager` 做离线优先体验 |

## 三、测试（Testing）

### 关键工具

| 工具 | 作用 |
|---|---|
| `QueryClientProvider` + `new QueryClient` | 包裹被测组件 |
| `queryClient.clear()` | 每个用例前清缓存 |
| `queryClient.setQueryDefaults` | 测试统一配置（如静默 retry） |
| React Testing Library | 交互与断言 |

### 标准模板（Vitest + RTL）

```tsx
import { render, screen } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

function renderWithClient(ui: React.ReactNode) {
  const testClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false, // 测试里关闭重试，快速暴露错误
      },
    },
  })
  return render(
    <QueryClientProvider client={testClient}>{ui}</QueryClientProvider>,
  )
}

// 每个用例新建 client，避免跨用例缓存污染
beforeEach(() => { /* testClient.clear() 或重建 */ })

it('渲染数据', async () => {
  renderWithClient(<Posts />)
  expect(await screen.findByText('Hello')).toBeInTheDocument()
})
```

### 测试常见坑

- 每次用例**新建 QueryClient**（否则缓存串用例）。
- 关掉 `retry`，否则失败用例会重试拖慢/误判。
- 异步断言用 `findBy*` / `waitFor`，别用同步 `getBy`。
- 需要隔离 server 用 **MSW（Mock Service Worker）** 拦截网络。

## 四、小节

| 场景 | 方案 |
|---|---|
| 开发期看缓存/调试 | ReactQueryDevtools |
| 刷新后想复用缓存 | persistQueryClient（实验性） |
| 本地大存储 | IndexedDB persister |
| 写单测 | 独立 QueryClient + 关闭 retry + MSW |

**相关官方链接**
- DevTools：<https://tanstack.com/query/v5/docs/framework/react/devtools>
- persistQueryClient：<https://tanstack.com/query/v5/docs/framework/react/plugins/persistQueryClient>
- Testing：<https://tanstack.com/query/v5/docs/framework/react/guides/testing>
